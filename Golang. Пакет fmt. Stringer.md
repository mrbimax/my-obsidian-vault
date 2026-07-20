`fmt.Stringer` — это интерфейс для типов, которые умеют преобразовываться в строку.

``` go
type Stringer interface {
	String() string
}
```

Если у структуры есть метод:

``` go
func (u User) String() string {
	return "user"
}
```

то `fmt.Println(u)` автоматически вызовет `String()`.

Пример:

``` run-go
package main

import "fmt"

type User struct {
	Name string
}

func (u User) String() string {
	return "User: " + u.Name
}

func main() {
	fmt.Println(User{Name: "Alex"})
}
```

Вывод:

``` go
User: Alex
```

Используется для кастомного форматирования вывода.