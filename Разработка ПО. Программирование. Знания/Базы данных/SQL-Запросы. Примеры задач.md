# Шпора: Сложные SQL-запросы для собесов

## Схема (простая, но богатая)

```sql
-- Сотрудники
CREATE TABLE employees (
    id          SERIAL PRIMARY KEY,
    name        TEXT NOT NULL,
    dept_id     INT REFERENCES departments(id),
    manager_id  INT REFERENCES employees(id),   -- self-join
    salary      NUMERIC(10,2),
    hired_at    DATE
);

-- Отделы
CREATE TABLE departments (
    id    SERIAL PRIMARY KEY,
    name  TEXT NOT NULL
);

-- Проекты
CREATE TABLE projects (
    id         SERIAL PRIMARY KEY,
    name       TEXT,
    dept_id    INT REFERENCES departments(id)
);

-- Назначения на проекты (M:N)
CREATE TABLE assignments (
    emp_id      INT REFERENCES employees(id),
    project_id  INT REFERENCES projects(id),
    hours       INT,
    PRIMARY KEY (emp_id, project_id)
);

-- Продажи (для оконных функций)
CREATE TABLE sales (
    id        SERIAL PRIMARY KEY,
    emp_id    INT,
    amount    NUMERIC(10,2),
    sale_date DATE
);
```

## 1. Второй по величине оклад в отделе

**Оконная функция `DENSE_RANK`:**
```sql
SELECT dept_id, name, salary
FROM (
    SELECT dept_id, name, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rk
    FROM employees
) t
WHERE rk = 2;
```
> ⚠️ Ловушка: `LIMIT 1 OFFSET 1` не работает per-department.

## 2. Топ-3 сотрудника по окладу в каждом отделе

```sql
SELECT *
FROM (
    SELECT e.*, 
           ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
    FROM employees e
) t
WHERE rn <= 3;
```
**Разница `ROW_NUMBER` / `RANK` / `DENSE_RANK`:**
- `ROW_NUMBER` — 1,2,3,4 (всегда уникально)
- `RANK` — 1,2,2,4 (пропуск после ничьей)
- `DENSE_RANK` — 1,2,2,3 (без пропуска)

## 3. Сотрудники, зарабатывающие больше среднего по отделу

```sql
SELECT e.*
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
) d ON e.dept_id = d.dept_id
WHERE e.salary > d.avg_sal;
```
**Или оконной функцией (быстрее, один проход):**
```sql
SELECT *
FROM (
    SELECT *, AVG(salary) OVER (PARTITION BY dept_id) AS avg_sal
    FROM employees
) t
WHERE salary > avg_sal;
```

## 4. Найти дубликаты

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Удалить дубликаты, оставить минимальный id:
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id) FROM users GROUP BY email
);
```

## 5. Сотрудники без проектов (анти-join)

**Три способа:**
```sql
-- NOT EXISTS (предпочтительно)
SELECT e.* FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM assignments a WHERE a.emp_id = e.id
);

-- LEFT JOIN ... IS NULL
SELECT e.* FROM employees e
LEFT JOIN assignments a ON a.emp_id = e.id
WHERE a.emp_id IS NULL;

-- NOT IN (ОПАСНО при NULL!)
SELECT e.* FROM employees e
WHERE e.id NOT IN (SELECT emp_id FROM assignments);
```
> ⚠️ `NOT IN` ломается если в подзапросе есть `NULL` → вернёт пусто.

## 6. Иерархия: все подчинённые менеджера (рекурсия)

```sql
WITH RECURSIVE subordinates AS (
    -- якорь: сам менеджер
    SELECT id, name, manager_id, 1 AS lvl
    FROM employees WHERE id = 1

    UNION ALL

    -- рекурсия: подчинённые
    SELECT e.id, e.name, e.manager_id, s.lvl + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates ORDER BY lvl;
```

## 7. Running total (накопительный итог)

```sql
SELECT emp_id, sale_date, amount,
       SUM(amount) OVER (
           PARTITION BY emp_id
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM sales;
```

## 8. Разница с предыдущей продажей (`LAG` / `LEAD`)

```sql
SELECT emp_id, sale_date, amount,
       amount - LAG(amount) OVER (PARTITION BY emp_id ORDER BY sale_date) AS diff
FROM sales;
```

## 9. Непрерывные дни активности (gaps & islands)

Классика! Найти серии подряд идущих дат:
```sql
WITH grp AS (
    SELECT user_id, login_date,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date))::int AS grp
    FROM logins
)
SELECT user_id, MIN(login_date), MAX(login_date), COUNT(*)
FROM grp
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;   -- серии ≥ 3 дней
```
**Идея:** разница между датой и row_number постоянна внутри серии.

## 10. Процент от общего (доля)

```sql
SELECT dept_id,
       SUM(salary) AS dept_total,
       ROUND(100.0 * SUM(salary) / SUM(SUM(salary)) OVER (), 2) AS pct
FROM employees
GROUP BY dept_id;
```
> `SUM(SUM(...)) OVER ()` — оконная функция поверх агрегата.

## 11. Self-join: сотрудники с одинаковым окладом

```sql
SELECT DISTINCT e1.name, e2.name, e1.salary
FROM employees e1
JOIN employees e2 
  ON e1.salary = e2.salary AND e1.id < e2.id;
```
> `e1.id < e2.id` — убирает зеркальные пары.

## 12. Пара «менеджер — подчинённый» (self-join)

```sql
SELECT e.name AS emp, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id;
```

## 13. Pivot: продажи по месяцам в колонки

```sql
SELECT emp_id,
       SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 1 THEN amount END) AS jan,
       SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 2 THEN amount END) AS feb,
       SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 3 THEN amount END) AS mar
FROM sales
GROUP BY emp_id;
```

## 14. Nth highest salary (общая задача)

```sql
-- N = 3
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;

-- Через DENSE_RANK (работает с группами):
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rk
    FROM employees
) t WHERE rk = 3;
```

## 15. Отделы с макс. средней зарплатой

```sql
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id
ORDER BY avg_sal DESC
LIMIT 1;

-- Или все отделы с максимумом (если ничья):
SELECT dept_id, avg_sal FROM (
    SELECT dept_id, AVG(salary) AS avg_sal,
           RANK() OVER (ORDER BY AVG(salary) DESC) AS rk
    FROM employees GROUP BY dept_id
) t WHERE rk = 1;
```

## 16. Сотрудники, работавшие над всеми проектами отдела (реляционное деление)

```sql
SELECT e.id, e.name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM projects p
    WHERE p.dept_id = e.dept_id
      AND NOT EXISTS (
          SELECT 1 FROM assignments a
          WHERE a.emp_id = e.id AND a.project_id = p.id
      )
);
```
**Идея:** «нет проекта, над которым он не работал» = «работал над всеми».

## 17. Месяцы без продаж (gap в датах)

```sql
WITH months AS (
    SELECT generate_series('2024-01-01'::date, '2024-12-01', '1 month') AS m
)
SELECT m FROM months
WHERE NOT EXISTS (
    SELECT 1 FROM sales
    WHERE date_trunc('month', sale_date) = m
);
```

## 18. Обновление через CTE / UPSERT

```sql
INSERT INTO employees (id, name, salary)
VALUES (1, 'Ivan', 100000)
ON CONFLICT (id) DO UPDATE
SET salary = EXCLUDED.salary;
```

## 19. Рейтинг + фильтр по окну (нельзя в WHERE!)

```sql
-- ❌ Не работает: оконные функции нельзя в WHERE
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM employees
) t WHERE rn <= 5;   -- ✅ через подзапрос/CTE
```

## 20. Медиана (PostgreSQL)

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median
FROM employees;

-- Или вручную через ROW_NUMBER:
SELECT AVG(salary) FROM (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER () AS cnt
    FROM employees
) t
WHERE rn IN ((cnt+1)/2, (cnt+2)/2);
```

## Шпаргалка по конструкциям

| Задача                     | Инструмент                             |
| -------------------------- | -------------------------------------- |
| Топ-N в группе             | `ROW_NUMBER() OVER (PARTITION BY ...)` |
| Сравнить с предыдущей      | `LAG` / `LEAD`                         |
| Накопительный итог         | `SUM() OVER (ORDER BY ...)`            |
| Доля от целого             | `x / SUM(x) OVER ()`                   |
| Анти-join                  | `NOT EXISTS`                           |
| Иерархия                   | `WITH RECURSIVE`                       |
| Деление (все/хотя бы один) | двойной `NOT EXISTS`                   |
| Gaps & islands             | `date - ROW_NUMBER()`                  |
| Pivot                      | `SUM(CASE WHEN ...)`                   |
| Дубликаты                  | `GROUP BY ... HAVING COUNT(*) > 1`     |

## Частые ловушки на собесах

1. **NULL в `NOT IN`** → пустой результат
2. **Оконные функции в `WHERE`** → нельзя, только в подзапросе
3. **`COUNT(*)` vs `COUNT(col)`** — второй игнорирует NULL
4. **`AVG` игнорирует NULL** — не то же, что `SUM/COUNT(*)`
5. **`LEFT JOIN` + условие в `WHERE`** → превращается в `INNER JOIN`, условие для правой таблицы — в `ON`
6. **`GROUP BY` и `SELECT`** — все неагрегированные колонки в GROUP BY
7. **`DISTINCT` + `ORDER BY`** — сортировка только по выбранным колонкам
8. **Деление на ноль** → `NULLIF(denom, 0)`

## Порядок выполнения SQL-запроса

```
FROM → JOIN → WHERE → GROUP BY → HAVING 
     → SELECT → DISTINCT → ORDER BY → LIMIT
```
> Поэтому алиасы из `SELECT` доступны в `ORDER BY`, но не в `WHERE`.

---
*Теги: #БД #SQL #собеседование #оконные_функции #CTE #JOIN*