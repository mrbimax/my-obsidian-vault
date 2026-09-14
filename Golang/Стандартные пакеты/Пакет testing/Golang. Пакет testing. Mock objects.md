## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Mock objects

## Назначение

**Mock object** — это тестовая реализация зависимости, которая не просто возвращает данные, а **отслеживает взаимодействие с ней**.

Главная идея:

> Mock нужен, чтобы проверить: "Правильно ли мой код общался с зависимостью?"

Mock может проверить:

- был ли вызван метод;
    
- с какими аргументами;
    
- сколько раз;
    
- в каком порядке.
    

---

# Пример: UserService

Есть сервис:

```go
type UserService struct {
	repo UserRepository
}

func (s UserService) DeleteUser(id int) error {

	return s.repo.Delete(id)

}
```

Зависимость:

```go
type UserRepository interface {

	Delete(id int) error

}
```

Production:

```text
UserService
      |
      ↓
UserRepository
      |
      ↓
PostgreSQL
```

---

# Проблема без Mock

Если использовать настоящий репозиторий:

```go
service := UserService{
	repo: PostgresRepository{},
}
```

Тест проверяет сразу много вещей:

```text
UserService
      |
      ↓
Repository
      |
      ↓
PostgreSQL
      |
      ↓
Database
```

Проблемы:

- нужен реальный сервер БД;
    
- тесты медленные;
    
- сложно проверить конкретное взаимодействие.
    

---

# Mock реализация

Создаём объект, который запоминает вызов:

```go
type UserRepositoryMock struct {

	Called bool
	DeletedID int

}


func (m *UserRepositoryMock) Delete(id int) error {

	m.Called = true
	m.DeletedID = id

	return nil

}
```

---

# Использование Mock

```go
func TestUserService_DeleteUser(t *testing.T) {

	mock := &UserRepositoryMock{}

	service := UserService{
		repo: mock,
	}


	err := service.DeleteUser(10)

	if err != nil {
		t.Fatal(err)
	}


	if !mock.Called {
		t.Fatal(
			"Delete was not called",
		)
	}


	if mock.DeletedID != 10 {
		t.Fatal(
			"wrong id",
		)
	}

}
```

---

Что произошло:

```text
UserService
      |
      ↓
UserRepositoryMock

Delete(10)

      ↓

запомнил:

Called = true
DeletedID = 10
```

Мы проверили:

> "Сервис действительно вызвал удаление пользователя с правильным ID?"

---

# Mock возвращающий ошибку

Например, база недоступна:

```go
type UserRepositoryMock struct {

	Err error

}


func (m UserRepositoryMock) Delete(id int) error {

	return m.Err

}
```

Тест:

```go
func TestUserService_Delete_Error(t *testing.T) {

	mock := UserRepositoryMock{
		Err: errors.New("database error"),
	}


	service := UserService{
		repo: mock,
	}


	err := service.DeleteUser(1)


	if err == nil {
		t.Fatal(
			"expected error",
		)
	}

}
```

Проверяем:

```text
Что сделает сервис,
если зависимость вернула ошибку?
```

---

# Главное отличие от Stub

## Stub

Фокус:

```text
какой ответ вернула зависимость?
```

Пример:

```text
Service
   |
   ↓
Stub

Get()
   |
   ↓
User{}
```

Проверяем:

```text
Service правильно обработал User?
```

---

## Mock

Фокус:

```text
как происходило взаимодействие?
```

Пример:

```text
Service
   |
   ↓
Mock

Delete(10)

   ↓

записал вызов
```

Проверяем:

```text
Delete вызван?
ID правильный?
```

---

# Mock vs Fake

Они тоже похожи.

---

## Mock

Проверяет действие:

```go
repo.Delete(10)
```

Вопрос:

> "Был ли вызван Delete?"

---

## Fake

Выполняет действие:

```go
repo.Delete(10)
```

Например:

```go
users = remove(users, 10)
```

Вопрос:

> "Пользователь реально удалился?"

---

# Mock через gomock

В реальных проектах обычно не пишут вручную.

Используют:

- gomock;
    
- mockery.
    

---

Интерфейс:

```go
type UserRepository interface {

	Delete(id int) error

}
```

Генерируется:

```go
MockUserRepository
```

Тест:

```go
mockRepo.
	EXPECT().
	Delete(10).
	Return(nil)
```

Это означает:

Ожидаем:

```text
Delete(10)
```

и возвращаем:

```text
nil
```

Если сервис не вызвал метод:

```text
FAIL
missing call
```

---

# Где используют Mock

Хорошие кандидаты:

## Внешние API

```text
Service
   |
   ↓
Payment API
```

Проверяем:

```text
Pay() вызван?
Сумма правильная?
```

---

## Kafka producer

```text
Service
   |
   ↓
Kafka
```

Проверяем:

```text
Publish() вызван?
Topic правильный?
```

---

## Email sender

```text
Service
   |
   ↓
Email
```

Проверяем:

```text
Send() вызван?
Адрес правильный?
```

---

# Когда НЕ использовать Mock

Не стоит мокать всё подряд.

Плохо:

```text
Service
 |
Mock
 |
Mock
 |
Mock
```

Проблемы:

- тесты зависят от деталей реализации;
    
- сложно менять код;
    
- тесты становятся хрупкими.
    

---

# Сравнение

||Stub|Mock|Fake|
|---|---|---|---|
|Возвращает данные|✅|✅|✅|
|Проверяет вызовы|❌|✅|❌|
|Проверяет аргументы|❌|✅|❌|
|Хранит состояние|❌|обычно ❌|✅|
|Выполняет реальную логику|❌|❌|✅|

---

# Для собеседования

> Mock object — это тестовая реализация зависимости, которая позволяет проверять взаимодействие между компонентами: вызовы методов, аргументы и количество вызовов. В Go mock обычно используется вместе с интерфейсами и Dependency Injection. В отличие от stub, который только возвращает заранее подготовленные данные, mock проверяет контракт взаимодействия между объектами. Часто используются библиотеки gomock и mockery.