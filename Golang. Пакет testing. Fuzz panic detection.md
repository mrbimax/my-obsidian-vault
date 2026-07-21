## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz panic detection

## Назначение

Fuzz panic detection — способность fuzz-тестов автоматически находить входные данные, которые приводят к `panic`.

Основная идея:

> В fuzzing часто не проверяют конкретный результат, а проверяют, что программа не падает на любых входных данных.

---

## Пример

Есть функция:

```go
func Parse(data []byte) {

	if len(data) == 0 {
		panic("empty data")
	}

}
```

Fuzz-тест:

```go
func FuzzParse(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Запуск:

```bash
go test -fuzz=FuzzParse
```

Fuzzer начинает передавать разные значения:

```text
[]byte("hello")
[]byte("123")
[]byte("")
[]byte{0xff, 0xff}
```

Когда найдёт:

```go
Parse([]byte{})
```

произойдёт:

```text
panic: empty data
```

---

## Что делает Go после panic

Go:

1. останавливает текущий fuzz case;
    
2. фиксирует входные данные;
    
3. сохраняет их в corpus;
    
4. выводит минимальный воспроизводимый пример.
    

Пример вывода:

```text
--- FAIL: FuzzParse

Failing input written to:
testdata/fuzz/FuzzParse/abc123

panic: empty data
```

---

## Найденный input становится обычным тестом

После сохранения:

```text
testdata/
└── fuzz/
    └── FuzzParse/
        └── abc123
```

При следующем:

```bash
go test
```

Go снова запустит этот случай.

То есть найденный баг больше не потеряется.

---

## Почему это полезнее обычного теста

Обычный тест:

```go
func TestParse(t *testing.T) {

	Parse([]byte{})

}
```

Ты должен заранее знать:

```text
"пустой массив ломает функцию"
```

---

Fuzz:

```go
func FuzzParse(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Сам ищет:

```text
пустой input
слишком длинный input
битые байты
неожиданные комбинации
```

---

## Panic считается ошибкой fuzz-теста

Не нужно писать:

```go
defer func() {

	if recover() != nil {
		t.Fail()
	}

}()
```

Go автоматически считает `panic`:

```text
FAIL
```

---

## Если panic ожидаемый

Например:

```go
func Divide(a, b int) int {

	if b == 0 {
		panic("zero division")
	}

	return a / b
}
```

Такой случай не подходит для fuzz без дополнительной логики.

Нужно явно обработать:

```go
func FuzzDivide(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		a int,
		b int,
	) {

		if b == 0 {
			t.Skip()
		}

		Divide(a, b)

	})

}
```

---

## Что обычно ищут через panic detection

Особенно эффективно для:

- парсеров;
    
- сериализации/десериализации;
    
- работы с байтами;
    
- обработки пользовательского ввода;
    
- сетевых протоколов;
    
- криптографических функций.
    

---

## Связь с coverage guided fuzzing

Механизм:

```text
seed
 |
 ↓
mutation
 |
 ↓
новый input
 |
 ↓
запуск функции
 |
 +----------------+
 |                |
panic?        нет panic
 |                |
 ↓                ↓
сохранить      анализ coverage
bug input
```

---

## Для собеседования

> Fuzz panic detection — это автоматический поиск входных данных, которые приводят программу к аварийному завершению. Go fuzzing запускает функцию с множеством сгенерированных входов и автоматически считает `panic` ошибкой, сохраняя минимальный воспроизводимый input в corpus. Это особенно полезно для поиска ошибок в парсерах, обработчиках внешних данных и низкоуровневом коде.