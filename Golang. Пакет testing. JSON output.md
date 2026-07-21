## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. JSON output

## Назначение

Флаг `-json` включает вывод результатов тестирования в формате **JSON**.

Используется для:

- интеграции с CI/CD;
    
- обработки результатов тестов инструментами;
    
- построения отчётов;
    
- анализа времени выполнения тестов.
    

---

## Синтаксис

```bash
go test -json
```

Для всех пакетов:

```bash
go test ./... -json
```

---

## Обычный вывод

Команда:

```bash
go test
```

Вывод:

```text
ok      example/user    0.002s
```

---

## JSON вывод

Команда:

```bash
go test -json
```

Вывод:

```json
{"Time":"2026-07-21T10:00:00Z","Action":"run","Package":"example/user","Test":"TestCreateUser"}
{"Time":"2026-07-21T10:00:00Z","Action":"output","Package":"example/user","Test":"TestCreateUser","Output":"=== RUN   TestCreateUser\n"}
{"Time":"2026-07-21T10:00:00Z","Action":"pass","Package":"example/user","Test":"TestCreateUser","Elapsed":0}
```

---

## Типы событий (`Action`)

### run

Тест начал выполняться:

```json
{
  "Action": "run",
  "Test": "TestUser"
}
```

---

### output

Тест вывел сообщение:

Например:

```go
t.Log("created user")
```

↓

```json
{
  "Action": "output",
  "Output": "created user\n"
}
```

---

### pass

Тест успешно завершился:

```json
{
  "Action": "pass",
  "Test": "TestUser"
}
```

---

### fail

Тест упал:

```json
{
  "Action": "fail",
  "Test": "TestUser"
}
```

---

### skip

Тест пропущен:

```json
{
  "Action": "skip",
  "Test": "TestIntegration"
}
```

---

## JSON и Subtests

Код:

```go
func TestUser(t *testing.T) {

	t.Run("Create", func(t *testing.T) {

	})

	t.Run("Delete", func(t *testing.T) {

	})

}
```

JSON будет содержать отдельные события:

```text
run TestUser

run TestUser/Create
pass TestUser/Create

run TestUser/Delete
pass TestUser/Delete

pass TestUser
```

---

## JSON и CI/CD

Обычный вывод:

```text
PASS
ok example
```

сложно анализировать программно.

JSON:

```json
{
  "Action": "fail",
  "Test": "TestPayment"
}
```

можно обработать:

- GitLab CI;
    
- GitHub Actions;
    
- Jenkins;
    
- тестовыми репортерами.
    

---

## Сохранение результата

Например:

```bash
go test ./... -json > test-results.json
```

Теперь:

```text
test-results.json
```

содержит полный журнал выполнения.

---

## Связь с `go tool test2json`

Внутри:

```bash
go test -json
```

использует тот же формат, который создаёт:

```bash
go tool test2json
```

Он преобразует стандартный вывод тестового бинарника в JSON-события.

---

## Для собеседования

> Флаг `-json` преобразует вывод `go test` в поток JSON-событий. Каждое событие описывает действие тестового раннера: запуск, вывод, успешное завершение, ошибку или пропуск теста. Используется в CI/CD и инструментах анализа результатов тестирования.