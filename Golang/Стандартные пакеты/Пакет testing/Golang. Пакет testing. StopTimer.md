## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. StopTimer

## Назначение

`b.StopTimer()` — временно останавливает измерение времени в benchmark.

Используется, когда внутри цикла `b.N` есть операции, которые **не должны попадать в замер**.

---

## Синтаксис

```go
b.StopTimer()
```

Возобновление измерения:

```go
b.StartTimer()
```

---

## Проблема без `StopTimer`

```go
func BenchmarkProcess(b *testing.B) {

	data := make([]int, 1000)

	for i := 0; i < b.N; i++ {

		update(data)

		reset(data)
	}
}
```

Измеряется:

```text
update()
+
reset()
```

Но нужно измерить только:

```text
update()
```

---

## Использование

```go
func BenchmarkProcess(b *testing.B) {

	data := make([]int, 1000)

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		reset(data)

		b.StartTimer()

		update(data)
	}
}
```

Теперь:

- `reset()` не учитывается;
    
- `update()` измеряется.
    

---

## Отличие от `ResetTimer`

`StopTimer`:

- временно ставит измерение на паузу;
    
- сохраняет текущие результаты;
    
- используется внутри benchmark.
    

`ResetTimer`:

- полностью сбрасывает измерения;
    
- начинает новый замер с нуля.
    

---

## Пример с генерацией данных

```go
func BenchmarkEncode(b *testing.B) {

	data := createData()

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		input := clone(data)

		b.StartTimer()

		Encode(input)
	}
}
```

Измеряется только:

```text
Encode(input)
```

---

## Важные моменты

- `StopTimer()` нужно использовать вместе с `StartTimer()`;
    
- не стоит часто включать/выключать таймер без необходимости;
    
- чаще всего подготовку выносят перед циклом и используют `ResetTimer()`.
    

---

## Для собеседования

> `b.StopTimer()` временно останавливает измерение benchmark. Используется, когда внутри цикла `b.N` есть подготовительные операции, которые не должны влиять на результат. После нужной подготовки вызывается `b.StartTimer()`. Для обычной подготовки перед benchmark чаще используют `b.ResetTimer()`.