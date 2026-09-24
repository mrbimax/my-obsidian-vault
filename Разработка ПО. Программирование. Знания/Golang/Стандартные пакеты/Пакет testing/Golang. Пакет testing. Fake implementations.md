## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fake implementations

## Назначение

**Fake implementation** — это упрощённая, но рабочая реализация интерфейса, которая используется вместо настоящей зависимости в тестах.

Главная идея:

> Fake не просто возвращает заранее заданное значение, а реально выполняет часть логики, но проще, быстрее и без внешних ресурсов.

---

# Пример: Repository

Есть интерфейс:

```go
type UserRepository interface {
	Save(user User) error
	Get(id int) (*User, error)
}
```

Production реализация:

```text
UserRepository
       |
       ↓
PostgreSQL
       |
       ↓
таблица users
```

---

# Fake реализация

Вместо PostgreSQL используем память:

```go
type FakeUserRepository struct {
	users map[int]User
}

func NewFakeUserRepository() *FakeUserRepository {
	return &FakeUserRepository{
		users: make(map[int]User),
	}
}


func (r *FakeUserRepository) Save(user User) error {

	r.users[user.ID] = user

	return nil
}


func (r *FakeUserRepository) Get(id int) (*User, error) {

	user, ok := r.users[id]

	if !ok {
		return nil, errors.New("not found")
	}

	return &user, nil
}
```

---

# Использование в тесте

```go
func TestUserService_Create(t *testing.T) {

	repo := NewFakeUserRepository()

	service := UserService{
		repo: repo,
	}


	err := service.Create(
		User{
			ID:   1,
			Name: "Max",
		},
	)

	if err != nil {
		t.Fatal(err)
	}


	user, err := repo.Get(1)

	if err != nil {
		t.Fatal(err)
	}


	if user.Name != "Max" {
		t.Fatal("wrong name")
	}

}
```

---

# Чем Fake отличается от Stub

## Stub

```go
func (s Stub) Get(id int) (*User, error) {

	return User{
		Name: "Max",
	}, nil

}
```

Он говорит:

> "Неважно, что ты спросил — вот заранее подготовленный ответ".

---

## Fake

```go
func (f Fake) Save(user User) {

	f.users[user.ID] = user

}
```

Он говорит:

> "Я действительно сохраню данные, просто не в PostgreSQL".

---

# Чем Fake отличается от Mock

## Mock

Проверяет взаимодействие:

```go
repo.EXPECT().
	Save(user)
```

Вопрос:

> "Сервис вызвал Save?"

---

## Fake

Проверяет состояние:

```go
user, _ := repo.Get(1)
```

Вопрос:

> "Данные действительно сохранились?"

---

# Сравнение

||Stub|Mock|Fake|
|---|---|---|---|
|Возвращает данные|✅|✅|✅|
|Хранит состояние|❌|обычно ❌|✅|
|Проверяет вызовы|❌|✅|❌|
|Реализует логику|❌|❌|✅|
|Близок к production|❌|❌|✅|

---

# Реальные примеры Fake в Go

## In-memory database

Вместо:

```text
PostgreSQL
```

Используем:

```go
map[string]User
```

---

## Fake clock

Вместо:

```go
time.Now()
```

Используем:

```go
type FakeClock struct {
	NowTime time.Time
}

func (c FakeClock) Now() time.Time {
	return c.NowTime
}
```

Тест:

```go
clock := FakeClock{
	NowTime: time.Date(
		2026, 1, 1,
		0, 0, 0, 0,
		time.UTC,
	),
}
```

Теперь время контролируемое.

---

## Fake message queue

Вместо Kafka:

```go
type FakeQueue struct {
	Messages []Message
}

func (q *FakeQueue) Publish(m Message) {

	q.Messages = append(
		q.Messages,
		m,
	)

}
```

Проверка:

```go
if len(queue.Messages) != 1 {
	t.Fatal("message not sent")
}
```

---

# Когда использовать Fake

Хорошо подходит для:

- repository;
    
- cache;
    
- queue;
    
- clock;
    
- storage;
    
- внешних сервисов.
    

Особенно когда:

- нужна реалистичная логика;
    
- много тестов;
    
- mock становится слишком сложным.
    

---

# Когда НЕ использовать Fake

Не стоит делать fake для всего подряд.

Плохо:

```text
FakePostgres
FakeKafka
FakeRedis
```

если они становятся полноценными копиями production.

Тогда лучше:

- integration tests;
    
- Testcontainers;
    
- настоящие сервисы.
    

---

# Для собеседования

> Fake implementation — это упрощённая рабочая реализация интерфейса, которая заменяет настоящую зависимость в тестах. В отличие от stub, fake содержит внутреннее состояние и выполняет реальную логику, например хранит данные в памяти вместо PostgreSQL. В отличие от mock, fake не проверяет вызовы, а позволяет тестировать результат работы через состояние системы. В Go часто используют in-memory fake реализации для репозиториев, очередей и внешних сервисов.