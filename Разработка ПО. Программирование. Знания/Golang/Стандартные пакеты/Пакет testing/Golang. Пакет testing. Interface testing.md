## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Interface testing

## Назначение

Interface testing — тестирование кода через интерфейсы вместо конкретных реализаций.

Основная идея:

> Тестировать поведение зависимости, а не её внутреннюю реализацию.

Чаще всего используется для:

- mock-объектов;
    
- dependency injection;
    
- изоляции unit-тестов;
    
- проверки контрактов.
    

---

## Пример проблемы

Есть сервис:

```go
type UserService struct {
	repo UserRepository
}
```

Зависимость:

```go
type UserRepository interface {

	Find(id int) (*User, error)

}
```

Реализация:

```go
type PostgresRepository struct {

}
```

Если тестировать напрямую:

```go
service := UserService{
	repo: PostgresRepository{},
}
```

тест зависит от:

- базы данных;
    
- сети;
    
- схемы таблиц.
    

Это уже ближе к integration test.

---

## Тест через интерфейс

Создаём mock:

```go
type MockRepository struct {

	user *User
	err  error

}
```

Реализуем интерфейс:

```go
func (m MockRepository) Find(id int) (*User, error) {

	return m.user, m.err

}
```

Тест:

```go
func TestUserService(t *testing.T) {

	repo := MockRepository{
		user: &User{
			ID: 1,
		},
	}

	service := UserService{
		repo: repo,
	}

	user, err := service.GetUser(1)

	if err != nil {
		t.Fatal(err)
	}

	if user.ID != 1 {
		t.Fatal("wrong user")
	}

}
```

---

## Что проверяем

Не:

```text
PostgreSQL правильно работает?
```

А:

```text
UserService правильно взаимодействует с Repository?
```

---

## Проверка контрактов интерфейса

Можно проверить, что тип реализует интерфейс:

```go
var _ UserRepository = (*PostgresRepository)(nil)
```

Если `PostgresRepository` не реализует интерфейс — ошибка компиляции.

---

## Использование mock библиотек

В больших проектах используют:

- [[gomock]];
    
- [[mockery]];
    
- [[mock]].
    

Пример с mockery:

Интерфейс:

```go
type Repository interface {
	Get(id int) error
}
```

Генерируется:

```go
type MockRepository struct {
	mock.Mock
}
```

Тест:

```go
repo.On(
	"Get",
	1,
).Return(nil)
```

---

## Interface testing vs Integration testing

## Interface testing

```text
Service
   |
   ↓
Interface
   |
   ↓
Mock
```

Проверяет:

- бизнес-логику;
    
- взаимодействие компонентов.
    

---

## Integration testing

```text
Service
   |
   ↓
PostgreSQL
```

Проверяет:

- реальную БД;
    
- SQL;
    
- миграции;
    
- инфраструктуру.
    

---

## Хорошая практика

Плохо:

```go
type Service struct {
	db *sql.DB
}
```

Сложно тестировать.

---

Лучше:

```go
type Service struct {
	repo Repository
}
```

Теперь:

```text
production:

Service
   |
PostgresRepository


test:

Service
   |
MockRepository
```

---

## Interface testing и Go

Go хорошо подходит для такого подхода, потому что:

- интерфейсы не требуют явного объявления реализации;
    
- зависимости легко заменить;
    
- маленькие интерфейсы проще мокать.
    

---

## Для собеседования

> Interface testing в Go — это тестирование компонентов через интерфейсы и подмену зависимостей mock-реализациями. Такой подход позволяет изолировать unit-тесты от базы данных, сети и внешних сервисов. Обычно через интерфейсы тестируют бизнес-логику, а реальные реализации проверяют отдельными integration-тестами. В Go это удобно благодаря неявной реализации интерфейсов и dependency injection.