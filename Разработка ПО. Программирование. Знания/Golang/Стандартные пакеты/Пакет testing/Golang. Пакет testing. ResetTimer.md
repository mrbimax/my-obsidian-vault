## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. ResetTimer

## Назначение

`b.ResetTimer()` — сбрасывает таймер benchmark.

Используется, когда перед измеряемым кодом есть подготовка, которую не нужно учитывать.

---

## Синтаксис

```go
b.ResetTimer()
```

---

## Проблема без `ResetTimer`

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareBigData()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Время включает:

```text
prepareBigData()
+
Process()
```

Но часто нужно измерять только:

```text
Process()
```

---

## Использование

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareBigData()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Теперь измеряется только:

```text
Process()
```

---

## Что делает `ResetTimer`

Сбрасывает:

- время выполнения benchmark;
    
- счётчик аллокаций;
    
- счётчик байтов памяти.
    

После вызова начинается новый этап измерения.

---

## Пример с несколькими этапами

```go
func BenchmarkExample(b *testing.B) {

	setup()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		work()
	}
}
```

Измеряется только:

```text
work()
```

---

## Отличие от `StopTimer`

`ResetTimer`:

- полностью сбрасывает измерения;
    
- начинает новый отсчёт.
    

`StopTimer`:

- временно останавливает измерение.
    

Пример:

```go
func BenchmarkExample(b *testing.B) {

	data := prepare()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		changeData(data)

		b.StartTimer()

		process(data)
	}
}
```

---

## Для собеседования

> `b.ResetTimer()` используется в benchmark, чтобы исключить время подготовки данных из измерения. Он сбрасывает таймер и счётчики аллокаций, после чего benchmark начинает считать только код после вызова `ResetTimer()`.