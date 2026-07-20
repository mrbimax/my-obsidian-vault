## _[[Golang. Пакет testing. Обзор фичей|Список фичей из пакета testing]]_

---
# Golang. Пакет testing. Race detector

## Назначение

Race detector — инструмент для поиска **data race** (гонок данных) в Go-программах.

Data race возникает, когда:

- несколько goroutine обращаются к одной переменной;    
- хотя бы одна goroutine изменяет данные;    
- нет синхронизации (`mutex`, `channel`, `atomic`).    

---

## Запуск

С флагом `-race`:

```bash
go test -race
```

Также работает:

```bash
go run -race main.go
```

```bash
go build -race
```

---

## Пример гонки

```go
package main

import (
	"testing"
)

func TestRace(t *testing.T) {
	counter := 0
	for i := 0; i < 10; i++ {
		go func() {
			counter++
		}()
	}
}
```

Проблема:

```text
goroutine 1 → counter++
goroutine 2 → counter++

одновременная запись в память
```

---

## Исправление через Mutex

```go
package main

import (
	"sync"
	"testing"
)

func TestRace(t *testing.T) {
	var mu sync.Mutex
	counter := 0
	for i := 0; i < 10; i++ {
		go func() {
			mu.Lock()
			counter++
			mu.Unlock()
		}()
	}
}
```

---

## Пример вывода Race Detector

При обнаружении:

```text
WARNING: DATA RACE

Read at 0x00...
Write at 0x00...

Goroutine 7:
...

Goroutine 8:
...
```

Показывает:

- место чтения;    
- место записи;    
- goroutine, где произошла проблема.    

---
## В тестах

Часто используется вместе с:

```bash
go test -race ./...
```

Проверяет весь проект:

```text
package1
 ├── tests
 └── race detector

package2
 ├── tests
 └── race detector
```

---
## Что умеет находить

Находит:

- гонки переменных;    
- проблемы с `map`;    
- неправильное использование shared state;    
- ошибки синхронизации.    

---

## Что НЕ находит

Не обнаруживает:

- логические ошибки;    
- deadlock;    
- неправильные алгоритмы;    
- race, который не был воспроизведён во время запуска.    

Race detector анализирует только реально выполненные пути.

---

## Ограничения

Минусы:

- медленнее выполнение тестов;    
- больше потребление памяти;    
- нельзя использовать в production workload.    

Обычно запускают:

- в CI;    
- перед релизом;    
- при изменениях конкурентного кода.    

---
## Для собеседования

> Race detector — встроенный инструмент Go для поиска data race. Запускается через `go test -race`. Он анализирует выполнение программы и показывает конкурентный доступ к общей памяти без синхронизации. Не заменяет тестирование, а дополняет его, поэтому проверяет только реально выполненные сценарии.