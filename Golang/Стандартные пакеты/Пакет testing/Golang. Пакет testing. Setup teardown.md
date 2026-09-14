## *[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]*
---
## Golang. Пакет testing. Setup teardown

Setup / teardown — подготовка и очистка окружения для тестов.

Используется для:
- создания тестовых данных
- поднятия ресурсов
- подключения к БД
- освобождения ресурсов после теста

В Go нет встроенных `beforeEach` / `afterEach`, вместо этого используются:
- обычный код перед тестом
- `t.Cleanup`
- `TestMain`
---
## Локальный setup / teardown

Пример:
```go
func TestUser(t *testing.T) {

	// setup
	db := setupDB()

	// teardown
	t.Cleanup(func() {
		db.Close()
	})

	// test
	// работа с db
}
```

Порядок:
```text
setup
  ↓
test
  ↓
cleanup
```
---

## `t.Cleanup`

Основной способ очистки ресурсов.
```go
t.Cleanup(func() {
	// очистка
})
```

Особенности:
- выполняется после завершения теста;
- вызывается даже если тест упал через `t.Fatal`;
- несколько cleanup выполняются в обратном порядке (LIFO).

Пример:
```go
func TestFile(t *testing.T) {

	file := createTempFile()

	t.Cleanup(func() {
		deleteFile(file)
	})

	// тест
}
```
---
## Setup для группы subtests
```go
func TestAPI(t *testing.T) {

	server := startServer()

	t.Cleanup(func() {
		server.Close()
	})

	t.Run("GET", func(t *testing.T) {
		// использует server
	})

	t.Run("POST", func(t *testing.T) {
		// использует server
	})
}
```

Структура:
```text
TestAPI
 ├── setup server
 ├── GET
 ├── POST
 └── cleanup server
```
---
## `TestMain`

Используется для setup/teardown всего пакета.

Сигнатура:
```go
func TestMain(m *testing.M)
```

Пример:
```go
func TestMain(m *testing.M) {

	// setup
	initDatabase()

	code := m.Run()

	// teardown
	closeDatabase()

	os.Exit(code)
}
```

Подходит для:
- тестовой БД;
- миграций;    
- глобальных ресурсов;    
- конфигурации.   

---
## `TestMain` порядок выполнения

```text
TestMain
   |
   ├── setup
   |
   ├── TestXxx
   ├── TestYyy
   ├── Benchmark
   |
   └── teardown
```
---
## Временные ресурсы

Встроенные helpers:
### `t.TempDir`

Создаёт временную директорию:
```go
func TestFile(t *testing.T) {

	dir := t.TempDir()

	// работа с файлами
}
```

Удаление происходит автоматически.

---
## Важные правила

- Использовать `t.Cleanup` для ресурсов конкретного теста    
- Использовать `TestMain` для ресурсов всего пакета    
- Не хранить состояние между тестами    
- Каждый тест должен быть независимым    
- Cleanup должен освобождать всё, что создал setup    