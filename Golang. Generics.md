# Golang Generics (Go 1.18+)

> **Generics** — это параметризация типов. Позволяют писать один алгоритм для разных типов без использования `interface{}` и без дублирования кода.

---
# Зачем нужны

До Go 1.18 часто приходилось писать:

```go
func MaxInt(a, b int) int
func MaxFloat(a, b float64) float64
func MaxString(a, b string) string
```

или использовать

```go
interface{}
```

что:
- не типобезопасно;
- требует type assertion;
- ошибки проявляются во время выполнения.
Generics решают эту проблему.

---
# Синтаксис

```go
func Name[T any](arg T) T
```

- `T` — параметр типа.
- `any` — ограничение (constraint).

---
# Самый простой пример

```run-go
package main

import "fmt"

func Print[T any](v T) {
	fmt.Println(v)
}

func main() {
	Print(10)
	Print("hello")
	Print(true)
}
```

Вывод

```
10
hello
true
```

---
# Несколько параметров типа

```run-go
package main

import "fmt"

func Pair[K any, V any](key K, value V) {
	fmt.Printf("%v -> %v\n", key, value)
}

func main() {
	Pair("id", 10)
	Pair(1, "Alice")
}
```

---
# Generic-функция Max

Нельзя написать

```go
func Max[T any](a, b T) T {
	if a > b {
		return a
	}
	return b
}
```

Потому что оператор `>` определён не для всех типов.

Нужно ограничить допустимые типы.

---
# Constraint

Создадим собственное ограничение ([[Golang. Generics. Constraint|constraint]]).

```run-go
package main

import "fmt"

type Ordered interface {
	~int |
		~int64 |
		~float64 |
		~string
}

func Max[T Ordered](a, b T) T {
	if a > b {
		return a
	}
	return b
}

func main() {
	fmt.Println(Max(10, 20))
	fmt.Println(Max(3.14, 2.71))
	fmt.Println(Max("cat", "apple"))
}
```

Вывод

```
20
3.14
cat
```

---
# Что означает ~

Например

```go
type MyInt int
```

Если написать

```go
interface {
	int
}
```

то `MyInt` **не подойдёт**.

Если написать

```go
interface {
	~int
}
```

то подойдут

- int
- MyInt
- любые типы с базовым типом int.

Пример.

```run-go
package main

import "fmt"

type MyInt int

type IntLike interface {
	~int
}

func Double[T IntLike](v T) T {
	return v * 2
}

func main() {
	var x MyInt = 15

	fmt.Println(Double(x))
}
```

---

# Компилятор выводит тип сам

Не обязательно писать

```go
Max[int](10, 20)
```

Можно

```go
Max(10, 20)
```

Go сам определит

```
T = int
```

---

# Явное указание типа

Иногда это необходимо.

```run-go
package main

import "fmt"

func Zero[T any]() T {
	var v T
	return v
}

func main() {
	fmt.Println(Zero[int]())
	fmt.Println(Zero[string]())
}
```

Вывод

```
0

```

---

# Generic-структуры

Можно параметризовать не только функции.

```run-go
package main

import "fmt"

type Box[T any] struct {
	Value T
}

func main() {
	a := Box[int]{Value: 10}
	b := Box[string]{Value: "Go"}

	fmt.Println(a.Value)
	fmt.Println(b.Value)
}
```

---

# Generic-методы

```run-go
package main

import "fmt"

type Box[T any] struct {
	Value T
}

func (b Box[T]) Get() T {
	return b.Value
}

func main() {
	box := Box[string]{Value: "hello"}

	fmt.Println(box.Get())
}
```

---

# Generic-слайс

```run-go
package main

import "fmt"

func Reverse[T any](items []T) {
	left := 0
	right := len(items) - 1

	for left < right {
		items[left], items[right] =
			items[right], items[left]

		left++
		right--
	}
}

func main() {
	a := []int{1, 2, 3, 4}
	b := []string{"A", "B", "C"}

	Reverse(a)
	Reverse(b)

	fmt.Println(a)
	fmt.Println(b)
}
```

---

# Generic-map

```run-go
package main

import "fmt"

func Keys[K comparable, V any](m map[K]V) []K {
	keys := make([]K, 0, len(m))

	for k := range m {
		keys = append(keys, k)
	}

	return keys
}

func main() {
	m := map[string]int{
		"A": 10,
		"B": 20,
	}

	fmt.Println(Keys(m))
}
```

---

# Почему ключ map ограничен comparable

Ключи map должны поддерживать операции

```
==
!=
```

Поэтому используется

```go
K comparable
```

а не

```go
K any
```

---

# comparable

Встроенное ограничение.

Подходят:

- int
- string
- bool
- указатели
- массивы
- структуры (если все поля comparable)

Не подходят:

- slice
- map
- function

Пример.

```go
map[[]int]int
```

не скомпилируется.

---

# Generic Stack

```run-go
package main

import "fmt"

type Stack[T any] struct {
	items []T
}

func (s *Stack[T]) Push(v T) {
	s.items = append(s.items, v)
}

func (s *Stack[T]) Pop() T {
	n := len(s.items)

	v := s.items[n-1]
	s.items = s.items[:n-1]

	return v
}

func main() {
	var s Stack[int]

	s.Push(10)
	s.Push(20)
	s.Push(30)

	fmt.Println(s.Pop())
	fmt.Println(s.Pop())
	fmt.Println(s.Pop())
}
```

---

# Generic Filter

```run-go
package main

import "fmt"

func Filter[T any](items []T, pred func(T) bool) []T {
	result := make([]T, 0)

	for _, item := range items {
		if pred(item) {
			result = append(result, item)
		}
	}

	return result
}

func main() {
	nums := []int{1, 2, 3, 4, 5, 6}

	even := Filter(nums, func(v int) bool {
		return v%2 == 0
	})

	fmt.Println(even)
}
```

---

# Generic Map (функция)

Аналог map() из других языков.

```run-go
package main

import "fmt"

func Map[T any, R any](items []T, f func(T) R) []R {
	result := make([]R, len(items))

	for i, v := range items {
		result[i] = f(v)
	}

	return result
}

func main() {
	nums := []int{1, 2, 3}

	squares := Map(nums, func(v int) int {
		return v * v
	})

	fmt.Println(squares)
}
```

---

# Ограничения Generics

Generics **не поддерживают** специализацию.

Нельзя написать

```go
func Foo[int](...)
```

как в C++.

---

Нельзя выполнять type switch над параметром типа напрямую.

Вместо

```go
switch v.(type)
```

обычно используют

```go
switch any(v).(type)
```

---

# Когда использовать Generics

✅ Контейнеры

- Stack
- Queue
- Set

---

✅ Коллекции

- Filter
- Map
- Reduce
- Reverse

---

✅ Кэш

```
Cache[K comparable, V any]
```

---

✅ Repository

```
Repository[T]
```

---

✅ Общие алгоритмы

- Max
- Min
- Sort
- Clone

---

# Когда НЕ использовать

Не стоит делать generic только ради generic.

Плохо:

```go
func Print[T any](v T)
```

если функция используется один раз.

Generics полезны, когда действительно есть повторяющийся алгоритм для разных типов.

---

# Что любят спрашивать на собеседовании

### Что такое Generics?

Позволяют писать типобезопасный код, работающий с разными типами.

---

### Что такое `T any`?

Параметр типа с ограничением `any`.

---

### Что такое constraint?

Ограничение допустимых типов.

Например

```go
T comparable
```

или

```go
T Ordered
```

---

### Что такое comparable?

Встроенное ограничение для типов, поддерживающих `==` и `!=`.

Используется, например, для ключей `map`.

---

### Что означает `~int`?

Любой тип, базовым типом которого является `int`.

---

### Есть ли Generics во время выполнения?

Нет.

Компилятор генерирует специализированный код во время компиляции. В рантайме информации о параметрах типа не остаётся в том виде, как, например, в Java с рефлексией.