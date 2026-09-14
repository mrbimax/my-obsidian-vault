## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_
---

# Golang. Пакет testing. HTTP client testing

## Назначение

**HTTP client testing** — тестирование кода, который делает HTTP-запросы к внешним сервисам.

Например:

```text
OrderService
      |
      ↓
PaymentClient
      |
      ↓
Payment API
```

Мы тестируем именно HTTP-клиент:

- правильный URL;
    
- HTTP method;
    
- headers;
    
- query parameters;
    
- request body;
    
- обработку JSON;
    
- обработку status codes;
    
- timeout;
    
- ошибки сети.
    

---

# Главная идея

В unit-тесте мы **не должны ходить в настоящий внешний API**.

Плохо:

```text
Test
 |
 ↓
HTTP Client
 |
 ↓
Real API
 |
 ↓
Internet
```

Проблемы:

- тест зависит от сети;
    
- API может быть недоступен;
    
- тесты медленные;
    
- данные могут измениться.
    

---

Правильно:

```text
Test

HTTP Client
     |
     ↓
httptest.Server
     |
     ↓
Mock API
```

`httptest.Server` создаёт настоящий HTTP сервер локально.

---

# Пример: HTTP client

Допустим, есть клиент пользователей.

Структура проекта:

```
client/
 ├── user_client.go
 └── user_client_test.go
```

---

# Production код

## user_client.go

```go
package client

import (
	"encoding/json"
	"errors"
	"fmt"
	"net/http"
)

type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}


type UserClient struct {
	BaseURL string
	Client  *http.Client
}


func (c UserClient) GetUser(id int) (*User, error) {

	req, err := http.NewRequest(
		http.MethodGet,
		fmt.Sprintf(
			"%s/users/%d",
			c.BaseURL,
			id,
		),
		nil,
	)

	if err != nil {
		return nil, err
	}


	resp, err := c.Client.Do(req)

	if err != nil {
		return nil, err
	}

	defer resp.Body.Close()


	if resp.StatusCode != http.StatusOK {
		return nil, errors.New(
			"unexpected status",
		)
	}


	var user User

	err = json.NewDecoder(
		resp.Body,
	).Decode(&user)

	if err != nil {
		return nil, err
	}


	return &user, nil
}
```

---

# Тест HTTP клиента

## user_client_test.go

```go
package client

import (
	"net/http"
	"net/http/httptest"
	"testing"
)


func TestUserClient_GetUser(t *testing.T) {

	server := httptest.NewServer(
		http.HandlerFunc(
			func(
				w http.ResponseWriter,
				r *http.Request,
			) {


				// Проверяем HTTP метод

				if r.Method != http.MethodGet {

					t.Errorf(
						"expected GET, got %s",
						r.Method,
					)

				}


				// Проверяем URL

				if r.URL.Path != "/users/1" {

					t.Errorf(
						"wrong path: %s",
						r.URL.Path,
					)

				}


				// Отдаём JSON ответ

				w.Header().Set(
					"Content-Type",
					"application/json",
				)


				w.WriteHeader(
					http.StatusOK,
				)


				w.Write([]byte(`
				{
					"id":1,
					"name":"Max"
				}
				`))

			},
		),
	)


	defer server.Close()


	client := UserClient{
		BaseURL: server.URL,
		Client:  server.Client(),
	}


	user, err := client.GetUser(1)


	if err != nil {

		t.Fatal(err)

	}


	if user.ID != 1 {

		t.Errorf(
			"expected id 1, got %d",
			user.ID,
		)

	}


	if user.Name != "Max" {

		t.Errorf(
			"expected Max, got %s",
			user.Name,
		)

	}

}
```

---

# Что реально проверяет этот тест

Полный путь:

```text
TestUserClient_GetUser

        |
        ↓

UserClient.GetUser(1)

        |
        ↓

HTTP GET /users/1

        |
        ↓

httptest.Server

        |
        ↓

JSON response

        |
        ↓

decode User struct
```

Проверяются сразу несколько вещей:

✅ запрос сформирован правильно  
✅ endpoint правильный  
✅ HTTP метод правильный  
✅ JSON корректно распарсился  
✅ клиент обработал ответ

---

# Проверка headers

Допустим, API требует токен.

Меняем клиент:

```go
req.Header.Set(
	"Authorization",
	"Bearer token",
)
```

Тест:

```go
if r.Header.Get(
	"Authorization",
) != "Bearer token" {

	t.Error(
		"missing auth header",
	)

}
```

Проверяем:

```text
Client
 |
 ↓
Authorization header
 |
 ↓
Server
```

---

# Проверка ошибки сервера

Например API вернул:

```
500 Internal Server Error
```

Тест:

```go
func TestUserClient_ServerError(t *testing.T) {


	server := httptest.NewServer(
		http.HandlerFunc(
			func(
				w http.ResponseWriter,
				r *http.Request,
			){

				w.WriteHeader(
					http.StatusInternalServerError,
				)

			},
		),
	)


	defer server.Close()


	client := UserClient{
		BaseURL: server.URL,
		Client: server.Client(),
	}


	_, err := client.GetUser(1)


	if err == nil {

		t.Fatal(
			"expected error",
		)

	}

}
```

Проверяем:

```text
API вернул ошибку
        |
        ↓
Client правильно обработал её
```

---

# Проверка некорректного JSON

Сервер:

```go
w.Write([]byte(`
{
	"wrong json"
}
`))
```

Тест:

```go
_, err := client.GetUser(1)

if err == nil {
	t.Fatal(
		"expected json error",
	)
}
```

---

# Проверка timeout

Клиент:

```go
http.Client{
	Timeout: time.Second,
}
```

Сервер:

```go
func(
	w http.ResponseWriter,
	r *http.Request,
){

	time.Sleep(
		5*time.Second,
	)

}
```

Проверяем:

```text
Server завис
       |
       ↓
HTTP client timeout
       |
       ↓
error
```

---

# HTTP client testing через Mock

Другой вариант:

Создать интерфейс:

```go
type HTTPDoer interface {

	Do(
		req *http.Request,
	) (*http.Response,error)

}
```

И заменить:

```text
Real HTTP client

        ↓

Mock HTTP client
```

---

Но:

## Mock

Проверяет:

```
Do() вызван?
С каким request?
```

---

## httptest.Server

Проверяет:

```
Реальный HTTP слой:

method
URL
headers
body
status
JSON
```

Для HTTP клиентов чаще используют именно `httptest`.

---

# Когда использовать HTTP client testing

Подходит для:

- REST клиентов;
    
- gRPC gateway через HTTP;
    
- OAuth клиентов;
    
- платежных API;
    
- внешних интеграций;
    
- webhook клиентов.
    

---

# Когда не использовать

Не надо тестировать через HTTP внутреннюю логику:

Плохо:

```
Handler
   |
   ↓
Service
   |
   ↓
Repository
```

Для этого:

- unit tests;
    
- mocks;
    
- fakes.
    

HTTP-тест нужен только для HTTP слоя.

---

# Для собеседования

> HTTP client testing — это проверка клиентов, которые делают HTTP-запросы во внешние сервисы. В Go обычно используют `httptest.Server`, который поднимает локальный HTTP сервер и позволяет проверить реальное поведение клиента: URL, method, headers, body, обработку JSON, status codes и сетевые ошибки. Такой подход позволяет тестировать HTTP слой без зависимости от настоящих внешних API.