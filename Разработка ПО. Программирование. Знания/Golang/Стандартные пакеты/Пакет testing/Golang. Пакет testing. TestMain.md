## *[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]* 
---
## Golang. Пакет testing. TestMain

`TestMain` — специальная функция для управления жизненным циклом тестов всего пакета.

Используется для:
- общего setup перед всеми тестами;    
- общего teardown после всех тестов;    
- инициализации глобальных ресурсов.    

Типичные случаи:
- подключение к тестовой БД;    
- запуск тестового сервера;    
- применение миграций;    
- очистка общих ресурсов.    

---
## Синтаксис

```go
func TestMain(m *testing.M)
```

`m.Run()` запускает все тесты пакета.

---
## Пример

```go
package main

import (
	"os"
	"testing"
)

func TestMain(m *testing.M) {

	// setup
	initDatabase()

	code := m.Run()

	// teardown
	closeDatabase()

	os.Exit(code)
}
```

---
## Порядок выполнения

```
TestMain
    ↓
setup
    ↓
TestXxx
TestYyy
BenchmarkXxx
    ↓
teardown
    ↓
os.Exit(code)
```

---
## Важный момент

Если вызвать `os.Exit()` до `m.Run()`:
```go
func TestMain(m *testing.M) {

	setup()

	os.Exit(0)

	m.Run()
}
```

Тесты **не запустятся**.

Правильно:
```go
code := m.Run()
os.Exit(code)
```

---
## Пример с тестовым сервером

```go
package main

import (
	"net/http/httptest"
	"os"
	"testing"
)

var server *httptest.Server

func TestMain(m *testing.M) {

	server = httptest.NewServer(handler())

	code := m.Run()

	server.Close()

	os.Exit(code)
}
```

Все тесты используют один сервер.

---
## Отличие `TestMain` и `t.Cleanup`

||TestMain|t.Cleanup|
|---|---|---|
|Область действия|весь пакет|один тест|
|Запуск|один раз|каждый тест|
|Использование|глобальные ресурсы|локальные ресурсы|
|Пример|БД, сервер|файл, connection|

---
## Важные особенности

- функция должна называться именно `TestMain`;    
- находится в файле `*_test.go`;    
- может быть только одна на пакет;    
- если `TestMain` отсутствует — Go запускает тесты автоматически;    
- нужно обязательно вызвать `m.Run()`.    
---
## Для собеседования

> `TestMain` — это точка входа для управления lifecycle тестов пакета. Он выполняется один раз перед всеми тестами и после них. Через него обычно делают общий setup/teardown: подключение к БД, запуск серверов, миграции. Для ресурсов конкретного теста лучше использовать `t.Cleanup`.