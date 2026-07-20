# КРАТКО: `context.WithCancelCause` позволяет указать причину отмены с помощью`cancel("some cause")`

---

`context.WithCancelCause` появился в **Go 1.20** и решает одну из старых проблем `context.Context`: раньше можно было узнать, **что контекст отменён**, но нельзя было понять **почему именно**.

---
## До Go 1.20

Было так:

``` go
ctx, cancel := context.WithCancel(context.Background())
cancel()
fmt.Println(ctx.Err()) // "context.Canceled"
```

Вывод:

``` go
context canceled
```

Причина всегда одна и та же — `context.Canceled`.

Если запрос отменили из-за ошибки БД, таймаута внешнего API или остановки сервиса — эта информация терялась.

---
## С Go 1.20

``` go
ctx, cancel := context.WithCancelCause(context.Background())
cancel(errors.New("database connection lost"))
fmt.Println(context.Cause(ctx)) // "database connection lost"
```

Вывод:

``` go
database connection lost
```

Теперь можно передать **реальную причину отмены**.

---
## Как это работает

``` go
// Создание:
ctx, cancel := context.WithCancelCause(parent)
```

``` go
// Отмена:
cancel(err)
```

``` go
// Получение причины:
err := context.Cause(ctx)
```

---
## Полный пример

``` run-go
package main

import (
    "context"
    "errors"
    "fmt"
)

func main() {
    ctx, cancel := context.WithCancelCause(context.Background())

    cancel(errors.New("service is shutting down"))

    fmt.Println(ctx.Err())
    fmt.Println(context.Cause(ctx))
}
```

Вывод:

``` go
context canceled
service is shutting down
```

Заметь разницу:

- `ctx.Err()` → **контекст отменён** (`context.Canceled`);
- `context.Cause(ctx)` → **конкретная причина** (`service is shutting down`).

---
## Где это полезно в backend

Представим HTTP-запрос:

``` go
HTTP Handler
      │
      ▼
Service
      │
      ▼
Repository
      │
      ▼
gRPC / PostgreSQL
```

Если сервис решил отменить выполнение:

``` go
cancel(fmt.Errorf("user quota exceeded"))
```

то глубоко внутри можно написать:

``` go
select {
case <-ctx.Done():
    log.Println(context.Cause(ctx))
}
```

Лог:

``` go
user quota exceeded
```

Вместо бесполезного:

``` go
context canceled
```

---
## Практический пример

``` run-go
package main

import (
	"context"
	"errors"
	"fmt"
	"time"
)

func Process(ctx context.Context) error {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("\nProcess: контекст отменён")
			fmt.Println("ctx.Err()        =", ctx.Err())
			fmt.Println("context.Cause() =", context.Cause(ctx))
			return context.Cause(ctx)
		default:
			fmt.Println("\nProcess: работаю...")
			fmt.Println("ctx.Err()        =", ctx.Err())
			fmt.Println("context.Cause() =", context.Cause(ctx))
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancelCause(context.Background())

	go func() {
		time.Sleep(2 * time.Second)

		fmt.Println("\nmain: отменяем контекст")
		cancel(errors.New("database connection lost"))
	}()

	err := Process(ctx)

	fmt.Println()
	fmt.Println("Process вернул:", err)
}
```

Теперь вызывающий код может различать причины:

``` go
err := Process(ctx)

if errors.Is(err, ErrQuotaExceeded) {
    ...
}
```

---
## Когда это особенно полезно

- микросервисы;
- [[gRPC]];
- фоновые worker'ы;
- [[graceful shutdown]];
- сложные пайплайны с несколькими [[goroutine]];
- логирование и трассировка (observability).