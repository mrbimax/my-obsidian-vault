## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. ReportAllocs

## Назначение

`b.ReportAllocs()` включает сбор информации о выделении памяти в benchmark.

Показывает:

- количество аллокаций;
    
- количество выделенной памяти.
    

## Синтаксис

```go
b.ReportAllocs()
```

---

## Пример

```go
package main

import (
	"testing"
)

func BenchmarkString(b *testing.B) {

	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		_ = []byte("hello")
	}
}
```

---

## Запуск

```bash
go test -bench=. 
```

или:

```bash
go test -bench=. -benchmem
```

---

## Пример вывода

```text
BenchmarkString-8    10000000    120 ns/op    16 B/op    1 allocs/op
```

Расшифровка:

```text
120 ns/op
```

- время одной операции
    

```text
16 B/op
```

- сколько байт выделяется на одну операцию
    

```text
1 allocs/op
```

- количество аллокаций на одну операцию
    

---

## Отличие от `-benchmem`

Вариант через флаг:

```bash
go test -bench=. -benchmem
```

включает аллокации для всех benchmark.

Вариант через код:

```go
b.ReportAllocs()
```

включает их только для конкретного benchmark.

---

## Использование с `ResetTimer`

```go
func BenchmarkProcess(b *testing.B) {

	data := prepare()

	b.ResetTimer()
	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Измеряются только:

- время `Process`;
    
- аллокации `Process`.
    

---

## Когда использовать

Полезно при оптимизации:

- уменьшения количества `alloc`;
    
- поиска лишних копирований;
    
- работы со строками;
    
- работы со структурами;
    
- оптимизации GC-нагрузки.
    

---

## Для собеседования

> `b.ReportAllocs()` включает отчёт по памяти в benchmark. Он показывает `B/op` и `allocs/op`, позволяя анализировать количество выделений памяти и искать лишние аллокации. Аналогично флагу `go test -benchmem`, но применяется для конкретного benchmark.