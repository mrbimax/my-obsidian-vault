## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark allocations

## Назначение

Benchmark allocations — измерение количества **выделений памяти (heap allocations)** во время выполнения операции.

Основная метрика:

```text
allocs/op
```

Показывает:

- сколько объектов создаётся в куче;
    
- насколько код создаёт нагрузку на Garbage Collector.
    

---

## Включение измерения

Через команду:

```bash
go test -bench=. -benchmem
```

или в коде:

```go
b.ReportAllocs()
```

---

## Пример

```go
package main

import "testing"

func CreateUser() map[string]string {

	user := make(map[string]string)

	user["name"] = "Alex"

	return user
}

func BenchmarkCreateUser(b *testing.B) {

	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		CreateUser()
	}
}
```

Вывод:

```text
BenchmarkCreateUser-8    1000000    500 ns/op    200 B/op    3 allocs/op
```

---

## `allocs/op`

Пример:

```text
3 allocs/op
```

означает:

```text
один вызов CreateUser()
создаёт 3 объекта в heap
```

---

## Почему аллокации важны

Каждая heap allocation:

```text
создание объекта
        ↓
попадание в heap
        ↓
работа Garbage Collector
        ↓
дополнительная нагрузка CPU
```

Много аллокаций могут привести к:

- большему времени GC;
    
- росту latency;
    
- увеличению потребления памяти.
    

---

## Пример оптимизации

До:

```go
func Join(a, b string) string {
	return a + b
}
```

Benchmark:

```text
50 ns/op
32 B/op
1 allocs/op
```

---

После:

```go
func Join(buf []byte, a, b string) []byte {

	buf = append(buf, a...)
	buf = append(buf, b...)

	return buf
}
```

Результат:

```text
20 ns/op
0 B/op
0 allocs/op
```

---

## Измерение только аллокаций

Можно использовать:

```go
testing.AllocsPerRun()
```

Пример:

```go
package main

import (
	"testing"
)

func TestAllocations(t *testing.T) {

	allocs := testing.AllocsPerRun(
		1000,
		func() {
			CreateUser()
		},
	)

	t.Logf(
		"allocations: %.2f",
		allocs,
	)
}
```

---

## Контроль количества аллокаций

Например:

```go
func BenchmarkNoAlloc(b *testing.B) {

	b.ReportAllocs()

	var x int

	for i := 0; i < b.N; i++ {
		x++
	}

	_ = x
}
```

Возможный результат:

```text
0 B/op
0 allocs/op
```

---

## Связь с escape analysis

Не каждая переменная создаёт heap allocation.

Компилятор анализирует:

```bash
go test -gcflags="-m"
```

Пример:

```text
moved to heap: variable
```

означает:

```text
переменная больше не может жить на stack
→ отправлена в heap
```

---

## Что часто оптимизируют в Go backend

- лишние `[]byte` ↔ `string` преобразования;
    
- создание временных структур;
    
- JSON serialization;
    
- конкатенацию строк;
    
- создание объектов в горячих циклах;
    
- лишние копирования данных.
    

---

## Для собеседования

> `allocs/op` в benchmark показывает количество heap-аллокаций на одну операцию. Вместе с `B/op` эта метрика помогает находить код, создающий лишнюю нагрузку на Garbage Collector. Для анализа используют `go test -benchmem` или `b.ReportAllocs()`. При оптимизации смотрят не только на скорость (`ns/op`), но и на количество аллокаций.