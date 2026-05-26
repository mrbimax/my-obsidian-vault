Для реализации [[TCP]]-соединения в Go используется стандартный пакет [[Golang/Стандартный пакет net|net]]. Ниже приведен минимальный и самый простой пример, состоящий из двух частей: **Сервера** и **Клиента**.

### 1. [[TCP]]-Сервер (слушает порт и отвечает на сообщения)

Сервер запускается, ожидает входящее соединение, читает из него данные и отправляет ответ обратно.

Go

``` run-go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	// 1. Начинаем слушать TCP-порт 8080
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println("Ошибка запуска сервера:", err)
		os.Exit(1)
	}
	defer listener.Close()
	fmt.Println("Сервер запущен и ожидает подключения на порту 8080...")

	for {
		// 2. Принимаем входящее подключение
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Ошибка подключения:", err)
			continue
		}

		// 3. Обрабатываем подключение в отдельной горутине
		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	// Буфер для чтения данных
	buffer := make([]byte, 1024)
	
	// Читаем данные из соединения
	n, err := conn.Read(buffer)
	if err != nil {
		return
	}

	fmt.Printf("Получено от клиента: %s\n", string(buffer[:n]))

	// Отправляем ответ клиенту
	conn.Write([]byte("Привет от TCP-сервера!"))
}
```

### 2. [[TCP]]-Клиент (подключается, отправляет данные и читает ответ)

Клиент подключается к серверу, отправляет строку, дожидается ответа и завершает работу.

Go

``` run-go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	// 1. Подключаемся к серверу на локальной машине
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		fmt.Println("Не удалось подключиться к серверу:", err)
		os.Exit(1)
	}
	defer conn.Close()

	// 2. Отправляем сообщение серверу
	message := "Привет, сервер!"
	_, err = conn.Write([]byte(message))
	if err != nil {
		fmt.Println("Ошибка отправки:", err)
		return
	}

	// 3. Читаем ответ от сервера
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Println("Ошибка чтения ответа:", err)
		return
	}

	fmt.Printf("Ответ от сервера: %s\n", string(buffer[:n]))
}
```

### Как это протестировать:

1. Сохраните код сервера в файл `server.go`, а код клиента в `client.go`.
2. Запустите сервер в одном терминале: `go run server.go`
3. Запустите клиент в другом терминале: `go run client.go`