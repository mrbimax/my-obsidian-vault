## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Dependency Injection

## Назначение

Dependency Injection (DI) — передача зависимостей в объект **извне**, вместо создания их внутри.

Главная идея:

> Код не должен сам создавать свои зависимости. Он должен получать их готовыми.

В тестах DI позволяет заменять реальные зависимости на:

- mock;
    
- fake;
    
- stub;
    
- test implementation.
    

---

# Проблема без DI

Плохо:

```go
type UserService struct {
}

func (s UserService) GetUser(id int) (*User, error) {

	db := postgres.New()

	return db.Find(id)

}
```

Проблемы:

- сервис жёстко связан с PostgreSQL;
    
- сложно тестировать;
    
- unit-тест требует настоящую БД.
    

---

# DI через интерфейс

Создаём интерфейс:

```go
type UserRepository interface {

	Find(id int) (*User, error)

}
```

Сервис принимает зависимость:

```go
type UserService struct {

	repo UserRepository

}
```

---

## Production код

Передаём реальную реализацию:

```go
repo := NewPostgresRepository(db)

service := UserService{
	repo: repo,
}
```

Получается:

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

## Test код

Передаём mock:

```go
mockRepo := MockRepository{}

service := UserService{
	repo: mockRepo,
}
```

Получается:

```text
UserService
      |
      ↓
UserRepository
      |
      ↓
Mock
```

---

# Пример теста

Интерфейс:

```go
type Repository interface {

	Get(id int) (*User, error)

}
```

Fake:

```go
type FakeRepository struct {

	user *User

}

func (f FakeRepository) Get(id int) (*User, error) {

	return f.user, nil

}
```

Тест:

```go
func TestUserService_Get(t *testing.T) {

	repo := FakeRepository{
		user: &User{
			ID: 1,
		},
	}

	service := UserService{
		repo: repo,
	}

	user, err := service.Get(1)

	if err != nil {
		t.Fatal(err)
	}

	if user.ID != 1 {
		t.Fatal("wrong id")
	}

}
```

---

# DI через конструктор

Частый стиль:

```go
type Service struct {

	repo Repository

}
```

Конструктор:

```go
func NewService(
	repo Repository,
) *Service {

	return &Service{
		repo: repo,
	}

}
```

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

# DI через функции

В Go часто используют функциональные зависимости:

```go
type Service struct {

	now func() time.Time

}
```

Production:

```go
Service{
	now: time.Now,
}
```

Test:

```go
Service{
	now: func() time.Time {

		return fixedTime

	},
}
```

Теперь тест не зависит от реального времени.

---

# DI и стандартный пакет testing

`testing` не предоставляет DI напрямую.

Он только даёт инструменты:

- `t.Cleanup()` — управление ресурсами;
    
- `t.TempDir()` — временные файлы;
    
- `t.Setenv()` — окружение;
    
- `t.Run()` — сценарии.
    

DI — это архитектурный подход.

---

# DI vs Mock

Это разные вещи.

DI:

```text
как передать зависимость
```

Mock:

```text
какую зависимость передать
```

Пример:

```text
DI:

Service(interface)


Mock:

Service(MockImplementation)
```

---

# DI vs Global variables

Плохо:

```go
var db *sql.DB

func GetUser(id int) {

	db.Query(...)

}
```

Проблемы:

- сложно заменить;
    
- тесты влияют друг на друга;
    
- скрытые зависимости.
    

Лучше:

```go
type Service struct {

	db DB

}
```

---

# DI и Clean Architecture

Частая структура:

```text
Handler
   |
   ↓
UseCase
   |
   ↓
Repository interface
   |
   ↓
Postgres implementation
```

В тестах:

```text
UseCase
   |
   ↓
Mock Repository
```

---

## Для собеседования

> Dependency Injection — это передача зависимостей в компонент извне вместо создания их внутри. В Go чаще всего DI реализуется через интерфейсы и конструкторы. Это позволяет в unit-тестах заменять реальные зависимости на mock или fake реализации и тестировать бизнес-логику изолированно от базы данных, сети и других внешних систем.