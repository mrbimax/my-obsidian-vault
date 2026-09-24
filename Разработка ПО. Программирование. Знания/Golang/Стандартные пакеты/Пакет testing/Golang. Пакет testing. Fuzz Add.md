## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz Add

## Назначение

`f.Add()` добавляет **seed corpus** — начальные входные данные для fuzz-теста.

Используется для:

- задания известных хороших примеров;
    
- покрытия важных случаев;
    
- улучшения старта fuzzing.
    

Важно:

`f.Add()` **не является проверкой ожидаемого результата**, как в table-driven tests.

---

## Синтаксис

```go
f.Add(value1, value2, ...)
```

Типы аргументов должны совпадать с аргументами fuzz-функции.

---

## Пример

```go
package main

import (
	"testing"
)

func FuzzReverse(f *testing.F) {

	f.Add("hello")
	f.Add("")
	f.Add("golang")

	f.Fuzz(func(t *testing.T, input string) {

		result := Reverse(input)

		if len(result) != len(input) {
			t.Errorf("length changed")
		}

	})
}
```

Здесь:

```go
f.Add("hello")
```

означает:

> Начни fuzzing с этой строки и попробуй создавать новые варианты.

---

## Несколько аргументов

```go
func FuzzUser(f *testing.F) {

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

`f.Add()` должен соответствовать сигнатуре:

```text
f.Add()
        ↓
Fuzz function
        ↓
name string
age int
```

---

## Seed и генерация

Например:

```go
f.Add("hello")
```

Fuzzer может создать:

```text
hello
helo
Hello
hello123
hello\x00
"" 
```

---

## Отличие от table-driven tests

Table test:

```go
tests := []struct{
	input string
	want bool
}{
	{"hello", true},
}
```

Проверяет:

```text
конкретный input
+
конкретный expected result
```

---

Fuzz:

```go
f.Add("hello")
```

Использует:

```text
input
↓
генерация новых input
↓
поиск ошибок
```

---

## Seed без `f.Add`

Можно написать:

```go
func FuzzParse(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})
}
```

Но тогда fuzzer начинает с пустого набора seed.

Обычно лучше добавить:

```go
f.Add([]byte(`{"id":1}`))
```

---

## Важное ограничение

`f.Add()` принимает только поддерживаемые fuzz-типы:

```text
string
[]byte
bool
int
int8
int16
int32
int64
uint
uint8
uint16
uint32
uint64
rune
byte
```

Нельзя:

```go
f.Add(User{})
```

Нужно:

```go
f.Add(
	"Max",
	25,
)
```

или:

```go
f.Add(
	[]byte(`{"name":"Max"}`),
)
```

---

## Seed corpus из файлов

Альтернатива `f.Add()`:

```text
testdata/
└── fuzz/
    └── FuzzParser/
        ├── valid.json
        └── invalid.json
```

Go автоматически загрузит эти значения.

---

## Для собеседования

> `f.Add()` добавляет seed corpus для fuzz-теста. В отличие от table-driven tests, это не набор проверяемых кейсов с ожидаемым результатом, а стартовые данные для генератора. Fuzzer использует их как основу для мутаций и поиска новых входных данных, которые могут привести к ошибкам или новым путям выполнения.