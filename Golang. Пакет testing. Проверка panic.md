## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Проверка panic

## Назначение

Иногда функция **должна вызвать `panic`**. Для проверки такого поведения используют `defer` + `recover()`.

В пакете `testing` нет отдельного метода для проверки `panic`.

---
## Проверка, что panic произошёл

```go
package main

import "testing"

func MustPositive(n int) {
	if n < 0 {
		panic("negative number")
	}
}

func TestMustPositive(t *testing.T) {
	defer func() {
		if recover() == nil {
			t.Fatal("expected panic")
		}
	}()
	MustPositive(-1)
}
```

---
## Проверка текста panic

```go
package main

import "testing"

func TestMustPositive(t *testing.T) {
	defer func() {
		got := recover()
		if got != "negative number" {
			t.Fatalf(
				"got %v, want %q",
				got,
				"negative number",
			)
		}
	}()
	MustPositive(-1)
}
```

---
## Проверка, что panic **не произошло**

```go
package main

import "testing"

func TestMustPositive(t *testing.T) {
	defer func() {
		if recover() != nil {
			t.Fatal("unexpected panic")
		}
	}()
	MustPositive(10)
}
```

---
## Универсальный helper

```go
package main

import "testing"

func shouldPanic(t *testing.T, fn func()) {
	t.Helper()
	defer func() {
		if recover() == nil {
			t.Fatal("expected panic")
		}
	}()
	fn()
}
```

Использование:

```go
func TestMustPositive(t *testing.T) {
	shouldPanic(t, func() {
		MustPositive(-1)
	})
}
```

---
## Когда проверяют panic

- функции `Must...`    
- некорректные аргументы    
- нарушения внутренних инвариантов    
- код, который по контракту должен завершаться через `panic`    

---
## Для собеседования

> В Go нет встроенного `assert.Panic`. Проверка выполняется через `defer` и `recover()`: если `recover()` вернул `nil`, значит `panic` не произошло. При необходимости можно проверить и значение, переданное в `panic`.