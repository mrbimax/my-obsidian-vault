## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark memory

## Назначение

Benchmark memory позволяет измерять:

- сколько памяти выделяет код;
    
- сколько аллокаций происходит;
    
- влияние изменений на нагрузку GC.
    

Основные метрики:

- `B/op`
    
- `allocs/op`
    

---

## Включение измерения памяти

Через флаг:

```bash
go test -bench=. -benchmem
```

или внутри benchmark:

```go
b.ReportAllocs()
```

---

## Пример

```go
package main

import "testing"

func CreateSlice() []int {
	return make([]int, 100)
}

func BenchmarkCreateSlice(b *testing.B) {

	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		CreateSlice()
	}
}
```

Вывод:

```text
BenchmarkCreateSlice-8    1000000    500 ns/op    800 B/op    1 allocs/op
```

---

## `B/op`

Количество выделенной памяти на одну операцию.

Пример:

```text
800 B/op
```

означает:

```text
каждый вызов CreateSlice()
выделяет 800 байт
```

---

## `allocs/op`

Количество аллокаций на одну операцию.

Пример:

```text
1 allocs/op
```

означает:

```text
один вызов функции
создаёт один объект в heap
```

---

## Пример с несколькими аллокациями

```go
package main

import "testing"

func Allocate() []string {

	a := make([]string, 10)

	for i := range a {
		a[i] = "hello"
	}

	return a
}

func BenchmarkAllocate(b *testing.B) {

	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		Allocate()
	}
}
```

Возможный результат:

```text
BenchmarkAllocate-8
1000000
700 ns/op
512 B/op
11 allocs/op
```

---

## Поиск лишних аллокаций

Было:

```go
func Concat(a, b string) string {
	return a + b
}
```

Benchmark:

```text
20 B/op
1 allocs/op
```

После оптимизации:

```go
func Concat(buf []byte, a, b string) []byte {
	buf = append(buf, a...)
	buf = append(buf, b...)
	return buf
}
```

Можно получить:

```text
0 B/op
0 allocs/op
```

---

## Измерение только нужного участка

Подготовку данных исключают:

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareLargeData()

	b.ResetTimer()
	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Измеряется только:

```text
Process(data)
```

---

## Проверка количества аллокаций

Для точных проверок:

```go
package main

import (
	"testing"
)

func TestAllocations(t *testing.T) {

	allocs := testing.AllocsPerRun(
		100,
		func() {
			CreateSlice()
		},
	)

	t.Log(allocs)
}
```

`AllocsPerRun`:

- запускает функцию несколько раз;
    
- считает среднее количество аллокаций.
    

---

## Связь с GC

Больше аллокаций:

```text
больше объектов в heap
        ↓
больше работы GC
        ↓
выше latency
        ↓
хуже throughput
```

Поэтому при оптимизации backend часто смотрят:

```text
ns/op
+
B/op
+
allocs/op
```

---

## Для собеседования

> Benchmark memory в Go используется для анализа влияния кода на память. Через `-benchmem` или `b.ReportAllocs()` можно увидеть `B/op` — количество выделенной памяти, и `allocs/op` — количество аллокаций на операцию. Это помогает находить лишние выделения памяти и уменьшать нагрузку на Garbage Collector.