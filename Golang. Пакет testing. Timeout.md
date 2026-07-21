## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Timeout

## Назначение

Флаг `-timeout` задаёт **максимальное время выполнения всего тестового запуска**.

Если все тесты, benchmarks или fuzzing работают дольше указанного времени — `go test` принудительно завершает процесс.

---

## Синтаксис

```bash
go test -timeout 30s
```

Примеры:

```bash
go test -timeout 1m
```

```bash
go test ./... -timeout 5m
```

Формат:

```text
число + единица времени

ns
us
ms
s
m
h
```

---

## Пример

Есть зависший тест:

```go
func TestDeadlock(t *testing.T) {

	select {}

}
```

Запуск:

```bash
go test -timeout 3s
```

Через 3 секунды:

```text
panic: test timed out after 3s
```

Go завершит процесс.

---

## Что происходит при timeout

Go:

1. останавливает выполнение тестов;
    
2. делает stack dump goroutines;
    
3. показывает, где завис код.
    

Пример:

```text
panic: test timed out after 3s

goroutine 18:
example.TestDeadlock()
    test.go:10
```

Это помогает найти:

- deadlock;
    
- зависший network call;
    
- бесконечный цикл;
    
- ожидание mutex/channel.
    

---

## Timeout применяется ко всему запуску

Важно:

```bash
go test -timeout 10s
```

это не:

```text
каждый тест по 10 секунд
```

а:

```text
весь процесс go test максимум 10 секунд
```

Например:

```go
func TestA(t *testing.T) {
	time.Sleep(6 * time.Second)
}

func TestB(t *testing.T) {
	time.Sleep(6 * time.Second)
}
```

При:

```bash
go test -timeout 10s
```

общий лимит:

```text
6s + 6s > 10s
```

тесты будут остановлены.

---

## Связь с `t.Deadline()`

Внутри теста можно узнать оставшееся время:

```go
func TestAPI(t *testing.T) {

	deadline, ok := t.Deadline()

	if ok {
		fmt.Println(deadline)
	}

}
```

`t.Deadline()` возвращает время, когда сработает общий timeout.

---

## Timeout и context

Часто используют вместе:

```go
func TestRequest(t *testing.T) {

	ctx, cancel := context.WithTimeout(
		context.Background(),
		time.Second,
	)

	defer cancel()

	err := DoRequest(ctx)

	if err != nil {
		t.Fatal(err)
	}

}
```

Разница:

`-timeout`

```text
защита всего тестового процесса
```

`context.WithTimeout`

```text
контроль конкретной операции
```

---

## Отключение timeout

Можно:

```bash
go test -timeout 0
```

означает:

```text
без ограничения времени
```

Обычно не рекомендуется для CI.

---

## Связанные команды

Все тесты:

```bash
go test ./... -timeout 2m
```

Benchmark:

```bash
go test -bench . -timeout 5m
```

Fuzz:

```bash
go test -fuzz FuzzParser -timeout 10m
```

---

## Для собеседования

> `-timeout` задаёт максимальное время выполнения команды `go test`. Если тесты зависли или выполняются слишком долго, Go завершает процесс и выводит stack trace goroutines. Это защита от deadlock, бесконечных циклов и зависших операций в тестах и CI.