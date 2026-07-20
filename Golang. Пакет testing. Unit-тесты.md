## Назначение

Unit-тесты проверяют корректность работы отдельных функций, методов или небольших компонентов без внешних зависимостей. 

```go
import "testing"
```
---
## Структура теста

Файл должен иметь суффикс:
``` go
*_test.go
```

Функция теста:
``` go
func TestName(t *testing.T)
```

Пример:
``` go
func TestAdd(t *testing.T) {
	result := Add(2, 3)

	if result != 5 {
		t.Fatal("wrong result")
	}
}
```

Запуск:
``` go
go test
```

---
## Основные методы `testing.T`

``` go
t.Error()
```
- помечает тест как failed
- выполнение продолжается

``` go
t.Fatal()
```
- помечает тест как failed
- немедленно останавливает тест

``` go
t.Log()
```
- вывод логов при запуске с `-v`

``` go
t.Skip()
```
- пропуск теста

---
## Проверка ошибок

Обычно используют:
``` go
if got != want {
	t.Fatalf("got %v want %v", got, want)
}
```

Где:

- `got` — фактический результат
- `want` — ожидаемый результат

---
## Table-driven tests

Основной паттерн в Go:
``` go
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

			got := Add(tt.a, tt.b)

			if got != tt.want {
				t.Fatal()
			}
		})
	}
}
```

Плюсы:

- легко добавлять новые кейсы
- единый формат тестов
- удобно тестировать edge cases

---

## Подтесты (`t.Run`)

Позволяют группировать проверки:
``` go
t.Run("case name", func(t *testing.T) {
	
	}
)
```

Запуск конкретного подтеста:
``` go
go test -run TestAdd/positive
```

---
## Параллельные тесты

``` go
func TestSomething(t *testing.T) {
	t.Parallel()
}
```

Позволяет запускать независимые тесты параллельно.

---
## Cleanup

Для очистки ресурсов:

``` go
t.Cleanup(func() {
	// cleanup
})
```

Используется для:
- удаления файлов
- закрытия соединений
- очистки тестовых данных

---
## Helper

Помечает вспомогательную функцию:

``` go
func check(t *testing.T, value int) {
	t.Helper()

}
```

Ошибки будут показывать место вызова, а не helper.

---
## Проверка panic

Пример:
``` go
func TestPanic(t *testing.T) {

	defer func() {
		if recover() == nil {
			t.Fatal("expected panic")
		}
	}()

	Function()
}
```

---
## Запуск

Все тесты:
``` go
go test
```

С подробным выводом:
``` go
go test -v
```

Конкретный тест:
``` go
go test -run TestAdd
```

Все пакеты:
``` go
go test ./...
```

---
## Best practices

- Один тест — одна проверяемая идея
- Использовать table-driven tests
- Не зависеть от внешних сервисов
- Использовать интерфейсы для моков
- Проверять ошибки
- Проверять граничные случаи
- Не писать тесты ради покрытия, проверять поведение