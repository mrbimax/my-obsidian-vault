## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz seed corpus

## Назначение

Seed corpus — это **набор начальных входных данных**, которые fuzzing использует как основу для генерации новых случаев.

Используется для:

- передачи важных примеров в fuzzing;
    
- покрытия известных граничных случаев;
    
- ускорения поиска ошибок.
    

---

## Добавление seed через `f.Add`

Самый простой способ:

```go
func FuzzParser(f *testing.F) {

	f.Add([]byte(`{"name":"Alex"}`))
	f.Add([]byte(`{}`))
	f.Add([]byte(``))

	f.Fuzz(func(t *testing.T, data []byte) {

		Parse(data)

	})
}
```

`f.Add()` добавляет стартовые значения.

---

## Как работает

```text
seed corpus
    |
    ↓
f.Add(...)
    |
    ↓
fuzzer изменяет данные
    |
    ↓
новые входы
    |
    ↓
поиск panic / ошибки
```

Например:

Seed:

```json
{"name":"Alex"}
```

Fuzzer может создать:

```json
{"name":""}
```

или:

```json
{"name":null}
```

или:

```text
битые данные
```

---

## Внешний corpus

Кроме `f.Add()` можно хранить данные в файлах.

Структура:

```text
testdata/
└── fuzz/
    └── FuzzParser/
        ├── file1
        ├── file2
        └── file3
```

Пример:

```text
parser/
├── parser.go
├── parser_test.go
└── testdata/
    └── fuzz/
        └── FuzzParser/
            ├── valid.json
            └── broken.json
```

Go автоматически загрузит эти файлы.

---

## Пример fuzz с corpus

Код:

```go
package parser

import "testing"

func FuzzParser(f *testing.F) {

	f.Fuzz(func(t *testing.T, data []byte) {

		Parse(data)

	})
}
```

Corpus:

```text
testdata/fuzz/FuzzParser/valid.json
```

Содержимое:

```json
{
	"name": "Max"
}
```

---

## Seed corpus vs generated corpus

Есть два типа данных:

### Seed corpus

Создаётся разработчиком:

```go
f.Add("hello")
```

или:

```text
testdata/fuzz/FuzzXxx/
```

Назначение:

- стартовые примеры;
    
- важные сценарии.
    

---

### Generated corpus

Создаётся самим fuzzing:

```text
testdata/fuzz/FuzzParser/
    ├── 8c91a2
    └── 91ab33
```

Когда найден новый интересный случай:

- Go сохраняет его;
    
- использует в следующих запусках.
    

---

## Пример с несколькими типами

```go
func FuzzUser(f *testing.F) {

	f.Add(
		"Alex",
		25,
		true,
	)

	f.Fuzz(func(
		t *testing.T,
		name string,
		age int,
		active bool,
	) {

		CreateUser(name, age, active)

	})
}
```

---

## Важные правила

Seed должен быть:

- валидным примером;
    
- небольшим;
    
- покрывать важные случаи.
    

Хорошие seed:

```text
empty input
minimum value
maximum value
обычный объект
сломанный объект
граничные значения
```

---

## Для собеседования

> Seed corpus — это начальный набор входных данных для fuzzing. В Go его можно задавать через `f.Add()` или хранить в `testdata/fuzz/FuzzXxx`. Fuzzer использует эти значения как основу для генерации новых входов. Найденные интересные случаи сохраняются и становятся частью corpus для следующих запусков.