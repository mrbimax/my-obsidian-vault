## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Logging

## Назначение

Логирование в тестах используется для вывода дополнительной информации при выполнении тестов.

Основные методы:

- `t.Log()`    
- `t.Logf()`    

---
## `t.Log`

Выводит сообщение в лог теста.

```go
package main

import "testing"

func TestExample(t *testing.T) {
	value := 42
	t.Log("value:", value)
}
```

---
## `t.Logf`

Форматированный вывод:

```go
package main

import "testing"

func TestExample(t *testing.T) {
	value := 42
	t.Logf("value = %d", value)
}
```

---
## Когда видны логи

Обычный запуск:

```bash
go test
```

Логи успешных тестов обычно не показываются.

Подробный режим:

```bash
go test -v
```

Вывод:

```text
=== RUN   TestExample
    example_test.go:10: value = 42
--- PASS: TestExample
```
---
## Логи при ошибке

Если тест упал:

```go
package main

import "testing"

func TestExample(t *testing.T) {
	t.Log("before error")
	t.Fatal("test failed")
}
```

Логи будут показаны даже без `-v`.

---
## Логирование в subtests

```go
package main

import "testing"

func TestUsers(t *testing.T) {
	t.Run("create", func(t *testing.T) {
		t.Log("creating user")
	})
	t.Run("delete", func(t *testing.T) {
		t.Log("deleting user")
	})
}
```
---

## Отличие от `fmt.Println`

`fmt.Println`:

- не связан с тестовым lifecycle;    
- выводится всегда;    
- хуже подходит для тестов.    

`t.Log`:

- привязан к конкретному тесту;    
- отображается с именем теста;    
- управляется флагом `-v`.    

---
## Для собеседования

`t.Log()` и `t.Logf()` — встроенные методы `testing.T` для вывода диагностической информации из тестов. Логи успешных тестов отображаются при `go test -v`, а при падении теста показываются автоматически. Лучше использовать их вместо `fmt.Println`, так как они интегрированы с тестовым раннером.