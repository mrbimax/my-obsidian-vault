## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Fuzz generation

## Назначение

Fuzz generation — процесс автоматической генерации новых входных данных для fuzz-теста.

Go fuzzing использует **coverage-guided fuzzing**:

- генерирует новые значения;
    
- запускает тест;
    
- анализирует покрытие кода;
    
- сохраняет интересные входы, которые открывают новые пути выполнения.
    

---

## Общая схема

```text
seed corpus
     |
     ↓
генерация нового input
     |
     ↓
запуск fuzz function
     |
     ↓
измерение coverage
     |
     ├── новый путь → сохранить input
     |
     └── нет нового пути → отбросить
```

---

## Пример

```go
func FuzzCheck(f *testing.F) {

	f.Add("hello")

	f.Fuzz(func(t *testing.T, input string) {

		Check(input)

	})

}
```

Seed:

```text
hello
```

Fuzzer создаёт новые значения:

```text
hello
helo
Hello
hello0
hello!
"" 
```

---

## Что именно генерирует Go

Go fuzzing работает с поддерживаемыми типами:

```text
string
[]byte
bool
int
int8
int16
int32
int64
uint
uint8
uint16
uint32
uint64
rune
byte
```

---

## Генерация строк

Например:

```go
f.Add("admin")
```

Возможные мутации:

```text
admin
Admin
admin1
adm
admin\x00
aaaaaaaa
```

---

## Генерация чисел

Seed:

```go
f.Add(100)
```

Fuzzer может попробовать:

```text
0
1
-1
99
101
MaxInt
MinInt
```

Особенно интересны граничные значения.

---

## Генерация `[]byte`

Seed:

```go
f.Add([]byte("hello"))
```

Мутации:

```text
hello
hell
hello000
\x00\x00
FF FF FF
```

Часто используется для:

- парсеров;
    
- декодеров;
    
- протоколов.
    

---

## Coverage-guided подход

Пример:

```go
func Parse(input string) {

	if input == "admin" {
		panic("bug")
	}

}
```

Простой random:

```text
abcd
hello
test
123
```

может никогда не найти:

```text
admin
```

Coverage-guided fuzzing:

```text
random
 |
 ↓
ad
 |
 ↓
adm
 |
 ↓
admin
 |
 ↓
panic
```

Почему?

Потому что каждый шаг открывает новый путь выполнения:

```go
if input[0] == 'a'
```

```go
if input == "ad"
```

```go
if input == "adm"
```

---

## Минимизация найденного бага

Если найден вход:

```text
aaaaaaaaaaaaaaaaadminbbbbbbbb
```

Go пытается уменьшить его:

```text
admin
```

Чтобы получить минимальный воспроизводимый пример.

---

## Где хранятся найденные значения

После нахождения проблемы:

```text
testdata/
└── fuzz/
    └── FuzzCheck/
        └── abc123
```

Этот файл становится новым corpus.

---

## Генерация vs seed

`f.Add()`:

```go
f.Add("hello")
```

Это:

```text
начальное значение
```

Fuzz generation:

```text
hello
 ↓
hello1
 ↓
hello12
 ↓
...
```

Это:

```text
автоматические мутации
```

---

## Что fuzzing НЕ делает

❌ Не понимает структуру:

```json
{
 "age": 20
}
```

Для него это:

```text
[]byte
```

❌ Не знает бизнес-логику:

```text
возраст должен быть > 18
```

Это должен проверять тест.

---

## Для собеседования

> Fuzz generation в Go основана на coverage-guided fuzzing. Fuzzer берёт seed corpus, генерирует новые входные данные с помощью мутаций, запускает fuzz function и анализирует, появились ли новые пути выполнения. Интересные входы сохраняются, а найденные ошибки минимизируются до простого воспроизводимого примера. Go fuzzing не понимает структуру данных, например JSON, а работает на уровне значений и байтов.