## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark b.N

## Назначение

`b.N` — количество итераций, которое benchmark должен выполнить.

Go автоматически подбирает значение `b.N`, чтобы измерение было достаточно точным.

---

## Базовый пример

```go
func BenchmarkAdd(b *testing.B) {

	for i := 0; i < b.N; i++ {
		_ = 1 + 2
	}
}
```

`testing` сам меняет `b.N`:

```text
10
100
10000
1000000
...
```

пока не получит стабильное измерение.

---

## Почему нельзя задавать количество вручную

Плохо:

```go
func BenchmarkAdd(b *testing.B) {

	for i := 0; i < 1000; i++ {
		_ = 1 + 2
	}
}
```

Проблемы:

- слишком мало операций → неточная статистика;
    
- слишком много операций → долго выполняется;
    
- зависит от скорости машины.
    

---

## `b.N` и подготовка данных

Подготовку обычно выполняют **до цикла**:

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareData()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Иначе подготовка попадёт в измерение:

```go
func BenchmarkProcess(b *testing.B) {

	for i := 0; i < b.N; i++ {

		data := prepareData()

		Process(data)
	}
}
```

---

## `b.ResetTimer`

Если подготовка нужна внутри benchmark:

```go
func BenchmarkProcess(b *testing.B) {

	data := prepareData()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

`ResetTimer()`:

- сбрасывает время;
    
- сбрасывает счётчики аллокаций;
    
- начинает измерять только код после вызова.
    

---

## `b.N` нельзя использовать после цикла

Неправильно:

```go
func BenchmarkExample(b *testing.B) {

	for i := 0; i < b.N; i++ {
		work()
	}

	fmt.Println(b.N)
}
```

Benchmark-код должен находиться внутри измеряемого участка.

---

## Пример с результатом

Код:

```go
func BenchmarkConcat(b *testing.B) {

	for i := 0; i < b.N; i++ {
		_ = "hello" + "world"
	}
}
```

Запуск:

```bash
go test -bench=.
```

Вывод:

```text
BenchmarkConcat-8    1000000000    0.3 ns/op
```

Где:

- `1000000000` — выбранное значение `b.N`;
    
- `0.3 ns/op` — среднее время одной итерации.
    

---

## Для собеседования

> `b.N` — это автоматически подобранное количество итераций benchmark. Go изменяет его во время запуска, чтобы получить стабильное измерение. Код, который измеряется, обычно помещают внутрь цикла `for i := 0; i < b.N; i++`.