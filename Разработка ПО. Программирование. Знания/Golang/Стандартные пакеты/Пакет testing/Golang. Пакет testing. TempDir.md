## *[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]* 
---
## Назначение

`t.TempDir()` — helper из пакета `testing` для создания временной директории внутри теста.

Используется для:
- тестирования работы с файлами;
- хранения временных данных;
- изоляции тестов друг от друга.

После завершения теста Go автоматически удаляет директорию.

---
## Синтаксис

```go
dir := t.TempDir()
```

Возвращает:

```go
string // путь к созданной директории
```

---
## Пример

```go
package main

import (
	"os"
	"path/filepath"
	"testing"
)

func TestFile(t *testing.T) {

	dir := t.TempDir()

	file := filepath.Join(dir, "test.txt")

	err := os.WriteFile(
		file,
		[]byte("hello"),
		0644,
	)

	if err != nil {
		t.Fatal(err)
	}

	data, err := os.ReadFile(file)

	if err != nil {
		t.Fatal(err)
	}

	if string(data) != "hello" {
		t.Fatal("wrong content")
	}
}
```

---
## Где создаётся директория

Используется системный временный каталог:

Linux / macOS:
```text
/tmp/
```

Windows:
```text
C:\Users\<user>\AppData\Local\Temp\
```

Пример пути:
```text
/tmp/TestFile123456789/
```

Посмотреть путь:

```go
package main

import "testing"

func TestTempDir(t *testing.T) {

	dir := t.TempDir()

	t.Log(dir)
}
```

Запуск:

```bash
go test -v
```

---
## Автоматическая очистка

Вручную удалять не нужно:

```go
dir := t.TempDir()

// работа с файлами

// после завершения теста:
// os.RemoveAll(dir) выполнится автоматически
```

Внутри `testing` это реализовано через `t.Cleanup()`.

---
## Использование с subtests

Родительский тест может создать общую директорию:

```go
package main

import "testing"

func TestFiles(t *testing.T) {

	dir := t.TempDir()

	t.Run("create", func(t *testing.T) {
		// использует dir
	})

	t.Run("read", func(t *testing.T) {
		// использует dir
	})
}
```

---
## Отличие от `os.MkdirTemp`

Ручной вариант:

```go
package main

import (
	"os"
	"testing"
)

func TestManualTemp(t *testing.T) {

	dir, err := os.MkdirTemp("", "test")

	if err != nil {
		t.Fatal(err)
	}

	defer os.RemoveAll(dir)
}
```

С `testing`:

```go
dir := t.TempDir()
```

Преимущества:

- меньше кода;    
- автоматический cleanup;    
- интеграция с тестовым lifecycle.    

---
## Для собеседования

> `t.TempDir()` создаёт уникальную временную директорию для теста и автоматически удаляет её после завершения. Используется для безопасного тестирования файловой системы без ручного управления cleanup.