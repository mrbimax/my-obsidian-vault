## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Error testing

## Назначение

Error testing — проверка функций, которые возвращают `error`.

Используется для проверки:

- успешного выполнения;
    
- возникновения ошибки;
    
- конкретного типа ошибки;
    
- конкретного значения ошибки.
    

---

## Проверка отсутствия ошибки

```go
func TestParse(t *testing.T) {

	_, err := Parse("123")

	if err != nil {
		t.Fatalf(
			"unexpected error: %v",
			err,
		)
	}

}
```

---

## Проверка наличия ошибки

```go
func TestParse(t *testing.T) {

	_, err := Parse("abc")

	if err == nil {
		t.Fatal("expected error")
	}

}
```

---

## Проверка конкретной ошибки

Есть ошибка:

```go
var ErrInvalidInput = errors.New("invalid input")
```

Функция:

```go
return ErrInvalidInput
```

Тест:

```go
if !errors.Is(err, ErrInvalidInput) {
	t.Fatalf(
		"expected ErrInvalidInput, got %v",
		err,
	)
}
```

Используется:

```go
errors.Is()
```

---

## Проверка типа ошибки

Есть тип:

```go
type ValidationError struct {
	Message string
}
```

Тест:

```go
var validationErr *ValidationError

if !errors.As(err, &validationErr) {
	t.Fatal("expected ValidationError")
}
```

Используется:

```go
errors.As()
```

---

## Проверка текста ошибки

Иногда проверяют сообщение:

```go
if err.Error() != "invalid input" {
	t.Fatal("wrong error")
}
```

Но это менее надёжно, чем `errors.Is()`.

---

## Table-driven пример

```go
func TestParse(t *testing.T) {

	tests := []struct {
		name    string
		input   string
		wantErr bool
	}{
		{"valid", "123", false},
		{"invalid", "abc", true},
	}

	for _, tt := range tests {

		t.Run(tt.name, func(t *testing.T) {

			_, err := Parse(tt.input)

			if tt.wantErr && err == nil {
				t.Fatal("expected error")
			}

			if !tt.wantErr && err != nil {
				t.Fatalf(
					"unexpected error: %v",
					err,
				)
			}

		})
	}

}
```

---

## Что использовать

Нет ошибки:

```go
err == nil
```

---

Есть ошибка:

```go
err != nil
```

---

Конкретная ошибка:

```go
errors.Is(err, ErrNotFound)
```

---

Конкретный тип:

```go
errors.As(err, &target)
```

---

## Для собеседования

> При тестировании ошибок сначала проверяют, должна ли ошибка возникнуть (`err == nil` или `err != nil`). Если важно проверить конкретную ошибку, используют `errors.Is()`, а если важен её тип — `errors.As()`. Сравнение строк через `err.Error()` возможно, но считается менее надёжным, так как текст сообщения может измениться без изменения логики.