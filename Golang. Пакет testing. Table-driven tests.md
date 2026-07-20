## *[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]* 
---
## Golang. Пакет testing. Table-driven tests

**Table-driven tests** — основной паттерн написания тестов в Go.

Идея:
- набор тестовых случаев хранится в таблице (`slice` структур);
- один тестовый код прогоняется для всех случаев;
- легко добавлять новые проверки.
---
## Базовая структура

```go
tests := []struct {
	name string
	input int
	want int
}{
	{
		name:  "case 1",
		input: 10,
		want:  20,
	},
}
```

Поля обычно:
- `name` — название теста
- `input` — входные данные
- `want` — ожидаемый результат
---
## Пример

Функция:
```go
func Double(x int) int {
	return x * 2
}
```

Тест:
```go
func TestDouble(t *testing.T) {

	tests := []struct {
		name  string
		input int
		want  int
	}{
		{
			name:  "positive",
			input: 5,
			want:  10,
		},
		{
			name:  "zero",
			input: 0,
			want:  0,
		},
		{
			name:  "negative",
			input: -5,
			want:  -10,
		},
	}

	for _, tt := range tests {

		t.Run(tt.name, func(t *testing.T) {

			got := Double(tt.input)

			if got != tt.want {
				t.Fatalf(
					"got %v want %v",
					got,
					tt.want,
				)
			}
		})
	}
}
```
---
## Почему используют `t.Run`

Без `t.Run`:
```go
for _, tt := range tests {
	// проверка
}
```

Минусы:
- непонятно какой кейс упал;
- нельзя запускать отдельный случай.

С `t.Run`:
```text
TestDouble
 ├── positive
 ├── zero
 └── negative
```

Можно запустить:
```bash
go test -run TestDouble/negative
```
---
## Проверка ошибок

Часто добавляют поле `wantErr`:
```go
tests := []struct {
	name    string
	input   string
	wantErr bool
}{
	{
		name:    "invalid input",
		input:   "abc",
		wantErr: true,
	},
}
```

Пример:
```go
got, err := Parse(tt.input)

if tt.wantErr && err == nil {
	t.Fatal("expected error")
}

if !tt.wantErr && err != nil {
	t.Fatal(err)
}
```
---
## Более сложный пример
```go
tests := []struct {
	name string
	a    int
	b    int
	want int
}{
	{
		name: "sum positive",
		a:    1,
		b:    2,
		want: 3,
	},
	{
		name: "sum negative",
		a:    -1,
		b:    -2,
		want: -3,
	},
}
```
---
## Best practices

- Давать каждому кейсу понятное имя
- Проверять edge cases:
    - пустые значения 
    - nil
    - ошибки
    - большие значения
- Использовать `t.Run`
- Не дублировать код проверки
- Хранить ожидаемый результат рядом с входными данными
---
## Где часто применяется

- unit-тесты функций
- тестирование парсеров
- проверка валидаторов
- HTTP handlers
- бизнес-логика
- сериализация / десериализация
---
## Для собеседования

> Table-driven tests — идиоматичный стиль тестов в Go, где тестовые сценарии описываются как таблица структур, а один тестовый цикл выполняет проверку каждого кейса. Обычно используется вместе с `t.Run` для именованных подтестов и удобного запуска отдельных сценариев.