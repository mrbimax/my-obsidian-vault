## Comparable типы в Go

✅ **Поддерживают `==` / `!=`:**

- `bool`
- `int`, `int8`, `int16`, `int32`, `int64`
- `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
- `float32`, `float64`
- `complex64`, `complex128`
- `string`
- `pointer` (`*T`)
- `channel` (`chan T`)
- `array` (`[N]T`) — если `T` comparable
- `struct` — если **все** поля comparable
- `interface` — если динамическое значение comparable

---
## Non-comparable типы в Go

❌ **Не поддерживают `==` / `!=`:**

- `slice` (`[]T`)
- `map` (`map[K]V`)
- `function` (`func(...)`)

---
## Примеры

Работает:

``` go
10 == 10

"a" == "a"

[2]int{1,2} == [2]int{1,2}

struct {ID int}{} == struct {ID int}{}
```

Не работает:

``` go
[]int{1,2} == []int{1,2} // ❌

map[string]int{} == map[string]int{} // ❌

func(){} == func(){} // ❌
```

---

## Для Generics

``` go
func Equal[T comparable](a, b T) bool {
	return a == b
}
```

Разрешит:

``` go
Equal(10, 20)                 // OK
Equal("go", "go")             // OK
Equal([3]int{}, [3]int{})     // OK
Equal(&x, &y)                 // OK
```

Запретит:

``` go
Equal([]int{}, []int{})       // ❌
Equal(map[string]int{}, nil)  // ❌
```

Главное правило:

> `array` и `struct` comparable только если **все их элементы/поля comparable**.

---

Пример: 

``` run-go
package main

import "fmt"

func Equal[T comparable](a, b T) bool {
	return a == b
}

func main() {	
	fmt.Println(Equal(10, 10)) // int 
	
	fmt.Println(Equal("go", "go")) // string
	
	fmt.Println(Equal(true, true)) 	// bool
	
	fmt.Println(Equal(
			[3]int{1, 2, 3}, 
			[3]int{1, 2, 3}, 
		),
	) // array
	
	fmt.Println(Equal( 
			struct{ A int }{A: 1}, 
			struct{ A int }{A: 1}, 
		),
	) // struct{}
	
	ch := make(chan struct{})
	ch2 := ch
	fmt.Println(Equal(ch, ch2)) // channel
	
	a := 10 
	ptr1 := &a
	ptr2 := &a 
	fmt.Println(Equal(ptr1, ptr2)) // pointer
}
```
