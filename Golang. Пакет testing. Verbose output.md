## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Verbose output

## Назначение

Флаг `-v` (`verbose`) включает **подробный вывод** при запуске тестов.

Используется для:

- просмотра запуска каждого теста;
    
- отладки падений;
    
- просмотра логов из `t.Log`;
    
- анализа выполнения subtests.
    

---

## Синтаксис

```bash
go test -v
```

Для конкретного пакета:

```bash
go test ./parser -v
```

---

## Обычный запуск

Тест:

```go
func TestAdd(t *testing.T) {

	t.Log("calculating")

}
```

Команда:

```bash
go test
```

Вывод:

```text
PASS
ok      example/parser    0.002s
```

Логи не показываются.

---

## С `-v`

```bash
go test -v
```

Вывод:

```text
=== RUN   TestAdd
    add_test.go:5: calculating
--- PASS: TestAdd (0.00s)
PASS
ok      example/parser    0.002s
```

---

# Что показывает `-v`

## Запуск тестов

```text
=== RUN   TestAdd
```

---

## Успешное завершение

```text
--- PASS: TestAdd (0.00s)
```

---

## Ошибка

```text
--- FAIL: TestAdd (0.00s)
```

---

## Skip

```text
--- SKIP: TestIntegration (0.00s)
```

---

## Subtests

Код:

```go
func TestMath(t *testing.T) {

	t.Run("Add", func(t *testing.T) {

	})

	t.Run("Sub", func(t *testing.T) {

	})

}
```

Вывод:

```text
=== RUN   TestMath
=== RUN   TestMath/Add
--- PASS: TestMath/Add (0.00s)
=== RUN   TestMath/Sub
--- PASS: TestMath/Sub (0.00s)
--- PASS: TestMath (0.00s)
```

---

# `-v` и `t.Log`

Без:

```bash
go test
```

```go
t.Log("debug")
```

не выводится, если тест успешный.

---

С:

```bash
go test -v
```

выводится:

```text
add_test.go:10: debug
```

---

# `-v` и Benchmark

Benchmark:

```go
func BenchmarkAdd(b *testing.B) {

	for i := 0; i < b.N; i++ {
		Add(1, 2)
	}

}
```

Запуск:

```bash
go test -bench . -v
```

Покажет дополнительную информацию о запуске.

---

# `-v` и Fuzz

Запуск:

```bash
go test -fuzz FuzzParser -v
```

Показывает:

- запуск fuzz-функции;
    
- найденные ошибки;
    
- дополнительные сообщения.
    

---

# Когда используют

Чаще всего:

## Локальная разработка

```bash
go test -v ./...
```

Чтобы видеть:

- какие тесты идут;
    
- где зависло;
    
- какие логи выводятся.
    

---

## CI

Обычно:

```bash
go test ./...
```

для компактного вывода.

При падении:

```bash
go test -v ./...
```

для диагностики.

---

## Для собеседования

> Флаг `-v` включает подробный вывод тестового раннера. Он показывает запуск и результат каждого теста, subtest'ов, а также вывод `t.Log`. Используется при отладке и анализе поведения тестов, когда стандартного вывода недостаточно.