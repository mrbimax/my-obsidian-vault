## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. ReportMetric

## Назначение

`b.ReportMetric()` позволяет добавить **собственную метрику** в вывод benchmark.

Используется, когда стандартных метрик (`ns/op`, `B/op`, `allocs/op`) недостаточно.

---

## Синтаксис

```go
b.ReportMetric(value float64, unit string)
```

Параметры:

- `value` — числовое значение метрики;
    
- `unit` — название единицы измерения.
    

---

## Пример

```go
package main

import (
	"testing"
)

func BenchmarkProcess(b *testing.B) {

	for i := 0; i < b.N; i++ {
		Process()
	}

	b.ReportMetric(100, "items/op")
}
```

Вывод:

```text
BenchmarkProcess-8    1000000    500 ns/op    100 items/op
```

---

## Пример: количество обработанных объектов

```go
package main

import "testing"

func BenchmarkParser(b *testing.B) {

	data := make([]byte, 1024)

	for i := 0; i < b.N; i++ {
		Parse(data)
	}

	items := float64(b.N * len(data))

	b.ReportMetric(items/float64(b.N), "bytes/op")
}
```

Вывод:

```text
BenchmarkParser-8    1000000    500 ns/op    1024 bytes/op
```

---

## Пример: собственная скорость

Например, измерить количество запросов в секунду:

```go
package main

import "testing"

func BenchmarkRequests(b *testing.B) {

	for i := 0; i < b.N; i++ {
		Request()
	}

	rps := float64(b.N) / b.Elapsed().Seconds()

	b.ReportMetric(rps, "req/s")
}
```

---

## Сброс стандартной метрики

Можно удалить стандартный вывод:

```go
b.ReportMetric(0, "ns/op")
```

Например:

```go
func BenchmarkExample(b *testing.B) {

	for i := 0; i < b.N; i++ {
		work()
	}

	b.ReportMetric(50, "MB/s")
	b.ReportMetric(1000, "ops/s")
	b.ReportMetric(0, "ns/op")
}
```

---

## Отличие от `ReportAllocs`

`ReportAllocs()`:

```go
b.ReportAllocs()
```

добавляет встроенные метрики:

```text
B/op
allocs/op
```

`ReportMetric()`:

```go
b.ReportMetric(...)
```

добавляет произвольные метрики.

---

## Когда использовать

Полезно для измерения:

- пропускной способности;
    
- количества обработанных элементов;
    
- размера данных;
    
- бизнес-метрик;
    
- специализированных операций.
    

Примеры:

```text
MB/s
requests/sec
records/op
messages/sec
```

---

## Для собеседования

> `b.ReportMetric()` позволяет добавлять пользовательские метрики в benchmark. Например, кроме времени выполнения можно вывести количество обработанных элементов в секунду, размер данных или другую специфическую характеристику. `ReportAllocs()` — частный случай стандартных метрик памяти, а `ReportMetric()` используется для своих измерений.

