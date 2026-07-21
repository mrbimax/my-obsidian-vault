## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark subtests

## Назначение

Benchmark subtests (`b.Run`) позволяют создавать **вложенные benchmark-тесты**.

Используются для:

- сравнения нескольких вариантов реализации;
    
- тестирования разных размеров входных данных;
    
- группировки benchmark.
    

---

## Синтаксис

```go
b.Run(name string, func(b *testing.B) {
	// benchmark
})
```

---

## Простой пример

```go
package main

import "testing"

func BenchmarkAdd(b *testing.B) {

	b.Run("small", func(b *testing.B) {

		for i := 0; i < b.N; i++ {
			_ = 1 + 2
		}

	})

	b.Run("large", func(b *testing.B) {

		for i := 0; i < b.N; i++ {
			_ = 1000000 + 2000000
		}

	})
}
```

Запуск:

```bash
go test -bench=.
```

Вывод:

```text
BenchmarkAdd/small-8     1000000000    0.3 ns/op
BenchmarkAdd/large-8     1000000000    0.3 ns/op
```

---

## Benchmark с разными параметрами

Частый паттерн:

```go
package main

import "testing"

func BenchmarkProcess(b *testing.B) {

	sizes := []int{
		10,
		1000,
		100000,
	}

	for _, size := range sizes {

		b.Run(
			fmt.Sprintf("size_%d", size),
			func(b *testing.B) {

				data := make([]int, size)

				b.ResetTimer()

				for i := 0; i < b.N; i++ {
					Process(data)
				}

			},
		)
	}
}
```

Результат:

```text
BenchmarkProcess/size_10-8
BenchmarkProcess/size_1000-8
BenchmarkProcess/size_100000-8
```

---

## Важный момент: захват переменной цикла

При использовании цикла лучше копировать переменную:

```go
for _, size := range sizes {

	size := size

	b.Run(
		fmt.Sprintf("size_%d", size),
		func(b *testing.B) {

			data := make([]int, size)

			for i := 0; i < b.N; i++ {
				Process(data)
			}

		},
	)
}
```

Иначе subtest может использовать неожиданное значение переменной.

---

## Parallel benchmark subtests

Можно запускать параллельно:

```go
func BenchmarkServices(b *testing.B) {

	services := []string{
		"redis",
		"postgres",
	}

	for _, service := range services {

		service := service

		b.Run(service, func(b *testing.B) {

			b.Parallel()

			for i := 0; i < b.N; i++ {
				Request(service)
			}

		})
	}
}
```

---

## Отличие от обычных тестов

`testing.T`:

```go
t.Run()
```

для unit-тестов.

`testing.B`:

```go
b.Run()
```

для benchmark.

Механика похожа, но внутри subtest управляется benchmark-таймером.

---

## Для собеседования

> Benchmark subtests создаются через `b.Run()` и позволяют запускать несколько измерений внутри одного benchmark. Обычно используются для сравнения реализаций или проверки производительности на разных размерах входных данных. Каждый subtest получает собственный `*testing.B` и отдельный цикл `b.N`.