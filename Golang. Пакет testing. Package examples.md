## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Package examples

## Назначение

Package example — пример использования **всего пакета**, а не отдельной функции или метода.

Используется, чтобы показать типичный сценарий работы с библиотекой.

---

## Синтаксис

Имя функции:

```go
func Example()
```

Без суффиксов.

---

## Пример

Пакет:

```go
package calc

func Add(a, b int) int {
	return a + b
}

func Sub(a, b int) int {
	return a - b
}
```

Example:

```go
func Example() {

	sum := Add(2, 3)
	diff := Sub(5, 2)

	fmt.Println(sum, diff)

	// Output:
	// 5 3
}
```

---

## Когда использовать

Подходит, если нужно показать:

- типичный сценарий использования библиотеки;
    
- последовательность вызовов нескольких функций;
    
- минимальный рабочий пример.
    

Например:

```go
func Example() {

	client := NewClient()

	client.Connect()

	client.Send("Hello")

	client.Close()

	// Output:
	// connected
	// message sent
	// disconnected
}
```

---

## Package example vs Function example

### Package example

```go
func Example()
```

Показывает использование **пакета целиком**.

---

### Function example

```go
func ExampleAdd()
```

Показывает использование **конкретной функции**.

---

### Method example

```go
func ExampleClient_Send()
```

Показывает использование **метода**.

---

## Где отображается

Package example отображается в документации пакета (`pkg.go.dev`) как основной пример использования библиотеки.

---

## Для собеседования

> `Example()` — это пример использования всего пакета. В отличие от `ExampleXxx()` и `ExampleType_Method()`, он демонстрирует типичный сценарий работы с библиотекой и одновременно выполняется как тест, если содержит комментарий `// Output:`.