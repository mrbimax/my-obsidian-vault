## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Setenv

## Назначение

`t.Setenv()` устанавливает **переменную окружения только на время выполнения теста**.

После завершения теста значение автоматически восстанавливается.

Используется для тестирования кода, который зависит от:

- `os.Getenv`;
    
- конфигурации через environment variables;
    
- feature flags;
    
- настроек окружения.
    

---

## Синтаксис

```go
t.Setenv(key, value string)
```

Пример:

```go
func TestConfig(t *testing.T) {

	t.Setenv(
		"APP_ENV",
		"test",
	)

	value := os.Getenv("APP_ENV")

	if value != "test" {
		t.Fatal("wrong env")
	}

}
```

---

## Что происходит внутри

До:

```text
APP_ENV=production
```

В тесте:

```go
t.Setenv(
	"APP_ENV",
	"test",
)
```

Получаем:

```text
APP_ENV=test
```

После завершения теста:

```text
APP_ENV=production
```

---

## Аналог без `Setenv`

До появления `t.Setenv` писали вручную:

```go
func TestConfig(t *testing.T) {

	old := os.Getenv("APP_ENV")

	os.Setenv(
		"APP_ENV",
		"test",
	)

	defer os.Setenv(
		"APP_ENV",
		old,
	)

}
```

Проблемы:

- легко забыть восстановление;
    
- код дублируется;
    
- при ошибке теста cleanup мог не выполниться.
    

`t.Setenv()` решает это автоматически.

---

## Использование с конфигурацией

Например:

```go
func LoadConfig() string {

	return os.Getenv("DATABASE_URL")

}
```

Тест:

```go
func TestLoadConfig(t *testing.T) {

	t.Setenv(
		"DATABASE_URL",
		"postgres://localhost/test",
	)

	result := LoadConfig()

	if result == "" {
		t.Fatal("empty database url")
	}

}
```

---

## Setenv и Cleanup

Внутри `Setenv` используется механизм:

```go
t.Cleanup()
```

Концептуально:

```go
func Setenv(key, value string) {

	old := os.Getenv(key)

	os.Setenv(key, value)

	t.Cleanup(func() {

		os.Setenv(key, old)

	})

}
```

После теста cleanup автоматически восстановит значение.

---

## Ограничение: нельзя использовать с Parallel

Нельзя:

```go
func TestEnv(t *testing.T) {

	t.Parallel()

	t.Setenv(
		"MODE",
		"test",
	)

}
```

Причина:

Environment variables — глобальное состояние процесса.

Если параллельные тесты изменяют:

```text
MODE=test1
MODE=test2
```

они будут влиять друг на друга.

---

## Влияние на subtests

Можно:

```go
func TestConfig(t *testing.T) {

	t.Setenv(
		"MODE",
		"test",
	)

	t.Run(
		"case1",
		func(t *testing.T) {

			fmt.Println(
				os.Getenv("MODE"),
			)

		},
	)

}
```

Subtest увидит:

```text
MODE=test
```

---

## Когда использовать

Хорошо подходит для:

- конфигурации приложения;
    
- переключателей функциональности;
    
- тестирования разных окружений.
    

Например:

```go
t.Setenv("LOG_LEVEL", "debug")
```

или:

```go
t.Setenv("FEATURE_NEW_API", "true")
```

---

## Для собеседования

> `t.Setenv()` временно изменяет переменную окружения во время теста и автоматически восстанавливает её после завершения через механизм cleanup. Используется для тестирования кода, который читает конфигурацию из environment variables. Из-за глобального характера окружения такие тесты нельзя запускать через `t.Parallel()`.