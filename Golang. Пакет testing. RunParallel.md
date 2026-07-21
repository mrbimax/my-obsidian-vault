## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. RunParallel

## Назначение

`b.RunParallel()` запускает benchmark в **нескольких goroutine одновременно**.

Используется для измерения производительности:

- потокобезопасного кода;
    
- конкурентных структур данных;
    
- работы с mutex;
    
- атомиков;
    
- concurrent cache.
    

---

## Синтаксис

```go
b.RunParallel(func(pb *testing.PB) {
	for pb.Next() {
		// измеряемый код
	}
})
```

---

## Пример

```go
package main

import (
	"sync"
	"testing"
)

func BenchmarkMutex(b *testing.B) {

	var mu sync.Mutex
	counter := 0

	b.RunParallel(func(pb *testing.PB) {

		for pb.Next() {

			mu.Lock()
			counter++
			mu.Unlock()

		}

	})
}
```

---

## Как работает

Обычный benchmark:

```text
goroutine 1

for i := 0; i < b.N; i++ {
	work()
}
```

---

`RunParallel`:

```text
goroutine 1 ─┐
goroutine 2 ─┤
goroutine 3 ─┼── work()
goroutine 4 ─┘
```

Несколько goroutine выполняют один и тот же benchmark одновременно.

---

## `testing.PB`

`PB` (Parallel Benchmark) управляет итерациями.

Главный метод:

```go
pb.Next()
```

Возвращает:

```go
true
```

пока есть оставшиеся операции.

Пример:

```go
for pb.Next() {

	operation()

}
```

---

## Количество goroutine

По умолчанию зависит от:

```go
GOMAXPROCS
```

Можно изменить:

```bash
go test -bench=. -cpu=1,2,4,8
```

Пример:

```text
BenchmarkMutex-8
BenchmarkMutex-16
```

---

## Отличие от обычного benchmark

Обычный:

```go
for i := 0; i < b.N; i++ {
	work()
}
```

Проверяет:

```text
скорость одной goroutine
```

---

`RunParallel`:

```go
b.RunParallel(...)
```

Проверяет:

```text
масштабирование при конкуренции
```

---

## Пример сравнения atomic и mutex

```go
package main

import (
	"sync/atomic"
	"testing"
)

func BenchmarkAtomic(b *testing.B) {

	var counter int64

	b.RunParallel(func(pb *testing.PB) {

		for pb.Next() {
			atomic.AddInt64(&counter, 1)
		}

	})
}
```

Можно сравнить с:

```go
sync.Mutex
```

и увидеть разницу при нагрузке.

---

## Когда использовать

Подходит для:

- concurrent map;
    
- sync.Pool;
    
- atomic operations;
    
- mutex;
    
- worker pools;
    
- connection pools.
    

Не подходит для:

- обычных функций без конкурентности;
    
- алгоритмов, которые не рассчитаны на параллельный доступ.
    

---

## Для собеседования

> `b.RunParallel()` используется для benchmark конкурентного кода. Он запускает несколько goroutine, каждая из которых выполняет операции через `testing.PB.Next()`. В отличие от обычного benchmark, который измеряет одну последовательную нагрузку, `RunParallel` позволяет оценить поведение приложения при конкуренции и проверить масштабирование.