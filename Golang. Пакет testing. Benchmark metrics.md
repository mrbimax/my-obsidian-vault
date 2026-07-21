## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Benchmark metrics

## Назначение

Benchmark metrics — это показатели, которые Go собирает при выполнении benchmark.

Основные метрики:

- время выполнения;
    
- количество операций;
    
- выделение памяти;
    
- количество аллокаций;
    
- пользовательские метрики.
    

---

## Основная метрика `ns/op`

По умолчанию benchmark показывает:

```text
ns/op
```

Количество наносекунд на одну операцию.

Пример:

```text
BenchmarkAdd-8    1000000000    0.32 ns/op
```

Расшифровка:

```text
0.32 ns/op
```

означает:

```text
один вызов операции занимает 0.32 наносекунды
```

---

## Количество итераций (`b.N`)

Пример вывода:

```text
BenchmarkProcess-8    1000000    500 ns/op
```

```text
1000000
```

это количество выполненных операций.

Соответствует:

```go
b.N
```

---

## Память (`-benchmem`)

Запуск:

```bash
go test -bench=. -benchmem
```

Пример:

```text
BenchmarkEncode-8
1000000
500 ns/op
256 B/op
4 allocs/op
```

---

## `B/op`

Количество выделенных байт на одну операцию.

Пример:

```text
256 B/op
```

означает:

```text
каждый вызов функции выделяет 256 байт памяти
```

---

## `allocs/op`

Количество аллокаций памяти на одну операцию.

Пример:

```text
4 allocs/op
```

означает:

```text
одна операция создаёт 4 объекта в куче
```

---

## Включение отчёта памяти

Через команду:

```bash
go test -bench=. -benchmem
```

или в коде:

```go
b.ReportAllocs()
```

Пример:

```go
func BenchmarkJSON(b *testing.B) {

	b.ReportAllocs()

	for i := 0; i < b.N; i++ {
		json.Marshal(data)
	}
}
```

---

## Пользовательские метрики

Через:

```go
b.ReportMetric(value, unit)
```

Пример:

```go
func BenchmarkParser(b *testing.B) {

	for i := 0; i < b.N; i++ {
		Parse(data)
	}

	b.ReportMetric(
		float64(len(data)),
		"bytes/op",
	)
}
```

Вывод:

```text
BenchmarkParser-8
1000000
500 ns/op
1024 bytes/op
```

---

## Примеры полезных метрик

Пропускная способность:

```text
MB/s
```

Количество обработанных элементов:

```text
items/op
```

Количество запросов:

```text
requests/sec
```

Количество сообщений:

```text
messages/op
```

---

## Метрики и таймер

Важно учитывать, что измеряется только код между запуском и остановкой таймера:

```go
func BenchmarkExample(b *testing.B) {

	data := prepare()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		Process(data)
	}
}
```

Измеряется:

```text
Process()
```

Не измеряется:

```text
prepare()
```

---

## Полезные флаги

Benchmark:

```bash
go test -bench=.
```

Память:

```bash
go test -bench=. -benchmem
```

Несколько запусков:

```bash
go test -bench=. -count=5
```

CPU варианты:

```bash
go test -bench=. -cpu=1,2,4
```

---

## Для собеседования

> Benchmark в Go показывает время выполнения (`ns/op`), количество итераций (`b.N`) и, при включении `-benchmem` или `ReportAllocs`, количество выделенной памяти (`B/op`) и аллокаций (`allocs/op`). Для специфичных случаев можно добавлять собственные метрики через `ReportMetric()`. Эти показатели используются для поиска узких мест и проверки эффективности оптимизаций.