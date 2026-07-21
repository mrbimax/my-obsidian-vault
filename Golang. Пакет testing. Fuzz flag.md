## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz flag

## Назначение

Флаг `-fuzz` запускает **fuzz-тесты** из пакета `testing`.

Используется для:

- генерации большого количества входных данных;
    
- поиска `panic`;
    
- поиска нарушения инвариантов;
    
- расширения fuzz corpus.
    

---

## Синтаксис

Запустить fuzz-функцию:

```bash
go test -fuzz=FuzzParse
```

Где:

```go
func FuzzParse(f *testing.F)
```

---

## Пример

Fuzz-тест:

```go
func FuzzParse(f *testing.F) {

	f.Add([]byte("hello"))

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Запуск:

```bash
go test -fuzz=FuzzParse
```

Go начинает:

```text
seed corpus
      |
      ↓
mutation
      |
      ↓
запуск функции
      |
      ↓
проверка panic/error/coverage
```

---

## Запуск всех fuzz-тестов

```bash
go test -fuzz=.
```

`.` — regexp, который совпадает со всеми fuzz-функциями.

---

## Fuzz запускается вместо обычных тестов?

Нет.

Команда:

```bash
go test -fuzz=FuzzParse
```

сначала запускает обычные тесты:

```text
TestXxx
ExampleXxx
```

а затем начинает fuzzing.

Чтобы запускать только fuzz:

```bash
go test -run ^$ -fuzz=FuzzParse
```

---

## Ограничение времени

По умолчанию fuzz работает ограниченное время.

Изменить:

```bash
go test -fuzz=FuzzParse -fuzztime=30s
```

или:

```bash
go test -fuzz=FuzzParse -fuzztime=1000x
```

Где:

```text
1000x
```

означает выполнить 1000 fuzz-итераций.

---

## Запуск конкретного fuzz case

Найденные случаи хранятся:

```text
testdata/
└── fuzz/
    └── FuzzParse/
```

Они запускаются обычным:

```bash
go test
```

как регрессионные тесты.

---

## Отличие от `-run`

`-run`:

```bash
go test -run TestUser
```

Запускает:

- unit-тесты;
    
- examples.
    

---

`-fuzz`:

```bash
go test -fuzz=FuzzUser
```

Запускает:

- fuzzing engine;
    
- генерацию новых входных данных.
    

---

## Связанные флаги

Ограничение времени:

```bash
-fuzztime=10s
```

Выбор пакета:

```bash
go test ./parser -fuzz=FuzzParse
```

Запуск без обычных тестов:

```bash
go test -run ^$ -fuzz=FuzzParse
```

---

## Для собеседования

> Флаг `-fuzz` запускает fuzz-тесты. Он передаёт управление fuzzing engine, который генерирует и мутирует входные данные, отслеживает покрытие и сохраняет интересные случаи в corpus. Параметр `-fuzztime` задаёт длительность fuzzing-сессии. Найденные случаи потом автоматически используются как регрессионные тесты через обычный `go test`.