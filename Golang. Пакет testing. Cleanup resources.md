## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Cleanup resources

## Назначение

Cleanup resources — освобождение ресурсов после завершения теста.

Главная идея:

> Всё, что тест создал, он должен удалить или закрыть.

Для этого используется:

```go
t.Cleanup()
```

---

## Какие ресурсы нужно освобождать

- файлы;
    
- соединения с БД;
    
- HTTP-серверы;
    
- сокеты;
    
- временные директории (если не используется `t.TempDir()`);
    
- goroutines;
    
- mock-серверы.
    

---

## Закрытие файла

```go
func TestFile(t *testing.T) {

	file, err := os.Create("test.txt")

	if err != nil {
		t.Fatal(err)
	}

	t.Cleanup(func() {

		file.Close()

	})

}
```

---

## Удаление файла

```go
func TestFile(t *testing.T) {

	file, err := os.Create("test.txt")

	if err != nil {
		t.Fatal(err)
	}

	t.Cleanup(func() {

		file.Close()
		os.Remove("test.txt")

	})

}
```

---

## Закрытие БД

```go
func TestRepository(t *testing.T) {

	db := setupDB()

	t.Cleanup(func() {

		db.Close()

	})

}
```

---

## Закрытие HTTP-сервера

```go
func TestAPI(t *testing.T) {

	server := httptest.NewServer(handler)

	t.Cleanup(func() {

		server.Close()

	})

}
```

---

## Остановка goroutine

```go
func TestWorker(t *testing.T) {

	ctx, cancel := context.WithCancel(
		context.Background(),
	)

	t.Cleanup(func() {

		cancel()

	})

	go worker(ctx)

}
```

---

## Cleanup выполняется всегда

Даже если тест завершился через:

```go
t.Fatal(...)
```

или

```go
t.FailNow()
```

зарегистрированные функции `Cleanup` всё равно будут вызваны.

---

## Автоматический Cleanup

Некоторые методы сами используют `t.Cleanup()`:

```go
t.TempDir()
```

```go
t.Setenv()
```

```go
t.Chdir()
```

Поэтому вручную освобождать их ресурсы не нужно.

---

## Порядок выполнения

Если зарегистрировано несколько функций:

```go
t.Cleanup(func() {
	fmt.Println("1")
})

t.Cleanup(func() {
	fmt.Println("2")
})
```

Результат:

```text
2
1
```

Cleanup выполняется в порядке **LIFO (Last In, First Out)**.

---

## Для собеседования

> Освобождение ресурсов в тестах обычно выполняется через `t.Cleanup()`. Он гарантирует, что зарегистрированная функция будет вызвана после завершения теста, даже если тест завершился через `t.Fatal()`. Это основной механизм для закрытия файлов, соединений с БД, HTTP-серверов, остановки goroutines и других ресурсов. Некоторые методы (`t.TempDir()`, `t.Setenv()`, `t.Chdir()`) уже используют `t.Cleanup()` внутри.