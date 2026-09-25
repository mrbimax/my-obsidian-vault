# Доменный слой платформы для изучения языков (Go)

## 1. Базовые строительные блоки (shared kernel)

- [ ] Пакет `domain/shared`
- [ ] Тип `ID` (UUID / ULID) + генератор
- [ ] Базовый интерфейс `Entity[ID]` (`GetID`, `Equal`)
- [ ] Базовый интерфейс `AggregateRoot` (`GetVersion`, `PullEvents`)
- [ ] Интерфейс `DomainEvent` (`EventName`, `OccurredAt`)
- [ ] Базовый `ValueObject` (immutable, `Equal`)
- [ ] Ошибки домена (`ErrNotFound`, `ErrConflict`, `ErrValidation`, `ErrForbidden`)
- [ ] Тип `DomainError` с кодом, сообщением и обёрткой
- [ ] Коллекция `Result[T]` / `Either[L,R]` (опционально)
- [ ] Пагинация (`Page`, `Cursor`, `Limit`)

## 2. Пользователи и роли

- [ ] `User` (aggregate root): ID, email, хеш пароля, статус, локаль, таймзона, создан/обновлён
- [ ] `UserProfile`: имя, аватар, цели обучения, родной язык
- [ ] `Role` / `Permission` (learner, teacher, admin, content-author)
- [ ] `Session` / `RefreshToken`
- [ ] Интерфейсы: `UserRepository`, `SessionRepository`
- [ ] Доменные события: `UserRegistered`, `UserPasswordChanged`, `UserDeactivated`

## 3. Языки и уровни

- [ ] `Language` (ISO 639-1/639-3, флаг, RTL)
- [ ] `LanguagePair` (source → target)
- [ ] `CEFRLevel` (A1…C2) как value object
- [ ] `Proficiency` (уровень пользователя по языку)
- [ ] Интерфейсы: `LanguageRepository`, `ProficiencyRepository`

## 4. Контент: карточки, слова, фразы

- [ ] `Card` (aggregate): ID, тип, front/back, media, теги, сложность
- [ ] `CardType` (enum: word, phrase, sentence, cloze, image, audio)
- [ ] `Media` (value object): URL, тип, размер, длительность
- [ ] `Translation` (value object)
- [ ] `Example` (value object)
- [ ] `Tag` / `TagSet`
- [ ] `Deck` (коллекция карточек): ID, автор, приватность, язык
- [ ] Интерфейсы: `CardRepository`, `DeckRepository`, `TagRepository`
- [ ] Доменные события: `CardCreated`, `CardUpdated`, `DeckPublished`

## 5. SRS / прогресс обучения

- [ ] `ReviewItem` (состояние карточки у пользователя)
- [ ] `SRSState` (ease factor, interval, repetitions, due date)
- [ ] `ReviewLog` (оценка, время, latency)
- [ ] `Rating` (again / hard / good / easy)
- [ ] Интерфейс `Scheduler` (стратегия: SM-2, FSRS, Leitner)
- [ ] `LearningStats` (streak, retention, XP)
- [ ] Интерфейсы: `ReviewRepository`, `SchedulerRepository`
- [ ] Доменные события: `CardReviewed`, `StreakBroken`, `LevelUp`

## 6. Уроки, курсы, программы

- [ ] `Course` (aggregate): ID, язык, уровень, автор, статус
- [ ] `Module` / `Unit`
- [ ] `Lesson` (набор упражнений + теория)
- [ ] `Exercise` (интерфейс + реализации: choice, translate, listen, speak)
- [ ] `LessonProgress`
- [ ] `Enrollment` (запись пользователя на курс)
- [ ] Интерфейсы: `CourseRepository`, `LessonRepository`, `ExerciseRepository`
- [ ] Доменные события: `CourseCompleted`, `LessonStarted`, `LessonFinished`

## 7. Usecases (гибкий движок)

- [ ] `Usecase` (aggregate): ID, название, шаги, триггеры, награды
- [ ] `UsecaseStep` (интерфейс + реализации)
- [ ] `StepResult` (success/fail, score, payload)
- [ ] `Condition` (предикаты: уровень, время, прогресс)
- [ ] `Reward` (XP, badge, unlock)
- [ ] `UsecaseRun` (инстанс прохождения пользователем)
- [ ] DSL / декларативное описание usecase (JSON/YAML → домен)
- [ ] Интерфейсы: `UsecaseRepository`, `UsecaseRunner`, `ConditionEvaluator`
- [ ] Доменные события: `UsecaseStarted`, `UsecaseStepCompleted`, `UsecaseFinished`

## 8. Игры и активности

- [ ] `Game` (aggregate): тип, правила, сложность
- [ ] `GameSession` (состояние партии)
- [ ] `GameMode` (quiz, match, memory, speed-run, battle)
- [ ] `Score` / `Leaderboard`
- [ ] Интерфейсы: `GameRepository`, `GameEngine`, `LeaderboardRepository`
- [ ] Доменные события: `GameStarted`, `GameFinished`, `HighScoreReached`

## 9. Достижения, геймификация

- [ ] `Achievement` (условие + награда)
- [ ] `Badge` / `Medal`
- [ ] `UserAchievement` (прогресс, дата получения)
- [ ] `Streak` (value object)
- [ ] `XP` / `Level` (value objects с правилами повышения)
- [ ] Интерфейсы: `AchievementRepository`, `AchievementEvaluator`
- [ ] Доменные события: `AchievementUnlocked`, `BadgeEarned`

## 10. Социальные фичи (опционально)

- [ ] `Friendship`
- [ ] `Chat` / `Message`
- [ ] `Classroom` (учитель + ученики)
- [ ] `Assignment`
- [ ] Интерфейсы: `FriendshipRepository`, `ChatRepository`, `ClassroomRepository`

## 11. Медиа и произношение

- [ ] `Audio` / `Pronunciation` (value object)
- [ ] `SpeechEvaluation` (score, feedback)
- [ ] Интерфейс `SpeechEvaluator`
- [ ] Интерфейс `TTSService` (внешний порт)
- [ ] Интерфейс `ASRService` (внешний порт)

## 12. Аналитика и события обучения

- [ ] `LearningEvent` (обобщённый)
- [ ] `Metrics` (value object)
- [ ] Интерфейс `EventBus` (`Publish`, `Subscribe`)
- [ ] Интерфейс `EventStore` (append-only)
- [ ] Обработчики доменных событий (`EventHandler`)

## 13. Интерфейсы репозиториев (порты)

- [ ] Единый паттерн: `Save`, `GetByID`, `Delete`, `List(ctx, filter, page)`
- [ ] `Filter` как value object (не `map[string]any`)
- [ ] Контекст во всех методах (`context.Context`)
- [ ] Только доменные типы в сигнатурах (без ORM-моделей)
- [ ] Ошибки — доменные (`ErrNotFound`, не `sql.ErrNoRows`)

## 14. Сервисы домена (чистая логика)

- [ ] `Scheduler` (SRS-алгоритм)
- [ ] `UsecaseRunner` (движок исполнения usecase)
- [ ] `AchievementEvaluator`
- [ ] `LevelCalculator`
- [ ] `ScoreCalculator`
- [ ] `ConditionEvaluator`
- [ ] Отсутствуют зависимости от инфраструктуры (БД, HTTP, кэш)

## 15. Правила и инварианты

- [ ] Все конструкторы — `NewXxx(...)` с валидацией
- [ ] Приватные поля, геттеры без сеттеров (изменения — через методы)
- [ ] Доменные события пушатся в агрегат, не публикуются напрямую
- [ ] Нет анемичных моделей: логика живёт в домене
- [ ] Нет импортов из `infra`, `transport`, `app` (только stdlib + shared)
- [ ] Юнит-тесты без моков инфраструктуры (только чистые)

## 16. Тестируемость и качество

- [ ] Табличные тесты для value objects
- [ ] Тесты инвариантов агрегатов
- [ ] Тесты доменных событий
- [ ] Фаззинг для парсеров (DSL usecase)
- [ ] Бенчмарки для SRS / scoring
- [ ] Покрытие домена ≥ 90%

## 17. Документация

- [ ] `doc.go` в каждом пакете
- [ ] Диаграмма агрегатов (PlantUML / Mermaid)
- [ ] Глоссарий терминов домена
- [ ] Описание доменных событий и их контрактов
- [ ] ADR на ключевые решения (SRS-алгоритм, формат Usecase DSL)

## 18. Структура пакетов

- [ ] `internal/domain/shared` — ID, ValueObject, Event, Error
- [ ] `internal/domain/user`
- [ ] `internal/domain/language`
- [ ] `internal/domain/content` — card, deck, media
- [ ] `internal/domain/learning` — srs, review, progress
- [ ] `internal/domain/course` — course, lesson, exercise
- [ ] `internal/domain/usecase` — движок usecases
- [ ] `internal/domain/game`
- [ ] `internal/domain/achievement`
- [ ] `internal/domain/social`
- [ ] `internal/domain/analytics`

## Критерий готовности доменного слоя

- [ ] Домен компилируется без зависимостей на инфраструктуру
- [ ] Все бизнес-правила покрыты тестами
- [ ] Публичные API стабильны и документированы
- [ ] Готов к реализации репозиториев в `infra/` и оркестрации в `app/`