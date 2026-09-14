## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Temporary files

## Назначение

Temporary files — временные файлы, создаваемые во время тестов.

Используются для тестирования:

- чтения файлов;
    
- записи файлов;
    
- конфигурации;
    
- импорта/экспорта данных;
    
- работы с файловой системой.
    

Главная идея:

> Тест не должен создавать мусор в проекте и зависеть от существующих файлов.

---

## Основной инструмент

Создать временную директорию:

```go
dir := t.TempDir()
```

Создать файл внутри неё:

```go
file := filepath.Join(
	dir,
	"config.json",
)
```

---

## Пример

```go
func TestReadConfig(t *testing.T) {

	dir := t.TempDir()

	file := filepath.Join(
		dir,
		"config.json",
	)

	err := os.WriteFile(
		file,
		[]byte(`{"port":8080}`),
		0644,
	)

	if err != nil {
		t.Fatal(err)
	}

}
```

---

## Что делает TempDir

```go
dir := t.TempDir()
```

Создаёт примерно:

```text
/tmp/TestReadConfig123456/
```

После завершения теста:

```text
директория удаляется автоматически
```

---

## Создание временного файла

Через стандартную библиотеку:

```go
file, err := os.CreateTemp(
	"",
	"config-*.json",
)
```

Пример:

```go
func TestFile(t *testing.T) {

	file, err := os.CreateTemp(
		"",
		"test-*.txt",
	)

	if err != nil {
		t.Fatal(err)
	}

	t.Cleanup(func() {

		os.Remove(file.Name())

	})

}
```

---

## Почему обычно лучше TempDir

Вместо:

```go
os.CreateTemp(...)
```

чаще пишут:

```go
dir := t.TempDir()

file := filepath.Join(
	dir,
	"test.txt",
)
```

Потому что:

- проще;
    
- вся структура теста в одном месте;
    
- автоматическая очистка.
    

---

## Тестирование чтения файла

Функция:

```go
func Read(path string) ([]byte, error) {

	return os.ReadFile(path)

}
```

Тест:

```go
func TestRead(t *testing.T) {

	dir := t.TempDir()

	file := filepath.Join(
		dir,
		"data.txt",
	)

	os.WriteFile(
		file,
		[]byte("hello"),
		0644,
	)

	data, err := Read(file)

	if err != nil {
		t.Fatal(err)
	}

	if string(data) != "hello" {
		t.Fatal("wrong data")
	}

}
```

---

## Тестирование записи файла

Функция:

```go
func Save(path string) error {

	return os.WriteFile(
		path,
		[]byte("data"),
		0644,
	)

}
```

Тест:

```go
func TestSave(t *testing.T) {

	dir := t.TempDir()

	file := filepath.Join(
		dir,
		"out.txt",
	)

	err := Save(file)

	if err != nil {
		t.Fatal(err)
	}

}
```

---

## TempDir и Chdir

Частый паттерн:

```go
func TestCLI(t *testing.T) {

	dir := t.TempDir()

	t.Chdir(dir)

}
```

Теперь все относительные пути работают внутри временной директории.

---

## Что НЕ делать

Плохо:

```go
func TestConfig(t *testing.T) {

	os.WriteFile(
		"config.json",
		[]byte("{}"),
		0644,
	)

}
```

Проблемы:

- загрязняет проект;
    
- может сломать реальные файлы;
    
- конфликтует с другими тестами.
    

---

## Для собеседования

> Для работы с временными файлами в тестах обычно используют `t.TempDir()`. Он создаёт уникальную временную директорию для каждого теста и автоматически удаляет её после завершения через `t.Cleanup()`. Это позволяет безопасно тестировать чтение и запись файлов без загрязнения рабочей директории проекта.