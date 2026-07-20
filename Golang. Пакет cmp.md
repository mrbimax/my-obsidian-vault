# Пакет `cmp` в Go

`cmp` — стандартный пакет Go (с **Go 1.21**) для операций сравнения значений.

Импорт:

``` go
import "cmp"
```

Основная идея:

- `cmp.Ordered` — constraint для сравниваемых типов;
- `cmp.Compare` — сравнение двух значений;
- `cmp.Or` — выбор первого ненулевого значения.

---
## `cmp.Ordered`

Самое частое использование в Generics.

``` go
cmp.Ordered
```

Это constraint, который разрешает типы, поддерживающие операторы:

``` go
<
>
<=
>=
```

Пример:

``` run-go
package main

import (
	"cmp"
	"fmt"
)

func Max[T cmp.Ordered](a, b T) T {
	if a > b {
		return a
	}

	return b
}

func main() {
	fmt.Println(Max(10, 20)) // 20
	fmt.Println(Max(3.14, 2.71)) // 3.14
	fmt.Println(Max("cat", "dog")) // dog
}
```

---
## Что входит в `cmp.Ordered`

Примерно:

``` go
interface {
	~int |
	~int8 |
	~int16 |
	~int32 |
	~int64 |
	~uint |
	~uint8 |
	~uint16 |
	~uint32 |
	~uint64 |
	~uintptr |
	~float32 |
	~float64 |
	~string
}
```

То есть:

✅ `int`  
✅ `float64`  
✅ `string`  
✅ свои типы через `~`

Например:

``` go
type Price float64

Max(Price(10.5), Price(20.5))
```

---
# `cmp.Compare`

Функция сравнения:

``` go
func Compare[T Ordered](x, y T) int
```

Возвращает:

```
-1  если x < y
 0  если x == y
 1  если x > y
```

Пример:

``` run-go
package main

import (
	"cmp"
	"fmt"
)

func main() {
	fmt.Println(cmp.Compare(10, 20)) // -1
	fmt.Println(cmp.Compare(20, 10)) // 1
	fmt.Println(cmp.Compare(10, 10)) // 0
	fmt.Println(cmp.Compare("a", "b")) // -1
}
```

---
## Использование для сортировки

Например сортировка структур:

``` run-go
package main

import (
	"cmp"
	"fmt"
	"slices"
)

type User struct {
	Name string
	Age  int
}

func main() {
	users := []User{
		{Name: "Bob", Age: 30},
		{Name: "Alex", Age: 20},
		{Name: "Tom", Age: 25},
	}

	slices.SortFunc(users, func(a, b User) int {
		return cmp.Compare(a.Age, b.Age)
	})

	fmt.Println(users) // [{Alex 20} {Tom 25} {Bob 30}]
}
```

---
# `cmp.Or`

Функция:

``` go
func Or[T comparable](vals ...T) T
```

Возвращает первое значение, которое не равно zero value.

---
Пример:

``` run-go
package main

import (
	"cmp"
	"fmt"
)

func main() {
	fmt.Println(
		cmp.Or("", "default"),
	)

	fmt.Println(
		cmp.Or(0, 10, 20),
	)

	fmt.Println(
		cmp.Or("hello", "world"),
	)
}
```

Вывод:

``` go
default
10
hello
```

---
## Где полезно

Например выбор значения из конфигурации:

``` go
port := cmp.Or(
	envPort,
	configPort,
	8080,
)
```

Логика:

``` go
envPort != zero?
    ↓
берём его

иначе configPort

иначе 8080
```

---
# Отличие `cmp` от `reflect.DeepEqual`

До появления `cmp` часто использовали:

``` go
reflect.DeepEqual(a, b)
```

Но это другое.

`reflect.DeepEqual`:

- работает с любыми типами;
- сравнивает структуры, slice, map;
- использует runtime reflection.

`cmp.Compare`:

- работает только с `Ordered`;
- быстрее;
- проверяется компилятором.

---
# Отличие `cmp.Compare` от `==`

`==`:

``` go
a == b
```

только:

- равны;
- не равны.

`cmp.Compare`:

``` go
cmp.Compare(a, b)
```

даёт порядок:

```
меньше
равно
больше
```

Удобно для сортировки.

---
# Для собеседования

Короткий ответ:

> `cmp` — стандартный пакет Go 1.21 для сравнения значений. Главная часть для Generics — `cmp.Ordered`, готовый constraint вместо ручного `~int | ~float64 | ~string`. Также есть `cmp.Compare`, который возвращает -1/0/1 и используется, например, в `slices.SortFunc`, и `cmp.Or` для выбора первого ненулевого значения.