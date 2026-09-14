## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Examples

## Назначение

Examples — специальные тесты-документация, которые показывают, **как использовать API**.

Особенность:

> Example-функции одновременно являются документацией и автоматически проверяемыми тестами.

---

## Синтаксис

Имя функции:

```go
Example
```

или:

```go
ExampleType
```

или:

```go
ExampleType_Method
```

---

## Простой пример

Код:

```go
package calc

func Add(a, b int) int {
	return a + b
}
```

Example:

```go
func ExampleAdd() {

	result := Add(2, 3)

	fmt.Println(result)

	// Output:
	// 5
}
```

---

## Как работает Output

Комментарий:

```go
// Output:
// 5
```

— это ожидаемый результат.

Go:

1. запускает example;
    
2. перехватывает stdout;
    
3. сравнивает вывод с `Output`.
    

---

Если вывод отличается:

```text
got:
6

want:
5
```

тест падает.

---

## Example как документация

Example отображается в:

```bash
go doc
```

и на:

```text
pkg.go.dev
```

Например:

```go
func ExampleAdd() {

	fmt.Println(Add(1, 2))

	// Output:
	// 3
}
```

Пользователь библиотеки сразу видит:

```go
Add(1, 2)
```

---

# Example без Output

Можно написать:

```go
func ExampleServer() {

	server := NewServer()

	server.Start()

}
```

Такой example:

- выполняется;
    
- проверяет отсутствие panic;
    
- но не проверяет вывод.
    

---

# Example для типа

Есть тип:

```go
type User struct {
	Name string
}

func (u User) String() string {
	return u.Name
}
```

Example:

```go
func ExampleUser() {

	user := User{
		Name: "Max",
	}

	fmt.Println(user)

	// Output:
	// Max
}
```

---

# Example метода

Метод:

```go
func (u User) Validate() error {
	return nil
}
```

Example:

```go
func ExampleUser_Validate() {

	user := User{}

	err := user.Validate()

	fmt.Println(err)

	// Output:
	// <nil>
}
```

Имя:

```text
Example<Type>_<Method>
```

---

# Example для пакета

Можно создать:

```go
func Example() {

	fmt.Println("package example")

	// Output:
	// package example
}
```

Он показывает использование всего пакета.

---

# Example vs Test

## Test

Проверяет:

```go
func TestAdd(t *testing.T)
```

Цель:

- проверка логики;
    
- поиск ошибок.
    

---

## Example

Показывает:

```go
func ExampleAdd()
```

Цель:

- документация;
    
- пример использования;
    
- проверка API.
    

---

# Example vs Benchmark

Test:

```go
func TestXxx(t *testing.T)
```

Проверка корректности.

---

Benchmark:

```go
func BenchmarkXxx(b *testing.B)
```

Проверка производительности.

---

Example:

```go
func ExampleXxx()
```

Проверка документации и поведения.

---

# Где используется

Часто в публичных библиотеках:

- стандартная библиотека Go;
    
- open source пакеты;
    
- SDK.
    

Например:

```go
func ExampleSort() {

	values := []int{3, 1, 2}

	sort.Ints(values)

	fmt.Println(values)

	// Output:
	// [1 2 3]
}
```

---

# Ограничения

Example должен быть:

- детерминированным;
    
- с предсказуемым выводом;
    
- быстрым.
    

Плохо:

```go
func ExampleTime() {

	fmt.Println(time.Now())

}
```

Потому что результат меняется.

---

## Для собеседования

> Examples в пакете testing — это тесты, которые одновременно служат документацией. Они запускаются как обычные тесты, а комментарий `// Output:` используется для проверки stdout. Examples показывают пользователям правильный способ работы с API и отображаются в Go documentation.