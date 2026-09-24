## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Short mode

## Назначение

`testing.Short()` позволяет определить, запущены ли тесты в **коротком режиме** (`-short`).

Используется для пропуска:

- долгих тестов;    
- интеграционных тестов;    
- тестов с внешними сервисами.    

---
## Синтаксис

```go
testing.Short()
```

Возвращает:

```go
bool
```

---
## Пример

```go
package main

import (
	"testing"
)

func TestIntegration(t *testing.T) {

	if testing.Short() {
		t.Skip("skip integration test")
	}

	// долгий тест
}
```

---
## Запуск

Обычный режим:

```bash
go test
```

`testing.Short()` вернёт:

```go
false
```

Короткий режим:

```bash
go test -short
```

`testing.Short()` вернёт:

```go
true
```

---
## Типичное использование

```go
package main

import (
	"testing"
	"time"
)

func TestLongOperation(t *testing.T) {

	if testing.Short() {
		t.Skip("skipping long-running test")
	}

	time.Sleep(5 * time.Second)
}
```

---
## Когда использовать

- интеграционные тесты;    
- тесты БД;    
- HTTP/API тесты;    
- Docker-тесты;    
- любые долгие тесты.    

---
## Для собеседования

> `testing.Short()` позволяет узнать, запущены ли тесты с флагом `-short`. Обычно используется вместе с `t.Skip()` для пропуска долгих или интеграционных тестов и ускорения локального запуска тестов.