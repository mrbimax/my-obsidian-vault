## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz regression tests

## Назначение

Fuzz regression tests — превращение найденных fuzzing ошибок в **постоянные регрессионные тесты**.

Идея:

> Если fuzzing один раз нашёл баг, этот input должен навсегда проверяться при обычном `go test`.

---

## Жизненный цикл бага

```text
fuzzing
   |
   ↓
найден плохой input
   |
   ↓
сохранение в corpus
   |
   ↓
исправление бага
   |
   ↓
go test проверяет input всегда
```

---

## Пример

Есть баг:

```go
func Parse(data []byte) error {

	if string(data) == "BUG" {
		panic("broken parser")
	}

	return nil
}
```

Fuzz:

```go
func FuzzParse(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

---

Fuzzer находит:

```text
BUG
```

Получаем:

```text
panic: broken parser
```

Go сохраняет вход:

```text
parser/
├── parser.go
├── parser_test.go
│
└── testdata/
    └── fuzz/
        └── FuzzParse/
            └── abc123
```

---

## После исправления

Исправляем код:

```go
func Parse(data []byte) error {

	if string(data) == "BUG" {
		return errors.New("invalid data")
	}

	return nil
}
```

Теперь:

```bash
go test ./...
```

снова запускает:

```text
testdata/fuzz/FuzzParse/abc123
```

Проверяет:

```text
BUG → не падает
```

---

## Почему это регрессионный тест

Обычный тест:

```go
func TestParse(t *testing.T) {

	Parse([]byte("BUG"))

}
```

Ты должен вручную написать.

---

Fuzz regression:

```text
fuzzer нашёл случай
        ↓
Go сохранил его
        ↓
он автоматически стал тестом
```

Разработчик может не знать о конкретном кейсе, но он уже защищён.

---

## Где хранятся найденные случаи

Структура:

```text
testdata/
└── fuzz/
    └── FuzzParse/
        ├── abc123
        ├── def456
        └── ghi789
```

Каждый файл:

- отдельный вход fuzz-теста;
    
- воспроизводимый пример;
    
- часть regression suite.
    

---

## Отличие от seed corpus

### Seed corpus

Создаёт разработчик:

```go
f.Add([]byte("hello"))
```

Назначение:

- стартовые примеры;
    
- важные сценарии;
    
- помощь fuzzing.
    

---

### Regression corpus

Создаёт fuzzing engine:

```text
panic case
coverage case
bug reproducer
```

Назначение:

- защита от возврата старого бага.
    

---

## Запуск

Обычный тест:

```bash
go test ./...
```

Запустит:

- обычные тесты;
    
- сохранённые fuzz regression cases.
    

---

Fuzzing:

```bash
go test -fuzz=FuzzParse
```

Продолжит искать новые случаи.

---

## Практический пример из backend

Допустим есть JSON parser:

```go
func ParseUser(data []byte) error
```

Fuzz нашёл:

```json
{
    "age": -999999999
}
```

который ломал программу.

После фикса:

```go
if age < 0 {
	return errors.New("invalid age")
}
```

файл:

```text
testdata/fuzz/FuzzParseUser/crash123
```

остаётся в проекте.

Через год кто-то меняет parser:

```go
int32(age)
```

и снова появляется проблема.

CI сразу поймает:

```text
FAIL: FuzzParseUser
```

---

## Для собеседования

> Fuzz regression tests — это сохранённые fuzz cases, которые Go автоматически использует как обычные регрессионные тесты. Когда fuzzing находит panic или нарушение проверки, вход сохраняется в corpus. Этот input коммитится в репозиторий и при последующих запусках `go test` проверяется снова, предотвращая возврат старого бага.