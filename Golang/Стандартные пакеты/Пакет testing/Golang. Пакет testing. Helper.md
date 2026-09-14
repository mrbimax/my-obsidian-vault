## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Helper

## Назначение

`t.Helper()` помечает функцию как **вспомогательную**.

Если внутри неё произойдёт ошибка, `testing` покажет место **вызова helper**, а не сам helper.

Используется для:

- устранения дублирования кода;    
- создания функций проверки (`assert`, `check`);    
- повышения читаемости тестов.    

---
## Синтаксис

```go
t.Helper()
```

---
## Без `t.Helper`

```go
package main

import "testing"

func checkEqual(t *testing.T, got, want int) {
	if got != want {
		t.Fatalf("got %d, want %d", got, want)
	}
}

func TestAdd(t *testing.T) {
	checkEqual(t, 2+2, 5)
}
```

Ошибка будет указывать на строку внутри `checkEqual()`.

---
## С `t.Helper`

```go
package main

import "testing"

func checkEqual(t *testing.T, got, want int) {
	t.Helper()
	if got != want {
		t.Fatalf("got %d, want %d", got, want)
	}
}

func TestAdd(t *testing.T) {
	checkEqual(t, 2+2, 5)
}
```

Теперь ошибка будет указывать на строку:

```go
checkEqual(t, 2+2, 5)
```

а не на код внутри helper.

---
## Типичный helper

```go
package main

import "testing"

func checkNoError(t *testing.T, err error) {
	t.Helper()
	if err != nil {
		t.Fatal(err)
	}
}
```

Использование:

```go
func TestOpen(t *testing.T) {
	err := Open()
	checkNoError(t, err)
}
```

---
## Когда использовать

- проверки результатов;    
- проверки ошибок;    
- создание собственных `assert`;    
- общие функции для нескольких тестов.    

---
## Для собеседования

> `t.Helper()` помечает функцию как вспомогательную. При ошибке `testing` скрывает её из стека вызовов и показывает место вызова helper, что делает сообщения об ошибках более понятными.