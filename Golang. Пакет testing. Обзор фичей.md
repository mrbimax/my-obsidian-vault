# Пакет Go testing — возможности

## Unit-тестирование

- Unit-тесты (`TestXxx`) | [[Golang. Пакет testing. Unit-тесты|Подробнее]]
- Табличные тесты (table-driven tests) | [[Golang. Пакет testing. Table-driven tests|Подробнее]]
- Подтесты (`t.Run`) | [[Golang. Пакет testing. Subtests|Подробнее]]
- Подготовка и очистка тестов (setup / teardown) | [[Golang. Пакет testing. Setup teardown|Подробнее]]
- Общий жизненный цикл тестов (`TestMain`) | [[Golang. Пакет testing. TestMain|Подробнее]]
- Вспомогательные функции тестов (`t.Helper`) | [[Golang. Пакет testing. Helper|Подробнее]]
- Очистка ресурсов после теста (`t.Cleanup`) | [[Golang. Пакет testing. Cleanup|Подробнее]]
- Логирование в тестах (`t.Log`, `t.Logf`) | [[Golang. Пакет testing. Logging|Подробнее]]
- Ошибки и падение тестов (`t.Error`, `t.Fatal`, `t.Fail`) | [[Golang. Пакет testing. Ошибки тестов|Подробнее]]
- Проверка panic | [[Golang. Пакет testing. Проверка panic|Подробнее]]
- Пропуск тестов (`t.Skip`) | [[Golang. Пакет testing. Skip|Подробнее]]
- Условное выполнение тестов (`testing.Short`) | [[Golang. Пакет testing. Short mode|Подробнее]]
- Параллельное выполнение тестов (`t.Parallel`) | [[Golang. Пакет testing. Parallel tests|Подробнее]]
- Поиск гонок данных (`go test -race`) | [[Golang. Пакет testing. Race detector|Подробнее]]

---

## Benchmark

- Benchmark-тесты (`BenchmarkXxx`) | [[Golang. Пакет testing. Benchmark|Подробнее]]
- Количество итераций (`b.N`) | [[Golang. Пакет testing. Benchmark b.N|Подробнее]]
- Управление таймером (`b.ResetTimer`) | [[Golang. Пакет testing. ResetTimer|Подробнее]]
- Остановка таймера (`b.StopTimer`) | [[Golang. Пакет testing. StopTimer|Подробнее]]
- Запуск таймера (`b.StartTimer`) | [[Golang. Пакет testing. StartTimer|Подробнее]]
- Измерение аллокаций (`b.ReportAllocs`) | [[Golang. Пакет testing. ReportAllocs|Подробнее]]
- Добавление пользовательских метрик (`b.ReportMetric`) | [[Golang. Пакет testing. ReportMetric|Подробнее]]
- Подтесты benchmark (`b.Run`) | [[Golang. Пакет testing. Benchmark subtests|Подробнее]]
- Параллельные benchmark (`b.RunParallel`) | [[Golang. Пакет testing. RunParallel|Подробнее]]
- Подготовка данных перед benchmark | [[Golang. Пакет testing. Benchmark setup|Подробнее]]
- Анализ `ns/op` | [[Golang. Пакет testing. Benchmark metrics|Подробнее]]
- Анализ `B/op` | [[Golang. Пакет testing. Benchmark memory|Подробнее]]
- Анализ `allocs/op` | [[Golang. Пакет testing. Benchmark allocations|Подробнее]]

---

## Fuzzing

- Fuzz-тесты (`FuzzXxx`) | [[Golang. Пакет testing. Fuzzing|Подробнее]]
- Начальные входные данные (seed corpus) | [[Golang. Пакет testing. Fuzz seed corpus|Подробнее]]
- Добавление seed данных (`f.Add`) | [[Golang. Пакет testing. Fuzz Add|Подробнее]]
- Запуск fuzz-функции (`f.Fuzz`) | [[Golang. Пакет testing. Fuzz function|Подробнее]]
- Автоматическая генерация входных данных | [[Golang. Пакет testing. Fuzz generation|Подробнее]]
- Мутация входных данных | [[Golang. Пакет testing. Fuzz mutation|Подробнее]]
- Coverage-guided fuzzing | [[Golang. Пакет testing. Coverage guided fuzzing|Подробнее]]
- Поиск panic | [[Golang. Пакет testing. Fuzz panic detection|Подробнее]]
- Поиск неожиданных ошибок | [[Golang. Пакет testing. Fuzz error detection|Подробнее]]
- Сохранение найденных проблемных случаев | [[Golang. Пакет testing. Fuzz corpus storage|Подробнее]]
- Regression-тесты из найденных случаев | [[Golang. Пакет testing. Fuzz regression tests|Подробнее]]

---

## Example-тесты

- Example-функции (`ExampleXxx`) | [[Golang. Пакет testing. Examples|Подробнее]]
- Примеры для пакета (`Example`) | [[Golang. Пакет testing. Package examples|Подробнее]]
- Проверка вывода (`// Output:`) | [[Golang. Пакет testing. Output examples|Подробнее]]
- Использование примеров как документации | [[Golang. Пакет testing. Examples documentation|Подробнее]]

---

## Управление запуском тестов

- Запуск конкретного теста (`-run`) | [[Golang. Пакет testing. Run flag|Подробнее]]
- Запуск benchmark (`-bench`) | [[Golang. Пакет testing. Bench flag|Подробнее]]
- Запуск fuzz (`-fuzz`) | [[Golang. Пакет testing. Fuzz flag|Подробнее]]
- Ограничение времени выполнения (`-timeout`) | [[Golang. Пакет testing. Timeout|Подробнее]]
- Подробный вывод (`-v`) | [[Golang. Пакет testing. Verbose output|Подробнее]]
- Список доступных тестов (`-list`) | [[Golang. Пакет testing. List tests|Подробнее]]
- Короткий режим (`-short`) | [[Golang. Пакет testing. Short mode|Подробнее]]
- JSON-вывод результатов (`-json`) | [[Golang. Пакет testing. JSON output|Подробнее]]
- Запуск всех пакетов (`./...`) | [[Golang. Пакет testing. All packages|Подробнее]]

---

## Окружение тестов

- Временные директории (`t.TempDir`) | [[Golang. Пакет testing. TempDir|Подробнее]]
- Переменные окружения (`t.Setenv`) | [[Golang. Пакет testing. Setenv|Подробнее]]
- Изменение рабочей директории (`t.Chdir`) | [[Golang. Пакет testing. Chdir|Подробнее]]
- Управление временными ресурсами | [[Golang. Пакет testing. Resources management|Подробнее]]
- Автоматическая очистка ресурсов | [[Golang. Пакет testing. Cleanup resources|Подробнее]]
- Работа с временными файлами | [[Golang. Пакет testing. Temporary files|Подробнее]]

---

## Тестовые паттерны

- Табличные тесты | [[Golang. Пакет testing. Table-driven pattern|Подробнее]]
- Тестирование ошибок | [[Golang. Пакет testing. Error testing|Подробнее]]
- Проверка граничных случаев | [[Golang. Пакет testing. Edge cases|Подробнее]]
- Проверка конкурентного кода | [[Golang. Пакет testing. Concurrent testing|Подробнее]]
- Тестирование интерфейсов | [[Golang. Пакет testing. Interface testing|Подробнее]]
- Dependency Injection в тестах | [[Golang. Пакет testing. Dependency Injection|Подробнее]]
- Mock-объекты | [[Golang. Пакет testing. Mock objects|Подробнее]]
- Stub-объекты | [[Golang. Пакет testing. Stub objects|Подробнее]]
- Fake-реализации | [[Golang. Пакет testing. Fake implementations|Подробнее]]
- Test doubles | [[Golang. Пакет testing. Test doubles|Подробнее]]

---

## HTTP / API тестирование

- Тестирование HTTP-клиентов | [[Golang. Пакет testing. HTTP client testing|Подробнее]]
- Тестирование HTTP-серверов | [[Golang. Пакет testing. HTTP server testing|Подробнее]]
- Тестирование handlers | [[Golang. Пакет testing. Handlers testing|Подробнее]]
- Тестирование middleware | [[Golang. Пакет testing. Middleware testing|Подробнее]]
- Тестирование REST API | [[Golang. Пакет testing. REST API testing|Подробнее]]
- Тестирование JSON API | [[Golang. Пакет testing. JSON API testing|Подробнее]]
- Использование `httptest` | [[Golang. Пакет httptest|Подробнее]]
- Проверка HTTP запросов | [[Golang. Пакет testing. HTTP requests|Подробнее]]
- Проверка HTTP ответов | [[Golang. Пакет testing. HTTP responses|Подробнее]]

---

## Интеграционные тесты

- Разделение unit и integration тестов | [[Golang. Пакет testing. Unit vs Integration|Подробнее]]
- Build tags для тестов | [[Golang. Пакет testing. Build tags|Подробнее]]
- Тестирование внешних сервисов | [[Golang. Пакет testing. External services|Подробнее]]
- Тестирование базы данных | [[Golang. Пакет testing. Database testing|Подробнее]]
- Тестирование миграций | [[Golang. Пакет testing. Migration testing|Подробнее]]
- Docker-based integration tests | [[Golang. Пакет testing. Docker integration tests|Подробнее]]
- Тестирование очередей сообщений | [[Golang. Пакет testing. Message queues testing|Подробнее]]
- Тестирование брокеров сообщений | [[Golang. Пакет testing. Brokers testing|Подробнее]]

---

## Покрытие кода

- Запуск покрытия (`go test -cover`) | [[Golang. Пакет testing. Code coverage|Подробнее]]
- Coverage profile (`-coverprofile`) | [[Golang. Пакет testing. Cover profile|Подробнее]]
- Анализ покрытия (`go tool cover`) | [[Golang. Пакет testing. Cover tool|Подробнее]]
- Просмотр покрытия в HTML | [[Golang. Пакет testing. HTML coverage|Подробнее]]
- Отчёт по покрытию кода | [[Golang. Пакет testing. Coverage report|Подробнее]]

---

## Основные типы testing API

- `testing.T` — обычные тесты | [[Golang. Пакет testing. testing.T|Подробнее]]
- `testing.B` — benchmark | [[Golang. Пакет testing. testing.B|Подробнее]]
- `testing.F` — fuzzing | [[Golang. Пакет testing. testing.F|Подробнее]]
- `testing.M` — управление жизненным циклом тестов | [[Golang. Пакет testing. testing.M|Подробнее]]
- `testing.PB` — параллельные benchmark | [[Golang. Пакет testing. testing.PB|Подробнее]]