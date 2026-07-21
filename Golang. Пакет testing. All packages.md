## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. All packages

## Назначение

Флаг `./...` позволяет запускать тесты **во всех пакетах текущего модуля и его подпакетах**.

Используется для:

- полного прогона тестов проекта;
    
- CI/CD проверки;
    
- поиска проблем в зависимостях между пакетами.
    

---

## Синтаксис

```bash
go test ./...
```

Разбор:

```text
./
```

текущая директория

```text
...
```

все вложенные пакеты

---

## Пример структуры проекта

```text
project/
├── go.mod
│
├── cmd/
│   └── app/
│       └── main.go
│
├── internal/
│   ├── user/
│   │   ├── user.go
│   │   └── user_test.go
│   │
│   └── payment/
│       ├── payment.go
│       └── payment_test.go
│
└── pkg/
    └── logger/
        ├── logger.go
        └── logger_test.go
```

Команда:

```bash
go test ./...
```

запустит:

```text
internal/user
internal/payment
pkg/logger
cmd/app
```

---

## Отличие от `go test`

Без аргументов:

```bash
go test
```

Запускает только текущий пакет.

Например:

```text
project/internal/user
```

проверит только:

```text
internal/user
```

---

С `./...`:

```bash
go test ./...
```

проверяет весь проект.

---

## Пример вывода

```text
?       example/cmd/app       [no test files]
ok      example/internal/user 0.02s
ok      example/internal/payment 0.01s
ok      example/pkg/logger    0.01s
```

---

## Часто используют вместе с флагами

Verbose:

```bash
go test ./... -v
```

Coverage:

```bash
go test ./... -cover
```

Race detector:

```bash
go test ./... -race
```

Timeout:

```bash
go test ./... -timeout 5m
```

JSON:

```bash
go test ./... -json
```

---

## Исключение пакетов

Go не имеет встроенного флага:

```bash
go test ./... --exclude package
```

Обычно используют:

- regexp фильтры;
    
- отдельные команды CI;
    
- Makefile.
    

Например:

```bash
go test $(go list ./... | grep -v /integration/)
```

---

## Связь с модулями

`./...` работает внутри текущего Go module:

```text
go.mod
```

Например:

```text
module github.com/example/project
```

Команда:

```bash
go test ./...
```

обрабатывает пакеты этого модуля.

---

## Для собеседования

> `go test ./...` запускает тесты во всех пакетах текущего Go module и его подпакетах. В отличие от `go test`, который проверяет только текущий пакет, `./...` обычно используется в CI/CD для полного прогона проекта вместе с флагами `-race`, `-cover`, `-v` и другими.