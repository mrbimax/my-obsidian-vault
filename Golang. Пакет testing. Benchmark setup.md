## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark setup

## Назначение

Benchmark setup — подготовка данных и окружения перед измерением производительности.

Главная идея:

**Не включать подготовку в замер**, если она не является частью проверяемой операции.

---

## Простой setup

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareData()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Здесь:

```text
prepareData()
```

выполняется до benchmark-цикла и не учитывается.

Измеряется:

```text
Process(data)
```

---

## Плохой вариант

```go
func BenchmarkProcess(b *testing.B) {

	for i := 0; i < b.N; i++ {

		data := prepareData()

		Process(data)
	}
}
```

Измеряется:

```text
prepareData()
+
Process()
```

Результат будет показывать не только производительность `Process`.

---

## Setup с ResetTimer

Если подготовка происходит после старта benchmark:

```go
func BenchmarkProcess(b *testing.B) {

	data := make([]int, 1000000)

	fillData(data)

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

`ResetTimer()` исключает:

- время подготовки;
    
- аллокации подготовки.
    

---

## Setup в sub-benchmark

```go
func BenchmarkParser(b *testing.B) {

	inputs := []string{
		"small",
		"large",
	}

	for _, input := range inputs {

		input := input

		b.Run(input, func(b *testing.B) {

			data := prepare(input)

			b.ResetTimer()

			for i := 0; i < b.N; i++ {
				Parse(data)
			}

		})
	}
}
```

Каждый sub-benchmark имеет свой setup.

---

## Setup и StopTimer

Когда подготовка нужна внутри цикла:

```go
func BenchmarkProcess(b *testing.B) {

	data := prepare()

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		copy := clone(data)

		b.StartTimer()

		Process(copy)
	}
}
```

Измеряется:

```text
Process(copy)
```

Не измеряется:

```text
clone(data)
```

---

## Cleanup после benchmark

Для освобождения ресурсов:

```go
func BenchmarkDB(b *testing.B) {

	db := createDB()

	b.Cleanup(func() {
		db.Close()
	})

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		Query(db)
	}
}
```

---

## Типичные ресурсы setup

Перед benchmark часто создают:

- большие структуры данных;
    
- соединение с БД;
    
- HTTP client;
    
- cache;
    
- файлы;
    
- тестовые данные.
    

---

## Правильная структура benchmark

```go
func BenchmarkExample(b *testing.B) {

	// setup
	resource := prepare()

	b.ResetTimer()

	// измерение
	for i := 0; i < b.N; i++ {
		Work(resource)
	}

	// cleanup
	b.Cleanup(func() {
		Close(resource)
	})
}
```

---

## Для собеседования

> Benchmark setup — это подготовка окружения перед измерением производительности. Обычно setup выполняется до цикла `b.N`, чтобы не влиять на результат. Если подготовка происходит после запуска benchmark, используется `b.ResetTimer()`. Для операций внутри цикла применяются `b.StopTimer()` и `b.StartTimer()`.