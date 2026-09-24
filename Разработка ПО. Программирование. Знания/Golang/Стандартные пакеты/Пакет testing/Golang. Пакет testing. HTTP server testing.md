## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. HTTP server testing

## Назначение

`HTTP server testing` — тестирование HTTP-серверной части приложения.

Используется для проверки:

- HTTP handlers;
    
- роутинга;
    
- middleware;
    
- формирования HTTP ответа.
    

Проверяется:

- HTTP method;
    
- URL;
    
- headers;
    
- status code;
    
- response body;
    
- JSON.
    

Основной пакет:

```go
net/http/httptest
```

---

## Основные инструменты

### `httptest.NewRequest`

Создаёт тестовый HTTP запрос.

```go
req := httptest.NewRequest(
	http.MethodGet,
	"/users/1",
	nil,
)
```

Аналог реального запроса:

```http
GET /users/1
```

---

### `httptest.NewRecorder`

Создаёт объект, который сохраняет ответ handler.

Хранит:

- status code;
    
- headers;
    
- body.
    

```go
recorder := httptest.NewRecorder()
```

---

## HTTP handler

`handler.go`

```go
package main

import (
	"encoding/json"
	"net/http"
)

type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

func GetUserHandler(
	w http.ResponseWriter,
	r *http.Request,
) {

	user := User{
		ID:   1,
		Name: "Max",
	}


	w.Header().Set(
		"Content-Type",
		"application/json",
	)


	w.WriteHeader(
		http.StatusOK,
	)


	json.NewEncoder(w).Encode(user)
}
```

---

## Тест HTTP handler

`handler_test.go`

```go
package main

import (
	"encoding/json"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestGetUserHandler(t *testing.T) {

	req := httptest.NewRequest(
		http.MethodGet,
		"/users/1",
		nil,
	)


	recorder := httptest.NewRecorder()


	GetUserHandler(
		recorder,
		req,
	)


	response := recorder.Result()

	defer response.Body.Close()


	if response.StatusCode != http.StatusOK {

		t.Fatalf(
			"expected status 200, got %d",
			response.StatusCode,
		)

	}


	if response.Header.Get(
		"Content-Type",
	) != "application/json" {

		t.Fatal(
			"wrong content type",
		)

	}


	var user User


	err := json.NewDecoder(
		response.Body,
	).Decode(&user)


	if err != nil {

		t.Fatal(err)

	}


	if user.Name != "Max" {

		t.Fatalf(
			"expected Max, got %s",
			user.Name,
		)

	}

}
```

---

## Проверка HTTP метода

Handler:

```go
func GetUserHandler(
	w http.ResponseWriter,
	r *http.Request,
) {

	if r.Method != http.MethodGet {

		http.Error(
			w,
			"method not allowed",
			http.StatusMethodNotAllowed,
		)

		return
	}

}
```

Тест:

```go
func TestGetUserHandler_Method(t *testing.T) {

	req := httptest.NewRequest(
		http.MethodPost,
		"/users/1",
		nil,
	)


	recorder := httptest.NewRecorder()


	GetUserHandler(
		recorder,
		req,
	)


	if recorder.Code != http.StatusMethodNotAllowed {

		t.Fatalf(
			"expected 405, got %d",
			recorder.Code,
		)

	}

}
```

---

## Проверка JSON body

Ответ handler:

```json
{
	"id":1,
	"name":"Max"
}
```

Проверка:

```go
var user User

json.NewDecoder(
	response.Body,
).Decode(&user)


if user.ID != 1 {

	t.Fatal(
		"wrong id",
	)

}
```

---

## Проверка HTTP headers

Handler:

```go
w.Header().Set(
	"X-Request-ID",
	"123",
)
```

Тест:

```go
if response.Header.Get(
	"X-Request-ID",
) != "123" {

	t.Fatal(
		"missing header",
	)

}
```

---

## Проверка ошибок

Например handler возвращает:

```go
http.Error(
	w,
	"not found",
	http.StatusNotFound,
)
```

Тест:

```go
if recorder.Code != http.StatusNotFound {

	t.Fatalf(
		"expected 404, got %d",
		recorder.Code,
	)

}
```

---

## `httptest.NewServer`

Иногда нужно протестировать полный HTTP путь через роутер.

Пример:

```go
server := httptest.NewServer(
	http.HandlerFunc(
		GetUserHandler,
	),
)

defer server.Close()
```

Теперь доступен настоящий URL:

```text
http://127.0.0.1:xxxxx
```

Можно отправлять реальные HTTP запросы через `http.Client`.

---

## HTTP handler test vs HTTP client test

### HTTP server testing

Тестируем свой сервер:

```text
HTTP Request
      |
      ↓
Handler
      |
      ↓
HTTP Response
```

Проверяем:

- обработку запроса;
    
- формирование ответа.
    

---

### HTTP client testing

Тестируем свой клиент:

```text
HTTP Client
      |
      ↓
External API
```

Проверяем:

- какой запрос отправили;
    
- как обработали ответ.
    

---

## Когда использовать

- REST API;
    
- HTTP handlers;
    
- middleware;
    
- webhook endpoints;
    
- authentication endpoints;
    
- validation.
    

---

## Для собеседования

> `HTTP server testing` — это тестирование HTTP handlers и серверной логики без запуска настоящего сервера. В Go используется пакет `httptest`: `NewRequest` создаёт тестовый HTTP запрос, а `NewRecorder` сохраняет ответ handler. С помощью таких тестов проверяют status codes, headers, JSON и корректность обработки HTTP запросов.