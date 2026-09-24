## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_
---
# Golang. Пакет testing. Cleanup

## Назначение

`t.Cleanup()` — механизм выполнения очистки после завершения теста.

Используется для:

- освобождения ресурсов;    
- удаления временных данных;    
- закрытия соединений;    
- выполнения teardown-логики.    
---
## Синтаксис

```go
t.Cleanup(func() {
	// очистка ресурсов
})
```

---
## Пример

```go
package main

import "testing"

func TestDatabase(t *testing.T) {

	db := connectDB()

	t.Cleanup(func() {
		db.Close()
	})

	// тест использует db
}
```

Порядок:

```text
setup
  ↓
test
  ↓
cleanup
```
---
## Особенности

- выполняется после завершения теста;    
- выполняется даже если тест завершился через `t.Fatal`;    
- не выполняется, если процесс завершился через `os.Exit`;    
- можно регистрировать несколько cleanup.   
---
## Несколько Cleanup

```go
package main

import "testing"

func TestCleanup(t *testing.T) {

	t.Cleanup(func() {
		println("first")
	})

	t.Cleanup(func() {
		println("second")
	})
}
```

Порядок выполнения:

```text
second
first
```

Cleanup, как и `defer`, выполняются в обратном порядке (LIFO).

---
## Cleanup в Subtests

```go
package main

import "testing"

func TestAPI(t *testing.T) {

	server := startServer()

	t.Cleanup(func() {
		server.Close()
	})

	t.Run("GET", func(t *testing.T) {
		// использует server
	})
}
```

Cleanup родительского теста выполнится после завершения всех subtests.

---
## Отличие от defer

`defer`:

- выполняется при выходе из текущей функции;    
- не привязан к lifecycle теста.    

`t.Cleanup`:

- выполняется после завершения теста;    
- учитывает subtests;    
- лучше подходит для тестовых ресурсов.    
---
## Где использовать

- удаление временных файлов;    
- закрытие БД соединений;    
- остановка тестовых серверов;    
- очистка mock-объектов;    
- удаление тестовых данных.    

---
## Для собеседования

`t.Cleanup()` — это способ зарегистрировать функцию очистки ресурса после завершения теста. В отличие от `defer`, он интегрирован с `testing` lifecycle, выполняется после теста и поддерживает корректную работу с subtests.