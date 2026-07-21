## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz error detection

## Назначение

Fuzz error detection — поиск ошибок, которые проявляются при определённых входных данных.

В отличие от `panic detection`, где программа падает, здесь fuzz-тест сам проверяет **условия корректности**.

---

## Panic detection vs Error detection

Panic:

```text
input
 ↓
panic
 ↓
fuzzer обнаружил проблему автоматически
```

Error detection:

```text
input
 ↓
неверный результат / нарушение условия
 ↓
тест сообщает ошибку
```

---

## Пример: проверка инварианта

Есть функция:

```go
func Reverse(s string) string {
	r := []rune(s)

	for i, j := 0, len(r)-1; i < j; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}

	return string(r)
}
```

Правило:

```text
Reverse(Reverse(x)) == x
```

Fuzz-тест:

```go
func FuzzReverse(f *testing.F) {

	f.Add("hello")

	f.Fuzz(func(
		t *testing.T,
		input string,
	) {

		result := Reverse(Reverse(input))

		if result != input {
			t.Errorf(
				"broken property: %q",
				input,
			)
		}

	})

}
```

---

## Как fuzz находит ошибку

Допустим есть баг:

```go
func Reverse(s string) string {

	if s == "abc" {
		return "wrong"
	}

	return s
}
```

Fuzzer генерирует:

```text
hello
test
abc
```

На:

```go
Reverse("abc")
```

получает:

```text
wrong
```

Проверка:

```text
Reverse(Reverse("abc")) != "abc"
```

Тест падает.

---

## Важно

Fuzzer сам не знает, что является ошибкой.

Он не понимает:

```text
"возраст должен быть положительным"
```

или:

```text
"JSON должен содержать id"
```

Это задаёт разработчик.

---

Пример:

```go
func FuzzValidateAge(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		age int,
	) {

		valid := ValidateAge(age)

		if age < 0 && valid {
			t.Errorf(
				"negative age accepted",
			)
		}

	})

}
```

Здесь правило:

```text
age < 0 → нельзя принимать
```

---

## Где используется

### Проверка round-trip

Например:

```text
Encode
  |
  ↓
Decode
  |
  ↓
исходные данные
```

Проверка:

```go
Decode(Encode(data)) == data
```

---

### Проверка сериализации

```go
func FuzzJSON(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		var obj User

		err := json.Unmarshal(data, &obj)

		if err == nil {

			result, _ := json.Marshal(obj)

			if len(result) == 0 {
				t.Error("empty result")
			}

		}

	})

}
```

---

### Проверка границ

Например:

```go
func FuzzCalculate(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		value int,
	) {

		result := Calculate(value)

		if result < 0 {
			t.Errorf(
				"negative result",
			)
		}

	})

}
```

Fuzzer найдёт:

```text
0
-1
MinInt
MaxInt
```

---

## Как сохранить найденную ошибку

Если:

```go
t.Errorf()
```

сработал:

Go сохраняет input:

```text
testdata/
└── fuzz/
    └── FuzzValidateAge/
        └── abc123
```

Теперь этот случай будет запускаться всегда.

---

## Отличие от unit-теста

Unit test:

```go
ValidateAge(-1)
```

Ты заранее знаешь:

```text
-1 → ошибка
```

---

Fuzz:

```go
ValidateAge(age)
```

Fuzzer сам найдёт:

```text
-1
-100
MinInt
```

и другие неожиданные значения.

---

## Для собеседования

> Fuzz error detection — это проверка корректности программы на автоматически сгенерированных входных данных. В отличие от panic detection, где ошибка определяется падением программы, здесь разработчик задаёт инварианты или ожидаемые свойства, а fuzzing ищет входы, которые эти свойства нарушают. Найденные случаи сохраняются в corpus и становятся регрессионными тестами.