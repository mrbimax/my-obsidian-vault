## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Table-driven pattern

## Назначение

Table-driven pattern — способ написания тестов, при котором тестовые данные хранятся в таблице (обычно слайсе структур), а один и тот же код выполняется для каждого набора данных.

Это **стандартный стиль написания unit-тестов в Go**.

---

## Зачем нужен

Без table-driven тестов:

```go
func TestAdd_Positive(t *testing.T) {}

func TestAdd_Negative(t *testing.T) {}

func TestAdd_Zero(t *testing.T) {}
```

Много дублирования.

С table-driven:

```go
func TestAdd(t *testing.T) {}
```

Все сценарии находятся в одном месте.

---

## Базовый шаблон

```go
func TestAdd(t *testing.T) {

	tests := []struct {
		a    int
		b    int
		want int
	}{
		{1, 2, 3},
		{5, 5, 10},
		{-1, 1, 0},
	}

	for _, tt := range tests {

		got := Add(tt.a, tt.b)

		if got != tt.want {
			t.Fatalf(
				"got %d, want %d",
				got,
				tt.want,
			)
		}
	}

}
```

---

## Именованные тесты

Часто добавляют поле `name`:

```go
func TestAdd(t *testing.T) {

	tests := []struct {
		name string
		a    int
		b    int
		want int
	}{
		{"positive", 1, 2, 3},
		{"zero", 0, 0, 0},
		{"negative", -1, 1, 0},
	}

	for _, tt := range tests {

		t.Run(tt.name, func(t *testing.T) {

			got := Add(tt.a, tt.b)

			if got != tt.want {
				t.Fatalf(
					"got %d, want %d",
					got,
					tt.want,
				)
			}

		})
	}

}
```

Так каждый набор данных становится отдельным **subtest**.

---

## Преимущества

- нет дублирования кода;
    
- легко добавлять новые сценарии;
    
- удобно читать;
    
- хорошо сочетается с `t.Run()` и `t.Parallel()`.
    

---

## Что обычно хранится в таблице

- входные данные;
    
- ожидаемый результат;
    
- ожидаемая ошибка;
    
- название тестового случая.
    

Например:

```go
tests := []struct {
	name    string
	input   string
	want    int
	wantErr bool
}{
	{"valid", "123", 123, false},
	{"invalid", "abc", 0, true},
}
```

---

## Table-driven + Subtests

Это наиболее распространённый шаблон:

```go
for _, tt := range tests {

	t.Run(tt.name, func(t *testing.T) {

		// проверка

	})

}
```

Преимущества:

- каждый сценарий отображается отдельно;
    
- можно запускать конкретный сценарий через `-run`;
    
- проще искать упавший кейс.
    

---

## Table-driven + Parallel

Если сценарии независимы:

```go
for _, tt := range tests {

	tt := tt

	t.Run(tt.name, func(t *testing.T) {

		t.Parallel()

		// проверка

	})

}
```

> `tt := tt` нужен, чтобы каждая замыкание получило свою копию переменной цикла.

---

## Когда использовать

Table-driven тесты подходят, когда одна функция должна быть проверена на множестве входных данных:

- математические функции;
    
- парсеры;
    
- валидаторы;
    
- конвертеры;
    
- обработчики строк;
    
- бизнес-правила.
    

---

## Для собеседования

> Table-driven tests — это идиоматичный способ написания тестов в Go. Тестовые данные хранятся в таблице (`[]struct`), а один тест последовательно проверяет все сценарии. Обычно такие тесты сочетают с `t.Run()`, чтобы каждый набор данных выполнялся как отдельный subtest. Это уменьшает дублирование кода и упрощает добавление новых тестовых случаев.