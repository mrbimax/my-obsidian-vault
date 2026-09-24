## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Concurrent testing

## Назначение

Concurrent testing — тестирование кода, который работает с конкурентностью:

- goroutines;
    
- channels;
    
- mutex;
    
- atomic operations;
    
- worker pools;
    
- concurrent access к данным.
    

Цель:

- проверить отсутствие race condition;
    
- проверить корректность работы при параллельном выполнении;
    
- найти проблемы синхронизации.
    

---

# Проверка goroutines

Пример кода:

```go
func Counter() int {

	counter := 0

	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {

		wg.Add(1)

		go func() {

			counter++

			wg.Done()

		}()

	}

	wg.Wait()

	return counter
}
```

Проблема:

```text
несколько goroutine одновременно изменяют counter
```

---

Тест:

```go
func TestCounter(t *testing.T) {

	result := Counter()

	if result != 100 {
		t.Fatalf(
			"got %d, want 100",
			result,
		)
	}

}
```

Может проходить, а может падать.

---

# Race detector

Для поиска data race:

```bash
go test -race ./...
```

Пример вывода:

```text
WARNING: DATA RACE

Write at:
counter++

Previous write:
counter++
```

---

# t.Parallel()

Позволяет запускать независимые тесты параллельно.

Пример:

```go
func TestUser(t *testing.T) {

	t.Parallel()

	// тест

}
```

Несколько тестов:

```go
func TestA(t *testing.T) {

	t.Parallel()

}

func TestB(t *testing.T) {

	t.Parallel()

}
```

Могут выполняться одновременно.

---

# Parallel subtests

Частый паттерн:

```go
func TestAPI(t *testing.T) {

	tests := []struct {
		name string
	}{
		{"create"},
		{"delete"},
		{"update"},
	}

	for _, tt := range tests {

		tt := tt

		t.Run(tt.name, func(t *testing.T) {

			t.Parallel()

			// тест

		})

	}

}
```

---

# Проверка channel

Пример:

```go
func TestWorker(t *testing.T) {

	ch := make(chan int)

	go func() {

		ch <- 42

	}()

	result := <-ch

	if result != 42 {
		t.Fatal("wrong value")
	}

}
```

---

# Проверка timeout

Для зависших goroutines:

```go
func TestWorker(t *testing.T) {

	done := make(chan bool)

	go func() {

		// работа

		done <- true

	}()

	select {

	case <-done:

	case <-time.After(time.Second):

		t.Fatal("timeout")

	}

}
```

---

# Cleanup goroutines

Проблема:

```go
go worker()
```

После теста:

```text
goroutine продолжает работать
```

Лучше:

```go
func TestWorker(t *testing.T) {

	ctx, cancel := context.WithCancel(
		context.Background(),
	)

	t.Cleanup(func() {

		cancel()

	})

	go worker(ctx)

}
```

---

# Проверка порядка выполнения

Для конкурентного кода часто используют:

- channels;
    
- WaitGroup;
    
- mutex;
    
- atomic.
    

Например:

```go
func TestConcurrent(t *testing.T) {

	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {

		wg.Add(1)

		go func() {

			defer wg.Done()

			// concurrent operation

		}()

	}

	wg.Wait()

}
```

---

# Concurrent testing vs t.Parallel()

Важно различать.

## t.Parallel()

Параллельно запускает **тесты**:

```text
TestA
TestB
TestC
```

---

## Concurrent testing

Проверяет **код внутри теста**:

```text
Test
 |
 ├── goroutine 1
 ├── goroutine 2
 └── goroutine 3
```

---

# Типичные проблемы

## Data race

Несинхронизированный доступ:

```go
counter++
```

из нескольких goroutines.

---

## Deadlock

Например:

```go
ch := make(chan int)

<-ch
```

Нет отправителя.

---

## Goroutine leak

Тест завершился, но:

```go
go worker()
```

продолжает работать.

---

# Для собеседования

> Concurrent testing в Go используется для проверки кода, работающего с goroutines, channels и синхронизацией. Обычно используют `go test -race` для поиска data race, `t.Parallel()` для запуска независимых тестов параллельно, а внутри тестов применяют `WaitGroup`, channels, mutex и timeout для проверки корректного поведения конкурентного кода. Важно отличать параллельный запуск тестов от тестирования конкурентности самого приложения.