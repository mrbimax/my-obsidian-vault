## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz corpus storage

## Назначение

Fuzz corpus storage — механизм хранения входных данных, которые используются fuzzing.

Corpus содержит:

- начальные seed-значения;
    
- найденные fuzzing engine интересные случаи;
    
- минимальные примеры, которые приводят к ошибкам.
    

---

## Где хранится corpus

Стандартное расположение:

```text
testdata/
└── fuzz/
    └── FuzzFunctionName/
        ├── input1
        ├── input2
        └── input3
```

Пример:

```text
parser/
├── parser.go
├── parser_test.go
└── testdata/
    └── fuzz/
        └── FuzzParse/
            ├── valid.json
            └── broken.json
```

Имя директории должно совпадать с именем fuzz-функции:

```go
func FuzzParse(f *testing.F)
```

↓

```text
testdata/fuzz/FuzzParse/
```

---

# Добавление corpus через f.Add()

Самый простой вариант:

```go
func FuzzParse(f *testing.F) {

	f.Add([]byte(`{"id":1}`))
	f.Add([]byte(`{}`))

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Эти значения становятся seed corpus.

---

# Corpus из файлов

Пример:

Файл:

```text
testdata/fuzz/FuzzParse/user.json
```

Содержимое:

```json
{
	"id": 1
}
```

Fuzz:

```go
func FuzzParse(f *testing.F) {

	f.Fuzz(func(
		t *testing.T,
		data []byte,
	) {

		Parse(data)

	})

}
```

Go автоматически загрузит файл как входные данные.

---

# Что сохраняет fuzzing engine

Во время работы fuzzing создаёт новые значения.

Например:

```text
seed:

{"id":1}
```

Мутации:

```text
{"id":2}
{"id":999}
{"id":}
```

Если новый input:

- открыл новый путь выполнения;
    
- вызвал ошибку;
    
- вызвал panic;
    

он сохраняется.

---

# Жизненный цикл corpus

```text
f.Add()
   |
   ↓
seed corpus

   |
   ↓

mutation

   |
   ↓

новый интересный input

   |
   ↓

testdata/fuzz/FuzzXxx/

   |
   ↓

следующие запуски
```

---

# Corpus как регрессионные тесты

Допустим fuzz нашёл:

```text
testdata/fuzz/FuzzParser/abc123
```

Содержимое:

```text
invalid input
```

После исправления бага:

```bash
go test
```

будет снова запускать этот input.

Если ошибка вернётся — тест упадёт.

То есть corpus становится:

```text
fuzz найденный баг
        ↓
регрессионный тест
```

---

# Seed corpus vs Generated corpus

## Seed corpus

Создаёт разработчик:

```go
f.Add("hello")
```

или:

```text
testdata/fuzz/FuzzXxx/
```

Назначение:

- важные примеры;
    
- граничные случаи;
    
- валидные структуры.
    

---

## Generated corpus

Создаёт сам fuzzing:

```text
testdata/fuzz/FuzzXxx/
    ├── abc123
    └── def456
```

Назначение:

- сохранить новые пути;
    
- повторно использовать найденные случаи.
    

---

# Почему corpus важен

Без corpus:

```text
random data
     ↓
много бесполезных входов
```

С corpus:

```text
хорошие примеры
     ↓
мутации
     ↓
глубокое исследование кода
```

Например:

Seed:

```json
{
	"user":"admin"
}
```

Fuzzer легче найдёт:

```json
{
	"user":""
}
```

чем начнёт с:

```text
FF 00 A1 23
```

---

# Запуск только seed corpus

Обычный запуск:

```bash
go test
```

проверяет:

- `f.Add()`;
    
- файлы из `testdata/fuzz`.
    

---

# Запуск fuzzing

```bash
go test -fuzz=FuzzParse
```

Начинается:

- генерация;
    
- мутация;
    
- расширение corpus.
    

---

## Для собеседования

> Fuzz corpus storage — это механизм хранения входных данных fuzz-тестов. Corpus включает seed-значения, заданные через `f.Add()` или файлы в `testdata/fuzz/FuzzXxx`, а также автоматически найденные fuzzing engine интересные случаи. Сохранённые значения используются при следующих запусках и превращаются в регрессионные тесты.