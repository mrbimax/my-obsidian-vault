## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Resources management

## Назначение

Resource management — управление ресурсами во время тестов.

Ресурсы:

- файлы;
    
- временные директории;
    
- соединения с БД;
    
- HTTP-серверы;
    
- goroutines;
    
- mock-объекты;
    
- внешние зависимости.
    

Главная идея:

> Тест должен сам создавать и освобождать ресурсы, чтобы не влиять на другие тесты.

---

# Проблема без управления ресурсами

Плохо:

```go
func TestDatabase(t *testing.T) {

	db, _ := sql.Open(
		"postgres",
		"connection",
	)

	// тест

}
```

Проблемы:

- соединение остаётся открытым;
    
- тесты могут влиять друг на друга;
    
- при `t.Fatal()` cleanup не выполнится.
    

---

# t.Cleanup()

Основной механизм управления ресурсами.

Синтаксис:

```go
t.Cleanup(func() {

	// освобождение ресурса

})
```

Пример:

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

После теста:

```text
ресурс создан
      |
      ↓
тест выполняется
      |
      ↓
cleanup вызывается автоматически
```

---

# Порядок выполнения Cleanup

Если зарегистрировано несколько:

```go
func TestResource(t *testing.T) {

	t.Cleanup(func() {
		fmt.Println("first")
	})

	t.Cleanup(func() {
		fmt.Println("second")
	})

}
```

Порядок:

```text
second
first
```

Cleanup работает как stack:

```text
LIFO

Last In First Out
```

---

# t.TempDir()

Создание временной директории:

```go
func TestFiles(t *testing.T) {

	dir := t.TempDir()

	fmt.Println(dir)

}
```

Go:

1. создаёт директорию;
    
2. возвращает путь;
    
3. удаляет после теста.
    

---

Пример:

```go
func TestConfig(t *testing.T) {

	dir := t.TempDir()

	file := filepath.Join(
		dir,
		"config.json",
	)

	os.WriteFile(
		file,
		[]byte("{}"),
		0644,
	)

}
```

После:

```text
/tmp/TestConfig123/
        |
        ↓
удалена
```

---

# t.Setenv()

Управление переменными окружения:

```go
func TestConfig(t *testing.T) {

	t.Setenv(
		"APP_ENV",
		"test",
	)

}
```

После теста:

```text
APP_ENV
   |
   ↓
старое значение
```

---

# t.Chdir()

Управление рабочей директорией:

```go
func TestCLI(t *testing.T) {

	dir := t.TempDir()

	t.Chdir(dir)

}
```

После:

```text
cwd восстановлен
```

---

# Серверы

Пример с HTTP сервером:

```go
func TestAPI(t *testing.T) {

	server := httptest.NewServer(
		handler,
	)

	t.Cleanup(func() {

		server.Close()

	})

}
```

После теста:

```text
server запущен
      |
      ↓
тест
      |
      ↓
server.Close()
```

---

# База данных

Пример:

```go
func TestRepository(t *testing.T) {

	db := setupDB()

	t.Cleanup(func() {

		db.Close()

	})

}
```

---

# Goroutines

Опасно:

```go
func TestWorker(t *testing.T) {

	go worker()

}
```

После теста:

```text
goroutine продолжает работать
```

Лучше:

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

# Правило хорошего теста

Ресурс должен иметь владельца:

```text
Test
 |
 ├── create resource
 |
 ├── use resource
 |
 └── cleanup resource
```

---

# Что использовать

## Файлы

Использовать:

```go
t.TempDir()
```

---

## Environment variables

Использовать:

```go
t.Setenv()
```

---

## Рабочая директория

Использовать:

```go
t.Chdir()
```

---

## Произвольное освобождение

Использовать:

```go
t.Cleanup()
```

---

## Для собеседования

> В Go тестах управление ресурсами обычно строится через `t.Cleanup()`. Он гарантирует выполнение освобождения ресурсов даже при `t.Fatal()`. Для стандартных случаев есть специальные помощники: `t.TempDir()` для временных файлов, `t.Setenv()` для переменных окружения и `t.Chdir()` для рабочей директории. Хороший тест должен полностью контролировать жизненный цикл создаваемых ресурсов и не оставлять побочных эффектов.