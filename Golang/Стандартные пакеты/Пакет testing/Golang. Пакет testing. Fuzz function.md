## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz function

## Назначение

`Fuzz function` — функция, которая описывает **логику проверки fuzz-теста**.

Она получает автоматически сгенерированные входные данные от fuzzing engine и проверяет, что программа:

- не вызывает `panic`;
    
- не нарушает инварианты;
    
- возвращает корректный результат.
    

---

## Синтаксис

```go
func FuzzXxx(f *testing.F) {

	f.Fuzz(func(t *testing.T, input Type) {

		// проверка

	})

}
```

---

## Простой пример

```go
package main

import "testing"

func IsValidName(name string) bool {
	return len(name) > 0
}

func FuzzIsValidName(f *testing.F) {

	f.Add("Max")
	f.Add("")

	f.Fuzz(func(t *testing.T, name string) {

		result := IsValidName(name)

		if result && len(name) == 0 {
			t.Errorf("empty name marked as valid")
		}

	})

}
```

---

## Что происходит внутри

```text
f.Add()
   |
   ↓
seed corpus

   |
   ↓

f.Fuzz()

   |
   ↓

fuzzer генерирует input

   |
   ↓

вызывает функцию

   |
   ↓

ошибка / panic?

```

---

## Аргументы fuzz function

Первый аргумент всегда:

```go
t *testing.T
```

Дальше идут данные для проверки:

```go
f.Fuzz(func(
	t *testing.T,
	name string,
	age int,
) {

})
```

Соответствуют:

```go
f.Add(
	"Max",
	25,
)
```

---

## Несколько аргументов

```go
package main

import "testing"

func CreateUser(name string, age int) error {
	if age < 0 {
		return nil
	}

	return nil
}

func FuzzCreateUser(f *testing.F) {

	f.Add("Max", 25)

	f.Fuzz(func(
		t *testing.T,
		name string,
		age int,
	) {

		CreateUser(name, age)

	})

}
```

Fuzzer будет генерировать:

```text
name = ""
name = "aaaa"
name = random bytes

age = 0
age = -1
age = MaxInt
```

---

## Проверка инвариантов

Fuzzing обычно проверяет не конкретный ответ, а правило.

Например:

```go
func FuzzReverse(f *testing.F) {

	f.Add("hello")

	f.Fuzz(func(
		t *testing.T,
		input string,
	) {

		result := Reverse(input)

		if Reverse(result) != input {
			t.Errorf("reverse property broken")
		}

	})

}
```

Проверяется свойство:

```text
Reverse(Reverse(x)) == x
```

А не:

```text
Reverse("hello") == "olleh"
```

---

## Panic считается ошибкой

Например:

```go
func FuzzParser(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Если:

```go
Parse([]byte{})
```

вызывает:

```go
panic("empty input")
```

Go сохранит этот input.

---

## Запуск

Обычный тест:

```bash
go test
```

Запускает только seed:

```text
f.Add(...)
```

---

Fuzz режим:

```bash
go test -fuzz=FuzzParser
```

---

С ограничением времени:

```bash
go test -fuzz=FuzzParser -fuzztime=30s
```

---

## Отличие от Test функции

Обычный тест:

```go
func TestParse(t *testing.T)
```

Сам задаёт данные:

```go
Parse("hello")
Parse("")
Parse("{}")
```

---

Fuzz:

```go
func FuzzParse(f *testing.F)
```

Передаёт данные генератору:

```text
seed
 ↓
mutations
 ↓
тысячи вариантов
```

---

## Важный момент

`f.Fuzz()` не генерирует тесты с ожидаемым результатом.

Плохо:

```go
if result != "expected" {
	t.Fail()
}
```

Чаще правильно:

```go
if err == nil && input == invalid {
	t.Fail()
}
```

или проверять свойства:

```text
после сериализации данные должны восстановиться
парсер не должен падать
размер не должен стать отрицательным
```

---

## Для собеседования

> Fuzz function — это функция, передаваемая в `f.Fuzz()`, которая получает сгенерированные fuzzing engine данные через `*testing.T`. Внутри неё описываются инварианты или проверки корректности. В отличие от unit-тестов, fuzz-тест обычно не проверяет конкретный ожидаемый результат, а ищет входные данные, которые приводят к panic или нарушению условий.