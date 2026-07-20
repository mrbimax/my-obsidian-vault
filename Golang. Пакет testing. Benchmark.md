## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Benchmark

## Назначение

Benchmark — механизм измерения производительности кода.

Используется для:

- сравнения алгоритмов;    
- оценки скорости функций;    
- отслеживания регрессий производительности.    

---
## Сигнатура

```go
func BenchmarkXxx(b *testing.B)
```

---
## Пример

```go
package main

import "testing"

func Sum(a, b int) int {
	return a + b
}
func BenchmarkSum(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Sum(1, 2)
	}
}
```

---
## Запуск

Все benchmark:

```bash
go test -bench=.
```

Конкретный benchmark:

```bash
go test -bench=BenchmarkSum
```

---
## Что такое `b.N`

`b.N` — количество итераций.

Go автоматически увеличивает его, пока результаты измерения не станут достаточно точными.

```go
for i := 0; i < b.N; i++ {
	// измеряемый код
}
```

Не нужно задавать количество итераций вручную.

---
## Пример вывода

```text
BenchmarkSum-8    1000000000    0.28 ns/op
```

Где:

- `BenchmarkSum` — имя benchmark;    
- `8` — значение `GOMAXPROCS`;    
- `1000000000` — число выполненных итераций (`b.N`);    
- `0.28 ns/op` — среднее время одной операции.    

---
## Измерение памяти

Запуск:

```bash
go test -bench=. -benchmem
```

Пример вывода:

```text
BenchmarkSum-8
1000000000
0.28 ns/op
0 B/op
0 allocs/op
```

Дополнительно показывается:

- `B/op` — байт выделяется на операцию;    
- `allocs/op` — количество аллокаций на операцию.    

---

## Исключение подготовки из измерений

Подготовку данных обычно не измеряют.

```go
func BenchmarkExample(b *testing.B) {

	data := prepare()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		process(data)
	}
}
```

---
## Остановка таймера

Если внутри benchmark есть код, который не нужно измерять:

```go
b.StopTimer()

// подготовка

b.StartTimer()
```

---
## Подтесты Benchmark

```go
func BenchmarkSort(b *testing.B) {

	b.Run("small", func(b *testing.B) {
		// benchmark
	})

	b.Run("large", func(b *testing.B) {
		// benchmark
	})
}
```

---
## Best practices

- измерять только нужный код;    
- использовать `b.ResetTimer()` после подготовки;    
- запускать с `-benchmem` для анализа аллокаций;    
- сравнивать несколько реализаций через `b.Run()`.    

---
## Для собеседования

> Benchmark в Go — это специальные функции `BenchmarkXxx`, которые измеряют производительность кода. Они запускаются через `go test -bench`, автоматически подбирают число итераций (`b.N`) и могут дополнительно показывать количество аллокаций и использование памяти с помощью флага `-benchmem`.