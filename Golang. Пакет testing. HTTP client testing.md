## _[[Golang. Пакет testing. HTTP / API тестирование|Список фичей из раздела]]_

---

# Golang. Пакет testing. HTTP client testing

## Назначение

**HTTP client testing** — тестирование кода, который отправляет HTTP-запросы во внешние сервисы.

Например:

```text
UserService
      |
      ↓
HTTP Client
      |
      ↓
External API
```

Проверяем:

- правильно ли формируется запрос;
    
- правильный ли URL;
    
- HTTP method;
    
- headers;
    
- body;
    
- обработку ответов;
    
- обработку ошибок сети.
    

---

# Пример: HTTP клиент

Допустим, есть клиент для получения пользователя.

```go
type UserClient struct {
	baseURL string
	client  *http.Client
}

func (c UserClient) GetUser(id int) (*User, error) {

	resp, err := c.client.Get(
		c.baseURL + "/users/" + strconv.Itoa(id),
	)

	if err != nil {
		return nil, err
	}

	defer resp.Body.Close()


	if resp.StatusCode != http.StatusOK {
		return nil, errors.New("bad status")
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

# Проблема обычного тестирования

Плохой вариант:

```go
client := UserClient{
	baseURL: "https://api.example.com",
}
```

Тест реально идёт:

```text
Test
 |
 ↓
HTTP Client
 |
 ↓
Internet
 |
 ↓
External API
```

Проблемы:

- медленно;
    
- зависит от сети;
    
- API может быть недоступно;
    
- сложно проверить ошибки.
    

---

# Решение: Test double для HTTP

Вместо настоящего API используем тестовый сервер.

```text
Test

UserClient
    |
    ↓
httptest.Server
```

---

# httptest.Server

Создаём локальный HTTP сервер:

```go
server := httptest.NewServer(
	http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {

			w.Header().Set(
				"Content-Type",
				"application/json",
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
```

Теперь есть настоящий HTTP endpoint:

```
http://127.0.0.1:xxxxx
```

---

# Тест HTTP клиента

```go
func TestUserClient_GetUser(t *testing.T) {

	server := httptest.NewServer(
		http.HandlerFunc(
			func(
				w http.ResponseWriter,
				r *http.Request,
			) {

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
		baseURL: server.URL,
		client:  server.Client(),
	}


	user, err := client.GetUser(1)


	if err != nil {
		t.Fatal(err)
	}


	if user.Name != "Max" {
		t.Fatal("wrong name")
	}

}
```

---

# Что проверили

Наш тест проверяет:

```text
UserClient
      |
      ↓
HTTP request
      |
      ↓
Test server
      |
      ↓
JSON response
```

Проверяем:

- клиент умеет делать HTTP запрос;
    
- умеет читать JSON;
    
- правильно обрабатывает ответ.
    

---

# Проверка HTTP метода

Например, клиент должен использовать `POST`.

Сервер:

```go
if r.Method != http.MethodPost {

	t.Fatal(
		"wrong method",
	)

}
```

---

# Проверка URL

```go
if r.URL.Path != "/users/1" {

	t.Fatal(
		"wrong path",
	)

}
```

---

# Проверка Headers

Например:

клиент должен отправлять токен:

```go
token := r.Header.Get(
	"Authorization",
)

if token != "Bearer test" {

	t.Fatal(
		"missing token",
	)

}
```

---

# Проверка Request Body

Например:

```go
var body CreateUserRequest

json.NewDecoder(
	r.Body,
).Decode(&body)


if body.Name != "Max" {

	t.Fatal(
		"wrong name",
	)

}
```

---

# Проверка ошибок сервера

Например:

API вернул:

```text
500 Internal Server Error
```

Сервер:

```go
w.WriteHeader(
	http.StatusInternalServerError,
)
```

Тест:

```go
_, err := client.GetUser(1)

if err == nil {

	t.Fatal(
		"expected error",
	)

}
```

---

# Проверка timeout

Сервер завис:

```go
server := httptest.NewServer(
	http.HandlerFunc(
		func(
			w http.ResponseWriter,
			r *http.Request,
		) {

			time.Sleep(
				time.Second * 5,
			)

		},
	),
)
```

Клиент:

```go
http.Client{
	Timeout: time.Second,
}
```

Тест:

```go
_, err := client.GetUser(1)

if err == nil {

	t.Fatal(
		"expected timeout",
	)

}
```

---

# HTTP client testing vs Mock

Можно сделать mock:

```go
type HTTPClient interface {
	Do(req *http.Request) (*http.Response,error)
}
```

и подставить fake.

Но для HTTP клиентов часто лучше:

```text
httptest.Server
```

потому что проверяется настоящий HTTP слой:

- сериализация;
    
- headers;
    
- URL;
    
- status codes.
    

---

# Когда использовать

Хорошо подходит для:

- REST клиентов;
    
- API Gateway клиентов;
    
- OAuth клиентов;
    
- внешних интеграций;
    
- webhook клиентов.
    

---

# Когда не использовать

Не надо проверять через HTTP:

```text
Service
    |
    ↓
Repository
```

Тут лучше:

- mock;
    
- stub;
    
- fake.
    

HTTP-тест нужен именно для HTTP слоя.

---

# Для собеседования

> HTTP client testing — это тестирование кода, который делает HTTP-запросы во внешние сервисы. Обычно используют `httptest.Server`, чтобы поднять локальный HTTP сервер и проверить реальное поведение клиента: формирование URL, HTTP method, headers, body, обработку JSON, статус-кодов и сетевых ошибок. Это позволяет тестировать HTTP слой без зависимости от реального внешнего API.