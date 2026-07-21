## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---

# Golang. Пакет testing. Chdir

## Назначение

`t.Chdir()` временно меняет **текущую рабочую директорию процесса** на указанную.

После завершения теста директория автоматически восстанавливается.

Используется для тестирования кода, который зависит от:

- относительных путей;
    
- `os.Getwd()`;
    
- чтения файлов без абсолютного пути;
    
- CLI-команд.
    

---

## Синтаксис

```go
t.Chdir(dir string)
```

Пример:

```go
func TestReadConfig(t *testing.T) {

	dir := t.TempDir()

	t.Chdir(dir)

	file, err := os.Create("config.json")

	if err != nil {
		t.Fatal(err)
	}

	file.Close()

}
```

После:

```text
до теста:

/project


во время теста:

/tmp/TestReadConfig123/


после теста:

/project
```

---

## Пример с относительным путём

Есть функция:

```go
func LoadConfig() ([]byte, error) {

	return os.ReadFile(
		"config.json",
	)

}
```

Тест:

```go
func TestLoadConfig(t *testing.T) {

	dir := t.TempDir()

	t.Chdir(dir)

	err := os.WriteFile(
		"config.json",
		[]byte("{}"),
		0644,
	)

	if err != nil {
		t.Fatal(err)
	}

	_, err = LoadConfig()

	if err != nil {
		t.Fatal(err)
	}

}
```

---

## Как работает внутри

До:

```text
cwd = /project
```

Вызов:

```go
t.Chdir("/tmp/test")
```

Меняет:

```text
cwd = /tmp/test
```

Также регистрируется cleanup:

```go
t.Cleanup(func() {
	os.Chdir(oldDir)
})
```

После теста:

```text
cwd = /project
```

---

## Chdir + TempDir

Частый паттерн:

```go
func TestCLI(t *testing.T) {

	dir := t.TempDir()

	t.Chdir(dir)

	// создаём файлы
	// запускаем программу

}
```

Плюсы:

- изоляция теста;
    
- нет загрязнения проекта;
    
- автоматическое удаление.
    

---

## Ограничение: нельзя использовать с Parallel

Нельзя:

```go
func TestFiles(t *testing.T) {

	t.Parallel()

	t.Chdir("/tmp")

}
```

Причина:

текущая директория — глобальное состояние процесса.

Параллельные тесты:

```text
TestA:
cwd=/tmp/a

TestB:
cwd=/tmp/b
```

будут конфликтовать.

---

## Отличие от `os.Chdir`

Вручную:

```go
old, _ := os.Getwd()

os.Chdir("/tmp")

defer os.Chdir(old)
```

Проблемы:

- нужно самому восстанавливать состояние;
    
- легко забыть cleanup.
    

С `t.Chdir()`:

```go
t.Chdir("/tmp")
```

Go делает cleanup автоматически.

---

## Setenv vs Chdir

`Setenv`:

```go
t.Setenv(
	"CONFIG_PATH",
	"/tmp/config",
)
```

Меняет:

```text
переменную окружения
```

---

`Chdir`:

```go
t.Chdir("/tmp")
```

Меняет:

```text
рабочую директорию процесса
```

Оба:

- временные;
    
- автоматически откатываются;
    
- нельзя использовать с `t.Parallel()`.
    

---

## Для собеседования

> `t.Chdir()` временно меняет текущую рабочую директорию тестового процесса и автоматически возвращает её обратно после завершения теста через cleanup. Используется для тестирования CLI и кода, который работает с относительными путями. Как и `t.Setenv()`, не совместим с `t.Parallel()`, потому что меняет глобальное состояние процесса.