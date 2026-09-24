## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Stub objects

## Назначение

**Stub object** — это тестовая заглушка, которая возвращает заранее подготовленные ответы.

Главная идея:

> Stub нужен, чтобы сказать зависимости: "Когда меня вызовут — верни вот такой результат".

Stub **не хранит сложное состояние** и **не проверяет, как его использовали**.

---

# Пример: UserService

Есть сервис:

```go
type UserService struct {
	repo UserRepository
}

func (s UserService) GetName(id int) (string, error) {

	user, err := s.repo.Get(id)

	if err != nil {
		return "", err
	}

	return user.Name, nil
}
```

Зависимость:

```go
type UserRepository interface {
	Get(id int) (*User, error)
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

В unit-тесте PostgreSQL не нужен.

---

# Stub реализация

Stub просто возвращает подготовленный результат:

```go
type UserRepositoryStub struct {
	User *User
	Err  error
}

func (s UserRepositoryStub) Get(id int) (*User, error) {

	return s.User, s.Err

}
```

---

# Использование Stub

```go
func TestUserService_GetName(t *testing.T) {

	stub := UserRepositoryStub{
		User: &User{
			ID:   1,
			Name: "Max",
		},
	}

	service := UserService{
		repo: stub,
	}


	name, err := service.GetName(100)

	if err != nil {
		t.Fatal(err)
	}


	if name != "Max" {
		t.Fatal("wrong name")
	}

}
```

---

Что произошло:

```
UserService
      |
      ↓
UserRepositoryStub

Get(100)

      ↓

User{
    Name: "Max"
}
```

Сервису всё равно:

```text
какой был id?
сколько раз вызвали?
кто вызвал?
```

Он получил ответ и обработал его.

---

# Stub с ошибкой

Частый сценарий:

```go
func TestUserService_Error(t *testing.T) {

	stub := UserRepositoryStub{
		Err: errors.New("database error"),
	}


	service := UserService{
		repo: stub,
	}


	_, err := service.GetName(1)


	if err == nil {
		t.Fatal("expected error")
	}

}
```

Проверяем:

```
Что сделает сервис,
если зависимость вернула ошибку?
```

---

# Stub vs Mock

## Stub

Фокус:

```
какой ответ получила система?
```

Пример:

```
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

```
Service правильно обработал User?
```

---

## Mock

Фокус:

```
как система взаимодействовала?
```

Пример:

```
Service
   |
   ↓
 Mock

Get(10)
 |
 ↓
запомнил вызов
```

Проверяем:

```
Метод вызвали?
Аргумент правильный?
Количество вызовов правильное?
```

---

# На одном примере

Допустим:

```go
service.GetName(10)
```

---

## Stub

Мы задаём:

```go
Stub{
	User: User{
		Name:"Max",
	},
}
```

Вопрос:

> Что произойдёт, если репозиторий вернёт Max?

---

## Mock

Мы задаём ожидание:

```go
Mock.EXPECT().
	Get(10)
```

Вопрос:

> Сервис вообще запросил пользователя 10?

---

# Stub vs Fake

Они похожи, но разница важная.

---

## Stub

Просто ответ:

```go
func Get(id int) (*User,error){

	return User{
		Name:"Max",
	},nil

}
```

Логика:

```
вызвали → вернул значение
```

---

## Fake

Настоящая упрощённая реализация:

```go
type FakeRepository struct {
	users map[int]User
}

func (f FakeRepository) Get(id int) (*User,error){

	user := f.users[id]

	return &user,nil
}
```

Логика:

```
сохранили данные
       ↓
получили данные
```

---

# Сравнение

||Stub|Mock|Fake|
|---|---|---|---|
|Возвращает данные|✅|✅|✅|
|Проверяет вызовы|❌|✅|❌|
|Хранит состояние|❌|❌|✅|
|Выполняет реальную логику|❌|❌|✅|
|Заменяет внешнюю систему|✅|✅|✅|

---

# Реальные примеры Stub в Go

## HTTP клиент

Вместо:

```
Service
   |
   ↓
External API
```

Stub:

```go
type HTTPClientStub struct{}

func (c HTTPClientStub) Get(url string) ([]byte,error){

	return []byte(`{"status":"ok"}`),nil

}
```

---

## Время

Вместо:

```go
time.Now()
```

Stub:

```go
type ClockStub struct{}

func (ClockStub) Now() time.Time {

	return time.Date(
		2026,
		1,
		1,
		0,
		0,
		0,
		0,
		time.UTC,
	)

}
```

---

## Конфигурация

Вместо:

```go
os.Getenv("TOKEN")
```

Stub:

```go
type ConfigStub struct{}

func (ConfigStub) Token() string {

	return "test-token"

}
```

---

# Когда использовать Stub

Использовать, когда нужно:

✅ проверить обработку результата зависимости

Например:

```
API client
     ↓
Stub response
     ↓
Service logic
```

---

✅ проверить ошибки:

```
Database
    ↓
Stub returns error
    ↓
Service handles error
```

---

# Когда НЕ использовать Stub

Если важно:

- был ли вызов;
    
- сколько раз вызвали;
    
- с какими аргументами;
    

тогда нужен **Mock**.

Если нужна настоящая упрощённая логика — нужен **Fake**.

---

# Для собеседования

> Stub object — это тестовая заглушка, которая возвращает заранее подготовленные данные или ошибки. Он используется для проверки того, как тестируемый код обрабатывает результат зависимости. В отличие от mock, stub не проверяет взаимодействие с объектом и не отслеживает вызовы. В Go stub обычно реализуется вручную через интерфейсы и Dependency Injection.