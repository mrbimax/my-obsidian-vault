## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzzing

## Назначение

Fuzzing — автоматическое тестирование функции с большим количеством случайных входных данных.

Цель:

- найти неожиданные ошибки;
    
- обнаружить panic;
    
- проверить граничные случаи;
    
- найти проблемы с парсерами и обработчиками данных.
    

В Go fuzzing встроен в пакет `testing` начиная с **Go 1.18**.

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

import (
	"strings"
	"testing"
)

func Reverse(s string) string {

	r := []rune(s)

	for i, j := 0, len(r)-1; i < j; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}

	return string(r)
}

func FuzzReverse(f *testing.F) {

	f.Add("hello")

	f.Fuzz(func(t *testing.T, input string) {

		result := Reverse(input)

		if len(result) != len(input) {
			t.Errorf("length changed")
		}

	})
}
```

---

## Seed corpus (`f.Add`)

`f.Add()` добавляет начальные данные.

```go
f.Add("hello")
f.Add("")
f.Add("12345")
```

Они используются перед генерацией случайных значений.

---

## Запуск fuzz-теста

Обычный запуск:

```bash
go test
```

Запускает только seed cases.

---

Fuzzing:

```bash
go test -fuzz=FuzzReverse
```

или:

```bash
go test -fuzz=.
```

---

## Ограничение времени

По умолчанию fuzzing работает долго.

Ограничение:

```bash
go test -fuzz=FuzzReverse -fuzztime=30s
```

Пример:

```bash
go test -fuzz=FuzzParser -fuzztime=10m
```

---

## Что происходит внутри

Fuzzing:

```text
seed inputs
      |
      ↓
генерация новых данных
      |
      ↓
запуск функции
      |
      ↓
panic / ошибка?
      |
      ↓
сохранение минимального примера
```

---

## Поиск panic

Пример:

```go
package main

import "testing"

func Parse(data []byte) {

	if len(data) == 0 {
		panic("empty data")
	}
}

func FuzzParse(f *testing.F) {

	f.Fuzz(func(t *testing.T, data []byte) {

		Parse(data)

	})
}
```

Fuzzer найдёт:

```text
[]byte{}
```

и сохранит его.

---

## Corpus

Найденные случаи сохраняются:

```text
testdata/
└── fuzz/
    └── FuzzParse/
        └── abc123
```

При следующих запусках они используются снова.

---

## Типы данных для fuzzing

Поддерживаются:

```go
string
[]byte
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
bool
```

Пример:

```go
f.Fuzz(func(
	t *testing.T,
	a int,
	b string,
	c []byte,
) {

})
```

---

## Отличие от обычных тестов

Unit test:

```go
func TestParser(t *testing.T)
```

Проверяет:

```text
конкретные случаи
```

---

Fuzz test:

```go
func FuzzParser(f *testing.F)
```

Проверяет:

```text
тысячи неизвестных случаев
```

---

## Где чаще используют

Особенно полезен для:

- JSON/XML парсеров;
    
- HTTP handlers;
    
- декодеров;
    
- криптографии;
    
- работы с файлами;
    
- сериализации;
    
- пользовательского ввода.
    

---

## Fuzzing в CI

Обычно не запускают бесконечно на каждом PR.

Частая схема:

PR:

```text
go test ./...
go test -race ./...
```

Nightly:

```text
go test -fuzz=. -fuzztime=10m
```

---

## Для собеседования

> Fuzzing в Go появился в версии 1.18 и встроен в пакет `testing`. Это способ автоматического поиска ошибок путём генерации большого количества входных данных. Fuzz-тесты используют `testing.F`, seed cases через `f.Add()` и сохраняют найденные проблемные случаи в corpus. Чаще всего применяются для парсеров, сериализации и проверки обработки некорректного пользовательского ввода.