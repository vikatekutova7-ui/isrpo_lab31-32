# Лабораторная работа №31-32. Введение в SQLite и Entity Framework Core

## Информация о студенте

**ФИО:** Текутова В.Д.
**Группа:** ИСП-231
**Дата выполнения:** 24.04.26

---

## Краткое описание работы

Веб-API для управления задачами на ASP.NET Core 10 с Entity Framework Core и SQLite. Реализованы CRUD-операции, поиск, фильтрация, пагинация, статистика и миграции Code First.

---

## Полезные команды dotnet ef

| Команда | Описание |
|---------|----------|
| `dotnet ef migrations add <Имя>` | Создать новую миграцию |
| `dotnet ef database update` | Применить все миграции к БД |
| `dotnet ef migrations list` | Показать список миграций |
| `dotnet ef migrations remove` | Удалить последнюю миграцию |
| `dotnet ef database drop` | Удалить базу данных |
| `dotnet ef migrations script` | Сгенерировать SQL-скрипт миграции |

---

## Структура проекта
Lab31-32_EFCore/
├── TaskDb/
│ ├── Controllers/
│ │ └── TasksController.cs
│ ├── Data/
│ │ └── AppDbContext.cs
│ ├── Migrations/
│ ├── Models/
│ │ ├── TaskItem.cs
│ │ └── TaskDtos.cs
│ ├── Properties/
│ │ └── launchSettings.json
│ ├── appsettings.json
│ ├── Program.cs
│ └── taskdb.db
├── img/
├── README.md
└── .editorconfig


---

## Реализованные маршруты API

### Базовые CRUD операции

| Метод | Маршрут | Описание |
|-------|---------|----------|
| GET | `/api/tasks` | Получить все задачи (с фильтрацией) |
| GET | `/api/tasks/{id}` | Получить задачу по ID |
| POST | `/api/tasks` | Создать новую задачу |
| PUT | `/api/tasks/{id}` | Полностью обновить задачу |
| PATCH | `/api/tasks/{id}/complete` | Переключить статус выполнения |
| DELETE | `/api/tasks/{id}` | Удалить задачу |

### Дополнительные маршруты

| Метод | Маршрут | Описание |
|-------|---------|----------|
| GET | `/api/tasks/search` | Поиск задач с фильтрами |
| GET | `/api/tasks/stats` | Получить статистику по задачам |
| GET | `/api/tasks/paged` | Пагинированный список задач |
| GET | `/api/tasks/overdue` | Получить просроченные задачи |
| PATCH | `/api/tasks/complete-all` | Отметить все задачи как выполненные |
| DELETE | `/api/tasks/completed` | Удалить все выполненные задачи |

---

## Применённые миграции

| № | Название миграции | Описание изменений |
|---|-------------------|-------------------|
| 1 | `InitialCreate` | Создание таблицы Tasks с начальными данными (3 задачи) |
| 2 | `AddDueDateToTask` | Добавление поля `DueDate` (срок выполнения) в таблицу Tasks |

---

## Сравнение LINQ и SQL

| LINQ (C#) | SQL (генерируется EF Core) |
|-----------|---------------------------|
| `db.Tasks.ToList()` | `SELECT * FROM Tasks` |
| `db.Tasks.Find(5)` | `SELECT * FROM Tasks WHERE Id = 5` |
| `.Where(t => t.IsCompleted == false)` | `WHERE IsCompleted = 0` |
| `.Where(t => t.Title.Contains("SQL"))` | `WHERE Title LIKE '%SQL%'` |
| `.OrderByDescending(t => t.CreatedAt)` | `ORDER BY CreatedAt DESC` |
| `.Take(10)` | `LIMIT 10` |
| `.Skip(20).Take(10)` | `OFFSET 20 LIMIT 10` |
| `.Count()` | `SELECT COUNT(*)` |
| `.Count(t => t.IsCompleted)` | `SELECT COUNT(*) WHERE IsCompleted = 1` |
| `.GroupBy(t => t.Priority)` | `GROUP BY Priority` |
| `.Select(t => t.Title)` | `SELECT Title` |
| `.Any(t => t.Priority == "High")` | `EXISTS (SELECT 1 WHERE Priority = 'High')` |

---

## Сравнительная таблица: Хранение в памяти vs EF Core + SQLite

| Концепция | Хранение в памяти (`static List<T>`) | EF Core + SQLite |
|-----------|--------------------------------------|------------------|
| **Хранение данных** | `static List<T>` в RAM | Файл `.db` на диске |
| **После перезапуска** | Данные пропадают | Данные сохраняются |
| **Поиск по условию** | LINQ to Objects | LINQ to Entities → SQL |
| **Создание структуры** | Не нужно | Миграции (`dotnet ef`) |
| **Начальные данные** | Хардкод в коде | `HasData()` в миграции |
| **Получение данных** | `list.FirstOrDefault(...)` | `await db.Table.FindAsync(id)` |
| **Добавление** | `list.Add(item)` | `db.Table.Add(item) + SaveChangesAsync()` |
| **Удаление** | `list.Remove(item)` | `db.Table.Remove(item) + SaveChangesAsync()` |
| **Масштабируемость** | Ограничена RAM | Гигабайты данных |
| **Транзакции** | Нет | Встроены в EF Core |

---

## Главные выводы

11. EF Core переводит LINQ-запросы в SQL автоматически — разработчик пишет на C#, а в базу уходит SQL.
2. Миграции позволяют версионировать структуру базы данных, как Git для кода.
3. Code First подход удобен: изменил класс C# → создал миграцию → база автоматически обновлена.
4. SaveChangesAsync() — ключевой момент: все изменения до него существуют только в памяти.
5. async/await при работе с БД обязателен — блокировать поток на время ожидания ответа от базы нельзя.
