## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Run flag

## Назначение

Флаг `-run` позволяет запускать **только выбранные тесты или examples**, не выполняя остальные.

Фильтрация происходит по имени функции с помощью регулярного выражения (regexp).

---

## Синтаксис

Запустить один тест:

```bash
go test -run TestAdd
```

Запустить несколько тестов:

```bash
go test -run "Test(Add|Sub)"
```

Запустить все тесты:

```bash
go test -run .
```

---

## Пример

```go
func TestAdd(t *testing.T) {}
func TestSub(t *testing.T) {}
func TestMul(t *testing.T) {}
```

Команда:

```bash
go test -run TestSub
```

Запустит только:

```text
TestSub
```

---

## Работа с Subtests

Можно запускать конкретный subtest.

```go
func TestMath(t *testing.T) {

	t.Run("Add", func(t *testing.T) {})

	t.Run("Sub", func(t *testing.T) {})

}
```

Запуск:

```bash
go test -run "TestMath/Add"
```

Будет выполнен только subtest:

```text
TestMath
└── Add
```

---

## Запуск Example

```go
func ExampleAdd() {}
```

```bash
go test -run ExampleAdd
```

Запустит только этот Example.

---

## Что **не** запускает `-run`

`-run` влияет на:

- `TestXxx`
    
- `ExampleXxx`
    

Бенчмарки запускаются через:

```bash
go test -bench .
```

Fuzz-тесты:

```bash
go test -fuzz FuzzParse
```

---

## Для чего используется

- запуск одного теста при разработке;
    
- отладка упавшего теста;
    
- запуск конкретного subtest;
    
- ускорение цикла разработки.
    

---

## Для собеседования

> Флаг `-run` позволяет запускать только выбранные тесты или Example-функции. В качестве фильтра используется регулярное выражение, которое применяется к имени теста. Также можно запускать отдельные subtests, указывая путь через `/`, например `TestMath/Add`.