# Golang Generics Constraints (ограничения типов)

Constraint (ограничение типа) — это правило, которое говорит компилятору, какие типы разрешены для generic-параметра.

Появились в Go 1.18 вместе с Generics.

---
# Зачем нужны constraints

Без ограничения:

``` run-go
package main

import "fmt"

func Max[T any](a, b T) T {
	if a > b {
		return a
	}
	return b
}

func main(){
	fmt.Println(Max(1,2))
}
```

Код не скомпилируется из-за `T any`.

Почему?
Потому что не любой тип поддерживает:

``` go
>
<
+
*
==
```

Например:

``` go
[]int
map[string]int
func()
```

не имеют оператора `>`.

---
# Базовый синтаксис

``` go
func Name[T Constraint](value T) {
}
```

Пример:

``` go
func Print[T fmt.Stringer](v T) {
	fmt.Println(v.String())
}
```

`T` обязан реализовывать [[Golang. Пакет fmt. Stringer|fmt.Stringer]].

---
# [[Golang. Comparable типы|comparable]]

Встроенный constraint.

Разрешает типы, которые поддерживают:

``` go
==
!=
```

Используется для ключей map.

Пример:

``` run-go
package main

import "fmt"

func Equal[T comparable](a, b T) bool {
	return a == b
}

func main() {
	fmt.Println(Equal(10, 10))
	fmt.Println(Equal("go", "go"))
}
```

Сравнимые и несравнимые типы:

``` go
// Comparable
int
string
bool
array
struct
pointer
channel

// Non-Comparable
slice
map
func
```

Пример: 

``` run-go
package main

import "fmt"

func Equal[T comparable](a, b T) bool {
	return a == b
}

func main() {
	// int 
	fmt.Println(Equal(10, 10)) 
	// string
	fmt.Println(Equal("go", "go")) 
	// bool
	fmt.Println(Equal(true, true)) 
	// array
	fmt.Println(Equal(
			[3]int{1, 2, 3}, 
			[3]int{1, 2, 3}, 
		),
	) 
	// struct{}
	fmt.Println(Equal( 
			struct{ A int }{A: 1}, 
			struct{ A int }{A: 1}, 
		),
	) 
	// channel
	fmt.Println(
		Equal(
			make(chan struct{}), 
			make(chan struct{}), 
		),
	) 
	// pointer
	a := 10 
	b := 10 
	fmt.Println(Equal(&a, &b)) 
}
```

---
# Type union

Можно перечислить разрешённые типы:

``` go
type Number interface {
	int | float64
} // int ИЛИ float64
```

---

# Пример union

``` run-go
package main

import "fmt"

type Number interface {
	int | float64
}

func Sum[T Number](a, b T) T {
	return a + b
}

func main() {
	fmt.Println(Sum(10, 20)) // 30
	fmt.Println(Sum(1.5, 2.5)) // 4
	fmt.Println(Sum(1.5, 2)) // 4
}
```

---
# Символ ~

Самая важная часть constraints.

`~` означает:

> разрешить типы с таким же underlying type (базовым типом)

---

## Без ~

``` go
type MyInt int

type OnlyInt interface {
	int
}
```

Так нельзя:

``` go
func Test[T OnlyInt](v T) {}

func main() {
	var x MyInt = 10

	Test(x) // ошибка
}
```

Почему?

Потому что:

``` go
MyInt != int
```

---
## С ~

``` go
type MyInt int

type IntLike interface {
	~int
}
```

Теперь:

``` run-go
package main

import "fmt"

type MyInt int

type IntLike interface {
	~int
}

func Double[T IntLike](v T) T {
	return v * 2
}

// эквивалентно
func Double2[T ~int](v T) T {
	return v * 2
}

func main() {
	var x MyInt = 10

	fmt.Println(Double(x)) // 20
	fmt.Println(Double2(x)) // 20
}
```

---

# Разница int и ~int

|Constraint|Разрешает|
|---|---|
|`int`|только int|
|`~int`|int + свои типы на основе int|

---

# Комбинация типов

``` go
type Ordered interface {
	~int |
	~int64 |
	~float64 |
	~string
}
```

Означает:

``` go
разрешены:

int
int64
float64
string

и пользовательские типы на их основе
```

---

# Практический пример Ordered

``` go
package main

import "fmt"

type Ordered interface {
	~int |
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
	fmt.Println(Max(2.5, 1.5))
	fmt.Println(Max("cat", "dog"))
}
```

---
# Методы в constraints

Constraint может требовать методы.

Например:

``` go
type Stringer interface {
	String() string
}
```

Использование:

``` go
func Print[T Stringer](v T) {
	fmt.Println(v.String())
}
```

---
# Constraint с методами + типами

Можно смешивать:

``` go
type StringInt interface {
	~int
	String() string
}
```

Теперь тип должен:

1. иметь underlying type int;
2. иметь метод String().

---
# Использование интерфейса как constraint

Обычный интерфейс:

``` go
type Writer interface {
	Write([]byte) error
}
```

может быть constraint:

``` go
func Save[T Writer](w T) {
	w.Write([]byte("data"))
}
```

---
# Нельзя использовать обычный интерфейс с union как значение

Например:

``` go
type Number interface {
	int | float64
}
```

Нельзя:

``` go
var n Number
```

Ошибка.

Почему?
Потому что это не обычный интерфейс значения.

Это constraint только для generic-кода.

---
# Встроенные constraints

## any

Любой тип:

``` go
T any
```

---
## comparable

Типы с:

``` go
==
!=
```

---
# [[Golang. Пакет cmp|Пакет cmp]] (Go 1.21+)

Вместо:
``` go
type Ordered interface {
	~int |
	~float64 |
	~string
}
```
можно:

``` go
import "cmp"

func Max[T cmp.Ordered](a, b T) T {
	if a > b {
		return a
	}
	return b
}
```

---
# Практические случаи использования

## Generic Cache

``` go
type Cache[K comparable, V any] struct {
	data map[K]V
}
```

---
## Generic Repository

``` go
type Repository[T any] struct {
	items []T
}
```

---

## Generic функции коллекций

``` go
Filter[T any]
Map[T,R]
Reduce[T]
Contains[T comparable]
```

---

# Правило использования

Используй constraint когда нужно:

✅ оператор (`+`, `>`, `<`)  
✅ сравнение (`==`)  
✅ конкретные методы  
✅ работа с несколькими похожими типами

Не используй:

❌ просто чтобы "сделать красиво"  
❌ если функция используется один раз  
❌ вместо обычного interface