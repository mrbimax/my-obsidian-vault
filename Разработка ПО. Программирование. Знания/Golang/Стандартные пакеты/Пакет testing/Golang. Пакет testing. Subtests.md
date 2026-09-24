## *[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]* 
---
## Golang. Пакет testing. Subtests

Subtests (`t.Run`) — вложенные тесты внутри основного теста.

Используются для:
- группировки сценариев
- удобного отображения ошибок
- запуска отдельных кейсов
- совместного использования setup/cleanup

---
## Синтаксис

```go
t.Run("name", func(t *testing.T) {
	// тест
})
```

---
## Пример

```go
func TestUser(t *testing.T) {

	t.Run("create", func(t *testing.T) {
		// проверка создания
	})

	t.Run("delete", func(t *testing.T) {
		// проверка удаления
	})
}
```

Структура:

```
TestUser
 ├── create
 └── delete
```
---
## Table-driven tests + Subtests

Основной стиль в Go:
```go
func TestAdd(t *testing.T) {

	tests := []struct {
		name string
		a    int
		b    int
		want int
	}{
		{
			name: "positive",
			a: 1,
			b: 2,
			want: 3,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {

			got := tt.a + tt.b

			if got != tt.want {
				t.Fatal("wrong result")
			}
		})
	}
}
```
---
## Запуск конкретного subtest

Все:
```bash
go test -run TestAdd
```

Только один:
```bash
go test -run TestAdd/positive
```
---
## Вложенные subtests

```go
func TestAPI(t *testing.T) {

	t.Run("users", func(t *testing.T) {

		t.Run("create", func(t *testing.T) {

		})

	})
}
```

Структура:
```
TestAPI
└── users
    └── create
```
---
## Cleanup внутри subtest

```go
t.Run("file", func(t *testing.T) {

	file := createFile()

	t.Cleanup(func() {
		removeFile(file)
	})
})
```
---
## Параллельные subtests

```go
t.Run("case", func(t *testing.T) {
	t.Parallel()
})
```

Позволяет запускать независимые тесты параллельно.

---
## Для собеседования

> Subtests в Go создаются через `t.Run`. Они позволяют организовать тесты в дерево сценариев, часто используются вместе с table-driven tests, поддерживают отдельный запуск через `-run`, cleanup и параллельное выполнение.