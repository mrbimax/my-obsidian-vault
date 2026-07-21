Интерфейс - структура, контракт на определенное поведение.

В интерфейс с определенным набором методов можно поместить только тот объект, который реализует этот интерфейс.

В Go непустой (с определенным набором методов) и пустой интерфейс (any, interfrace{}) - это две разные структуры: iface и eface.

Нам интересен iface для примера.

``` GO
type iface struct {
    tab  *itab // набор методов, который интерфейс определяет
    data unsafe.Pointer // ссылка на объект, который реализует интерфейс
}
```
# Частный случай:

Если присвоить интерфейсу Reader (определяет сигнатуру метода Read) объект типа Buffer (реализует метод Read), но при этом сам Buffer содержит nil, то интерфейс не будет являться nil: itab содержит набор методов, который реализованы типом Buffer, а data будет содержать ансейв указатель на nil, но сама структура интерфейса не будет являться nil.
# Пример кода:

``` run-go
package main

import "fmt"

type Reader interface {
    Read() string
    SafeRead() string
}

type Buffer struct {
    data string
}

func (b *Buffer) Read() string {
    return b.data // если b == nil то будет panic
}
func (b *Buffer) SafeRead() string {
    if b == nil {
           return "ошибка, b == nil"   
    }
    return b.data
}

func main() {
    var r Reader
    var b *Buffer // nil
    r = b         // интерфейс получает тип *Buffer, data = nil

    if r != nil {
        fmt.Println("интерфейс не nil")
        fmt.Println(r.SafeRead()) // вывод  "ошибка, b == nil"
        fmt.Println(r.Read()) // здесь panic
    }
}
```