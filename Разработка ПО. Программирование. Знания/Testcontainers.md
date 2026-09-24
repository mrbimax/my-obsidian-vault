---
tags:
  - golang
  - testing
  - docker
  - architecture
  - senior
---
ё# Testcontainers в Go: Senior Cheat Sheet

**Testcontainers** — это библиотека, позволяющая запускать реальные зависимости (БД, брокеры, кэши) в [[Docker]]-контейнерах прямо из Go-кода перед выполнением тестов. 

## 1. Архитектурные паттерны интеграции

### А. Паттерн "Single Container Lifecycle" (Global Suite)
Используется для тяжелых контейнеров (например, Kafka, ClickHouse), которые инициализируются **один раз** на весь тестовый запуск (через `TestMain`).

```go
// internal/testutil/postgres.go
package testutil

import (
	"context"
	"database/sql"
	"log"
	"testing"

	_ "github.com/lib/pq"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go/modules/postgres](https://github.com/testcontainers/testcontainers-go/modules/postgres)"
)

var DB *sql.DB

func RunTestContainersSuite(m *testing.M) int {
	ctx := context.Background()

	pgContainer, err := postgres.Run(ctx,
		"postgres:16-alpine",
		postgres.WithDatabase("testdb"),
		postgres.WithUsername("user"),
		postgres.WithPassword("password"),
	)
	if err != nil {
		log.Fatalf("failed to start container: %s", err)
	}

	// Гарантируем очистку при завершении процесса
	defer func() {
		if err := pgContainer.Terminate(ctx); err != nil {
			log.Printf("failed to terminate container: %s", err)
		}
	}()

	connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
	if err != nil {
		log.Fatalf("failed to get connection string: %s", err)
	}

	DB, err = sql.Open("postgres", connStr)
	if err != nil {
		log.Fatalf("failed to open db: %s", err)
	}

	return m.Run()
}
```

```go
// main_test.go
package main

import (
	"os"
	"testing"
	"your-project/internal/testutil"
)

func TestMain(m *testing.M) {
	code := testutil.RunTestContainersSuite(m)
	os.Exit(code)
}
```
### Б. Паттерн "Isolated Containers per Test" (С фокусом на параллелизацию)

Используется для быстрых контейнеров (Redis) или когда тесты мутируют глобальное состояние схемы/данных так, что это мешает соседним тестам. Запускается через `t.Run` в связке с `t.Parallel()`.

## 2. Оптимизация производительности (Speed Up)

Запуск контейнеров на каждый `go test` — это дорого. Способы ускорения CI/CD и локальной разработки:

### 🚀 Включение Reusable Containers (Локальная разработка)

Позволяет повторно использовать уже запущенный контейнер между разными запусками `go test`, если его конфигурация не изменилась.

1. В корне домашней директории создай/дополни файл `~/.testcontainers.properties`:
    ```python
    testcontainers.reuse.enable=true
    ```
2. В коде добавь опцию `testcontainers.WithReuse()`:
    ```go
    pgContainer, err := postgres.Run(ctx,
        "postgres:16-alpine",
        testcontainers.WithReuse(true), // <- Важно
    )
    ```

> ⚠️ **Senior Warning:** При использовании `WithReuse` состояние БД (данные) сохраняется между запусками тестов. Обязательно накатывай очистку таблиц (Truncate) перед/после каждого теста.

### 🔄 Пул соединений и очистка данных вместо перезапуска

Вместо пересоздания контейнера для изоляции тестов, используй одну БД, но заворачивай каждый тест в **БД-транзакцию с Rollback**, либо запускай очистку:

Go

```go
func CleanTables(t *testing.T, db *sql.DB, tables ...string) {
	t.Helper()
	for _, table := range tables {
		_, err := db.Exec(fmt.Sprintf("TRUNCATE TABLE %s RESTART IDENTITY CASCADE;", table))
		if err != nil {
			t.Fatalf("failed to truncate table %s: %v", table, err)
		}
	}
}
```

## 3. Продвинутые техники (Advanced Tricks)

### 🩺 Custom Wait Strategies (Стратегии ожидания readiness)

По умолчанию Testcontainers ждет, пока порт станет доступен. Но если это БД, она может быть еще не готова принимать запросы.

Go

```go
import "[github.com/testcontainers/testcontainers-go/wait](https://github.com/testcontainers/testcontainers-go/wait)"

req := testcontainers.ContainerRequest{
    Image:        "redis:7-alpine",
    ExposedPorts: []string{"6379/tcp"},
    WaitingFor:   wait.ForLog("Ready to accept connections"), // Ждем лог
    // Или по HTTP:
    // WaitingFor: wait.ForHTTP("/healthz").WithPort("8080"),
    // Или SQL-запросом:
    // WaitingFor: wait.ForSQL("5432/tcp", "postgres", func(host string, port nat.Port) string { ... }),
}
```

### 🌉 Работа в Docker-in-Docker (CI/CD Pipelines)

Если тесты запускаются внутри GitLab CI / GitHub Actions, которые сами работают в Docker:

- Убедись, что сокет `/var/run/docker.sock` прокинут внутрь CI-раннера.
    
- Выстави переменную окружения `TESTCONTAINERS_RYUK_DISABLED=true`, если раннер имеет жесткие ограничения по правам и встроенный сборщик мусора Ryuk не может запуститься.
    

### 🐳 Использование сетей (Network Topology)

Если нужно поднять связку из двух контейнеров (например, App + DB), чтобы они видели друг друга:

Go

```go
newNetwork, err := testcontainers.GenericNetwork(ctx, testcontainers.GenericNetworkRequest{
    Network %+testcontainers.NetworkRequest{Name: "test-network"},
})

dbContainer, err := postgres.Run(ctx,
    "postgres:16-alpine",
    testcontainers.WithNetwork(newNetwork),
    testcontainers.WithNetworkAliases("db-host"),
)
```

## 4. Общие проблемы и их решение (Troubleshooting)

|**Проблема**|**Причина**|**Решение**|
|---|---|---|
|**"Port already allocated"**|Статический маппинг портов в коде.|**Никогда** не хардкодь порты хоста (например, `"5432:5432"`). Testcontainers автоматически маппит порт контейнера на случайный свободный порт хоста. Получай его через `.MappedPort(ctx, "5432")`.|
|**Утечка контейнеров (Zombies)**|Тест упал по `panic`, `os.Exit` или прерван по `Ctrl+C`.|За это отвечает контейнер **Ryuk** (Moby Myna). Не отключай его локально. Он подчищает всё, что было помечено специальными лейблами при старте.|
|**Медленный запуск в CI**|Каждый раз скачивается образ (Pull).|Настрой локальный Docker Registry/Proxy в вашей инфраструктуре и передавай кастомный репозиторий через префикс образа или переменные окружения.|

## 5. Достойные альтернативы (Когда Testcontainers — оверхед?)

1. **SQLite (in-memory) / pgxmock** — если тестируется чистая бизнес-логика и нет специфичных для конкретной СУБД фич (типа JSONB-индексов, оконных функций Postgres).
2. **Docker Compose через `dockertest` (ory/dockertest)** — старая школа. Хорош, но инфраструктура контейнеров описывается менее нативно для Go-разработчика, чем в Testcontainers-go.