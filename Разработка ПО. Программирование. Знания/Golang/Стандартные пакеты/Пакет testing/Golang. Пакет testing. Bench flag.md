## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Bench flag

## Назначение

Флаг `-bench` позволяет запускать **benchmark-функции** из пакета `testing`.

Используется для измерения:

- времени выполнения;
    
- количества операций;
    
- аллокаций памяти;
    
- пользовательских метрик.
    

---

## Синтаксис

Запустить все benchmarks:

```bash
go test -bench .
```

`.` — регулярное выражение, которое совпадает со всеми benchmark-именами.

---

Запустить конкретный benchmark:

```bash
go test -bench BenchmarkAdd
```

---

## Пример

Benchmark:

```go
func BenchmarkAdd(b *testing.B) {

	for i := 0; i < b.N; i++ {

		Add(10, 20)

	}

}
```

Запуск:

```bash
go test -bench BenchmarkAdd
```

Пример вывода:

```text
BenchmarkAdd-8    1000000000    0.25 ns/op
```

---

## Формат имени

Benchmark-функции должны начинаться с:

```go
BenchmarkXxx
```

Например:

```go
func BenchmarkJSONMarshal(b *testing.B)
func BenchmarkParser(b *testing.B)
```

---

## Отличие от `-run`

`-run`:

```bash
go test -run TestUser
```

Запускает:

- `TestXxx`
    
- `ExampleXxx`
    

---

`-bench`:

```bash
go test -bench BenchmarkUser
```

Запускает:

- `BenchmarkXxx`
    

---

## Запуск тестов вместе с benchmark

По умолчанию:

```bash
go test -bench .
```

сначала выполняет обычные тесты.

Чтобы не запускать тесты:

```bash
go test -run ^$ -bench .
```

Где:

```text
^$
```

— регулярное выражение, которое ничего не совпадает.

---

## Запуск нескольких benchmark

Например:

```go
func BenchmarkAdd(b *testing.B)
func BenchmarkSub(b *testing.B)
func BenchmarkMul(b *testing.B)
```

Запуск:

```bash
go test -bench "Benchmark(Add|Sub)"
```

Запустит:

```text
BenchmarkAdd
BenchmarkSub
```

---

## Работа с subbenchmarks

Есть:

```go
func BenchmarkMath(b *testing.B) {

	b.Run("Add", func(b *testing.B) {

	})

	b.Run("Sub", func(b *testing.B) {

	})

}
```

Запуск:

```bash
go test -bench "BenchmarkMath/Add"
```

---

## Полезные связанные флаги

Количество времени benchmark:

```bash
go test -bench . -benchtime=5s
```

Количество запусков:

```bash
go test -bench . -count=5
```

Память:

```bash
go test -bench . -benchmem
```

---

## Для собеседования

> Флаг `-bench` используется для запуска benchmark-функций из пакета `testing`. Он принимает регулярное выражение для выбора нужных benchmarks. Например, `go test -bench .` запускает все benchmarks, а `-benchmem` дополнительно показывает количество аллокаций памяти.