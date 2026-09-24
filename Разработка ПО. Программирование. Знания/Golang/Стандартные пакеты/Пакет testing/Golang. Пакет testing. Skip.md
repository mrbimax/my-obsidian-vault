## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Skip

## Назначение

Методы `t.Skip()` и `t.Skipf()` позволяют **пропустить выполнение теста**.

Используются, когда тест временно нельзя выполнить.

---
## `t.Skip`

Пропускает тест.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	t.Skip("not implemented yet")

	// код ниже не выполнится
}
```

---
## `t.Skipf`

То же самое, но с форматированием.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	version := "v2"

	t.Skipf("unsupported version: %s", version)
}
```

---
## Когда используют

- функциональность ещё не реализована;    
- отсутствует внешний сервис;    
- тест предназначен только для определённой ОС;    
- тест запускается только в определённых условиях.    

---
## Пример

```go
package main

import (
	"runtime"
	"testing"
)

func TestWindowsOnly(t *testing.T) {

	if runtime.GOOS != "windows" {
		t.Skip("Windows only")
	}

	// тест
}
```

---
## Вывод

```text
=== RUN   TestWindowsOnly
    test.go:10: Windows only
--- SKIP: TestWindowsOnly
```

---
## Отличие от `t.Fatal`

`t.Skip`:

- тест пропущен;    
- не считается ошибкой.    

`t.Fatal`:

- тест завершился с ошибкой;    
- считается failed.    

---
## Для собеседования

> `t.Skip()` и `t.Skipf()` используются для условного пропуска тестов. После вызова выполнение теста прекращается, но тест помечается как **SKIP**, а не **FAIL**. Обычно применяются для платформозависимых тестов или при отсутствии необходимых внешних условий.