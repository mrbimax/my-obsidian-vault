## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. List tests

## Назначение

Флаг `-list` выводит список тестов, benchmarks и examples, которые доступны для запуска.

Используется для:

- просмотра структуры тестов;
    
- поиска нужного имени;
    
- проверки regexp для `-run`, `-bench`, `-fuzz`.
    

---

## Синтаксис

```bash
go test -list .
```

`.` — регулярное выражение, совпадающее со всеми именами.

---

## Пример

Есть тесты:

```go
func TestAdd(t *testing.T) {}

func TestSub(t *testing.T) {}

func TestUser_Create(t *testing.T) {}
```

Команда:

```bash
go test -list .
```

Вывод:

```text
TestAdd
TestSub
TestUser_Create
ok      example
```

---

## Поиск конкретных тестов

Можно использовать regexp:

```bash
go test -list User
```

Вывод:

```text
TestUser_Create
```

---

## Subtests

Важно:

`-list` показывает только верхнеуровневые тесты.

Например:

```go
func TestMath(t *testing.T) {

	t.Run("Add", func(t *testing.T) {})

	t.Run("Sub", func(t *testing.T) {})

}
```

Команда:

```bash
go test -list .
```

Вывод:

```text
TestMath
```

А не:

```text
TestMath/Add
TestMath/Sub
```

---

## Что показывает `-list`

Показывает:

- `TestXxx`
    
- `BenchmarkXxx`
    
- `ExampleXxx`
    
- `FuzzXxx`
    

---

## Пример с benchmark

Есть:

```go
func BenchmarkParser(b *testing.B) {}
```

Команда:

```bash
go test -list .
```

Вывод:

```text
BenchmarkParser
```

---

## Пример с Example

Есть:

```go
func ExampleAdd() {}
```

Вывод:

```text
ExampleAdd
```

---

## Пример с fuzz

Есть:

```go
func FuzzParse(f *testing.F) {}
```

Вывод:

```text
FuzzParse
```

---

## Отличие от других флагов

`-list`

```bash
go test -list .
```

Только показывает.

---

`-run`

```bash
go test -run TestUser
```

Запускает найденные тесты.

---

`-bench`

```bash
go test -bench BenchmarkParser
```

Запускает benchmark.

---

`-fuzz`

```bash
go test -fuzz FuzzParse
```

Запускает fuzzing.

---

## Практический сценарий

Посмотреть доступные тесты:

```bash
go test -list .
```

Найти нужный:

```bash
go test -list User
```

Запустить:

```bash
go test -run TestUser_Create
```

---

## Для собеседования

> Флаг `-list` используется для вывода списка доступных тестов, benchmarks, examples и fuzz-функций. Он принимает регулярное выражение для фильтрации имён, но не запускает найденные тесты. Полезен для изучения структуры тестового набора и проверки правильности фильтров `-run`, `-bench` и `-fuzz`.