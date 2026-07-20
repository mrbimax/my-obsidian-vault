## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. StartTimer

## Назначение

`b.StartTimer()` — запускает измерение времени benchmark после того, как оно было остановлено через `b.StopTimer()`.

Используется вместе с:

- `b.StopTimer()`
    
- `b.ResetTimer()`
    

---

## Синтаксис

```go
b.StartTimer()
```

---

## Пример

```go
func BenchmarkProcess(b *testing.B) {

	data := prepare()

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		reset(data)

		b.StartTimer()

		Process(data)
	}
}
```

Измеряется:

```text
Process(data)
```

Не измеряется:

```text
reset(data)
```

---

## Порядок работы

```text
Benchmark start

        ↓

StartTimer()
        |
        ↓
измерение включено

        ↓

StopTimer()
        |
        ↓
измерение выключено

        ↓

StartTimer()
        |
        ↓
измерение снова включено
```

---

## Отличие от ResetTimer

`StartTimer`:

- продолжает текущий замер;
    
- используется после `StopTimer`.
    

`ResetTimer`:

- сбрасывает результаты;
    
- начинает новый замер с нуля.
    

---

## Пример с подготовкой

```go
func BenchmarkEncode(b *testing.B) {

	data := createData()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {

		b.StopTimer()

		tmp := clone(data)

		b.StartTimer()

		Encode(tmp)
	}
}
```

---

## Важные моменты

- `StartTimer()` обычно не вызывают в начале benchmark — таймер уже запущен автоматически.
    
- Основной сценарий использования:
    

```go
b.StopTimer()

// действия вне измерения

b.StartTimer()

// измеряемый код
```

- Если нужно просто исключить первоначальный setup — чаще используется `b.ResetTimer()`.
    

---

## Для собеседования

> `b.StartTimer()` возобновляет измерение benchmark после `b.StopTimer()`. В начале benchmark таймер уже запущен автоматически, поэтому чаще `StartTimer()` используется только после временной остановки измерения для исключения подготовительных операций.