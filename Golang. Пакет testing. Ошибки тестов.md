## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Ошибки тестов

## Назначение

Пакет `testing` предоставляет методы для:

- фиксации ошибки теста;    
- остановки выполнения теста;    
- проверки ожидаемого поведения.    

Основные методы:

- `t.Error()`    
- `t.Errorf()`    
- `t.Fatal()`    
- `t.Fatalf()`    
- `t.Fail()`    
- `t.FailNow()`    

---
## `t.Error`

Помечает тест как упавший, но продолжает выполнение.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	got := 10
	want := 20

	if got != want {
		t.Error("wrong result")
	}

	// выполнение продолжается
}
```

---
## `t.Errorf`

Ошибка с форматированием:

```go
package main

import "testing"

func TestExample(t *testing.T) {

	got := 10
	want := 20

	if got != want {
		t.Errorf(
			"got %d, want %d",
			got,
			want,
		)
	}
}
```

---
## `t.Fatal`

Ошибка с немедленной остановкой теста.

Используется когда дальнейшее выполнение бессмысленно.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	err := initDB()

	if err != nil {
		t.Fatal(err)
	}

	// не выполнится при ошибке
}
```

---
## `t.Fatalf`

`Fatal` + форматирование.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	value := 10

	if value != 20 {
		t.Fatalf(
			"expected 20, got %d",
			value,
		)
	}
}
```

---
## `t.Fail`

Помечает тест как failed, но продолжает выполнение.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	t.Fail()

	// выполнение продолжается
}
```

Обычно используется редко.

---
## `t.FailNow`

Немедленно останавливает выполнение теста.

```go
package main

import "testing"

func TestExample(t *testing.T) {

	t.FailNow()

	// код не выполнится
}
```

Внутри вызывает завершение через `runtime.Goexit`.

---
## Разница Error и Fatal

`Error`:

- тест считается проваленным;    
- выполнение продолжается;    
- можно собрать несколько ошибок.    

`Fatal`:

- тест считается проваленным;    
- выполнение сразу прекращается.    

---
## Разница Fail и FailNow

`Fail`:

- пометить тест как failed;    
- продолжить выполнение.    

`FailNow`:

- пометить тест как failed;    
- остановить текущую goroutine.    

---
## Типичный паттерн проверки

```go
package main

import "testing"

func TestAdd(t *testing.T) {

	got := 2 + 2
	want := 4

	if got != want {
		t.Fatalf(
			"got %d, want %d",
			got,
			want,
		)
	}
}
```

---
## Когда использовать

`Error/Errorf`:

- нужно проверить несколько независимых условий;    
- ошибка не мешает дальнейшим проверкам.    

`Fatal/Fatalf`:

- нет смысла продолжать тест;    
- не удалось создать обязательный ресурс;    
- неправильная подготовка окружения.    

---
## Для собеседования

`t.Error` и `t.Errorf` отмечают тест как failed, но позволяют ему продолжиться. `t.Fatal` и `t.Fatalf` сразу завершают текущий тест. Обычно `Fatal` используют для ошибок setup, когда дальнейшая проверка невозможна.