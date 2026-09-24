## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Test doubles

## Назначение

**Test double** — это общее название для любых объектов, которые заменяют реальные зависимости в тестах.

Главная идея:

> Test double позволяет заменить настоящую зависимость на контролируемую тестовую версию.

Например:

Production:

```text
UserService
      |
      ↓
PostgreSQL
```

Test:

```text
UserService
      |
      ↓
Test double
```

---

# Зачем нужны Test doubles

Без них:

```text
Unit test

UserService
      |
      ↓
PostgreSQL
      |
      ↓
Database
```

Проблемы:

- тесты медленные;
    
- нужны внешние сервисы;
    
- сложно воспроизвести ошибки;
    
- результат зависит от окружения.
    

С test double:

```text
Unit test

UserService
      |
      ↓
Fake / Mock / Stub
```

Тест становится:

- быстрым;
    
- изолированным;
    
- предсказуемым.
    

---

# Виды Test doubles

Основные виды:

```text
Test double
    |
    ├── Stub
    |
    ├── Mock
    |
    ├── Fake
    |
    └── Spy
```

---

# 1. Stub

## Идея

Stub предоставляет заранее подготовленный ответ.

Вопрос:

> "Что вернула зависимость?"

---

Пример:

```go
type UserRepositoryStub struct {
	user *User
}

func (s UserRepositoryStub) Get(id int) (*User, error) {

	return s.user, nil

}
```

Использование:

```go
stub := UserRepositoryStub{
	user: &User{
		ID: 1,
		Name: "Max",
	},
}
```

Проверяем:

```text
Repository вернул пользователя
        |
        ↓
Service правильно обработал его
```

Stub ничего не знает:

- кто его вызвал;
    
- сколько раз;
    
- с каким аргументом.
    

---

# 2. Mock

## Идея

Mock проверяет взаимодействие.

Вопрос:

> "Правильно ли мой код вызвал зависимость?"

---

Пример:

```go
type UserRepositoryMock struct {
	called bool
	id int
}

func (m *UserRepositoryMock) Get(id int) (*User, error) {

	m.called = true
	m.id = id

	return &User{
		ID: id,
	}, nil
}
```

Проверка:

```go
if !mock.called {
	t.Fatal("method was not called")
}

if mock.id != 10 {
	t.Fatal("wrong id")
}
```

Проверяем:

```text
Service
   |
   ↓
Get(10)

Был вызов?
Аргумент правильный?
```

---

# 3. Fake

## Идея

Fake — рабочая, но упрощённая реализация.

Вопрос:

> "Работает ли логика с настоящей реализацией?"

---

Пример:

Вместо PostgreSQL:

```go
type InMemoryRepository struct {

	users map[int]User

}


func (r *InMemoryRepository) Save(
	user User,
) {

	r.users[user.ID] = user

}


func (r *InMemoryRepository) Get(
	id int,
) (*User,error) {

	user := r.users[id]

	return &user,nil

}
```

Это уже настоящая логика:

```text
Save()
   |
   ↓
хранение данных

Get()
   |
   ↓
получение данных
```

---

# 4. Spy

Spy похож на Mock, но обычно **не ломает выполнение**, а только собирает информацию.

Вопрос:

> "Что произошло во время выполнения?"

---

Пример:

```go
type EmailSenderSpy struct {

	sentTo []string

}


func (s *EmailSenderSpy) Send(
	email string,
) error {

	s.sentTo = append(
		s.sentTo,
		email,
	)

	return nil
}
```

После теста:

```go
if len(spy.sentTo) != 1 {
	t.Fatal("email not sent")
}
```

Проверяем:

```text
Что произошло?
Сколько писем отправили?
Кому?
```

---

# Stub vs Mock vs Fake vs Spy

||Stub|Mock|Fake|Spy|
|---|---|---|---|---|
|Даёт ответ|✅|✅|✅|✅|
|Проверяет вызовы|❌|✅|❌|✅ после выполнения|
|Хранит состояние|❌|обычно ❌|✅|✅|
|Реализует логику|❌|❌|✅|частично|
|Основная цель|данные|взаимодействие|реализация|наблюдение|

---

# Аналогия

## Stub

Ты спрашиваешь:

> "Что скажет сервер?"

Stub:

> "Всегда отвечу: `200 OK`"

---

## Mock

Ты спрашиваешь:

> "Ты точно позвонил серверу?"

Mock:

> "Да, ты вызвал меня с параметром X"

---

## Fake

Ты спрашиваешь:

> "Можно ли работать без настоящего сервера?"

Fake:

> "Я маленькая копия сервера в памяти"

---

## Spy

Ты спрашиваешь:

> "Что происходило?"

Spy:

> "Я записал все действия"

---

# В Go

Test doubles обычно используются вместе с:

## Interface

```go
type Repository interface {
	Get(id int) (*User,error)
}
```

---

## Dependency Injection

Production:

```go
service := NewService(
	postgresRepo,
)
```

Test:

```go
service := NewService(
	mockRepo,
)
```

---

## Библиотеки

Для Mock:

- `gomock`
    
- `mockery`
    

Для Fake/Stub:

обычно пишут вручную.

---

# Когда что использовать

## Stub

Когда нужно проверить обработку ответа:

```text
API client
    ↓
Stub response
    ↓
Business logic
```

---

## Mock

Когда важно взаимодействие:

```text
Service
    ↓
Kafka producer

Publish() вызван?
```

---

## Fake

Когда нужна рабочая замена:

```text
PostgreSQL
      ↓
In-memory repository
```

---

## Spy

Когда нужно собрать информацию:

```text
Email sender
      ↓
Spy

сколько писем отправлено?
```

---

# Для собеседования

> Test double — это общий термин для тестовых заменителей реальных зависимостей. К ним относятся stub, mock, fake и spy. Stub предоставляет заранее заданные ответы, mock проверяет взаимодействие через вызовы и аргументы, fake представляет собой упрощённую рабочую реализацию, а spy записывает информацию о выполнении. В Go test doubles обычно используются через интерфейсы и Dependency Injection для изоляции unit-тестов от внешних систем.