# Лабораторные работы по БД

## Постановка задачи:

**Классное руководство (18 вариант)**

**Условие задачи:**

Имеются ученики (фамилия, имя, домашний адрес, телефон, ФИО отца
и матери) и предметы (название). Накапливаются сведения об
успеваемости учеников по предметам с указанием оценок, дат и сведений
о пропусках.

**Сущности:**

    Ученики (student_id, ФИО, Имя, Отчество, Адрес, Номер телефона, ФИО отца, ФИО матери, Номер телефона матери),
    Предметы (student_id, Название предмета), 
    Оценки (student_id, Оценка, Дата, Тип записи),
    Типы записей (type_id, Название записи).
    
**Процессы:** (Отношения)

    Накапливаются сведения об успеваемости учеников по предметам с указанием оценок, дат и сведений о пропусках.

**Входные документы:**
   - Выдать список учеников, получивших двойки или пропустивших занятия в определенный интервал дат.

   - Выдать список учеников и предметов, по которым число пятерок больше заданного, отсортированный ученикам и предметам.
# Лабораторная работа 1

### Базовые сущности 

    Ученики (student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, 
    father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number), первичный ключ - student_id,
    Предметы (subject_id, subject_name), первичный ключ - subject_id,
    Оценки (grade_id, student_id, subject_id, grade_value, date), первичный ключ - grade_id, внешние ключи - student_id, subject_id, type_id,
    Типы записей (type_id, type_name), первичный ключ - type_id
    
### Отношения 

[Ученики]-1,Required------------------N, Required-[Оценки]   

[Предметы]-1,Required------------------N, Required-[Оценки]

[Типы запией]-1,Required------------------N, Required-[Оценки]

## Логическая модель 
Используя правило 1, Required -- N, Required получаем две *таблицы*:
- ```students(student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number)```, primary key - student_id

- ```subjects(subject_id, subject_name)```, primary key - subject_id

- ```grades(grade_id, student_id, subject_id, grade_value, date, type_id)```, primary key - grade_ id, secondary key -(student_id, subject_id, type_id)
  
- ```record_types(type_id, type_name)```, primary key - type_id

## Физическая модель
```sql
-- Типы записей
CREATE TABLE record_types (
    type_id   INTEGER PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE
);

-- Ученики
CREATE TABLE students (
    student_id           INTEGER PRIMARY KEY,
    last_name            TEXT NOT NULL,
    first_name           TEXT NOT NULL,
    middle_name          TEXT,
    address              VARCHAR(100),
    phone_number         VARCHAR(20),
    father_last_name     TEXT,
    father_first_name    TEXT,
    father_middle_name   TEXT,
    mother_last_name     TEXT,
    mother_first_name    TEXT,
    mother_middle_name   TEXT,
    mother_phone_number  VARCHAR(20)
);

-- Предметы
CREATE TABLE subjects (
    subject_id   INTEGER PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE
);

-- Оценки и другие записи об успеваемости
CREATE TABLE grades (
    grade_id     INTEGER PRIMARY KEY,
    student_id   INTEGER NOT NULL,
    subject_id   INTEGER NOT NULL,
    grade_value  INTEGER,
    date         DATE NOT NULL,
    type_id      INTEGER NOT NULL,

    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (subject_id) REFERENCES subjects(subject_id) ON DELETE CASCADE,
    FOREIGN KEY (type_id)    REFERENCES record_types(type_id) ON DELETE RESTRICT
);
``` 
## Проверка нормальных форм: 
### Критические ошибки:

### Проблемы:

**1. Несоответствие постановки задачи и реализации**
В постановке: Предметы (student_id, Название предмета) - это *ошибка*! Предметы не должны зависеть от ученика. В реализации вы правильно сделали отдельную таблицу subjects.

**2. Проблема с типами записей и оценками:**
В таблице *grades* поле *grade_value* должно быть *NULLABLE*, поскольку:
* Для типа записи "пропуск" оценки не существует
* Для типа записи "оценка" поле обязательно

**3. Отсутствие индексов для производительности:**
Нет индексов для частых запросов по датам, студентам, предметам.

**Анализ нормальных форм:**
* **1NF - ✅ Соответствует**
Все поля атомарны, нет повторяющихся групп

* **2NF - ✅ Соответствует**
Нет частичных зависимостей от составного ключа

* **3NF - ⚠️ Есть проблемы**
*Нарушение в students:*
* address → phone_number (функциональная зависимость)
* Город/улица могут определять телефонный код

### Решение проблемы:
```sql
CREATE TABLE addresses (
    address_id SERIAL PRIMARY KEY,
    city VARCHAR(50) NOT NULL,
    street VARCHAR(100) NOT NULL,
    house VARCHAR(10) NOT NULL,
    apartment VARCHAR(10)
);
```
* **BCNF - ✅ Соответствует**
Нет нетривиальных зависимых от не-ключа атрибутов

**Исправления:**
**Исправленная физическая модель:**
```sql
-- Типы записей (расширенный вариант)
CREATE TABLE record_types (
    type_id SERIAL PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE,
    requires_grade BOOLEAN NOT NULL DEFAULT FALSE
);

-- Ученики (улучшенная версия)
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    last_name TEXT NOT NULL,
    first_name TEXT NOT NULL,
    middle_name TEXT,
    phone_number VARCHAR(20),
    
    -- Вынесены в отдельные таблицы
    father_id INTEGER REFERENCES parents(parent_id),
    mother_id INTEGER REFERENCES parents(parent_id),
    address_id INTEGER REFERENCES addresses(address_id)
);

-- Родители (новая таблица)
CREATE TABLE parents (
    parent_id SERIAL PRIMARY KEY,
    last_name TEXT NOT NULL,
    first_name TEXT NOT NULL,
    middle_name TEXT,
    phone_number VARCHAR(20)
);

-- Оценки с проверками
CREATE TABLE grades (
    grade_id SERIAL PRIMARY KEY,
    student_id INTEGER NOT NULL REFERENCES students(student_id) ON DELETE CASCADE,
    subject_id INTEGER NOT NULL REFERENCES subjects(subject_id) ON DELETE CASCADE,
    type_id INTEGER NOT NULL REFERENCES record_types(type_id) ON DELETE RESTRICT,
    grade_value INTEGER,
    date DATE NOT NULL,
    
    -- Проверка целостности данных
    CONSTRAINT valid_grade_check CHECK (
        (grade_value IS NULL AND (SELECT requires_grade FROM record_types WHERE type_id = grades.type_id) = false) OR
        (grade_value BETWEEN 1 AND 5 AND (SELECT requires_grade FROM record_types WHERE type_id = grades.type_id) = true)
    )
);

-- Индексы для производительности
CREATE INDEX idx_grades_student_date ON grades(student_id, date);
CREATE INDEX idx_grades_subject_grade ON grades(subject_id, grade_value);
CREATE INDEX idx_grades_date ON grades(date);
```
### Проверка требований:
**Запрос 1: Двойки и пропуски:**
```sql
-- Будет работать с текущей структурой
SELECT DISTINCT s.* 
FROM students s
JOIN grades g ON s.student_id = g.student_id
WHERE g.date BETWEEN '2024-01-01' AND '2024-12-31'
AND (g.grade_value = 2 OR g.type_id = (SELECT type_id FROM record_types WHERE type_name = 'пропуск'));
```

**Запрос 2: Много пятерок:**
```sql
-- Будет работать
SELECT s.student_id, s.last_name, s.first_name, sub.subject_name, COUNT(*) as five_count
FROM students s
JOIN grades g ON s.student_id = g.student_id
JOIN subjects sub ON g.subject_id = sub.subject_id
WHERE g.grade_value = 5
GROUP BY s.student_id, s.last_name, s.first_name, sub.subject_name
HAVING COUNT(*) > 3  -- больше 3 пятерок
ORDER BY s.last_name, s.first_name, sub.subject_name;
```
### Итоговые рекомендации:

*1. Исправить тип primary key на SERIAL/BIGSERIAL*

*2. Добавить проверки для grade_value (1-5)*

*3. Решить проблему с NULL в grade_value для пропусков*

*4. Добавить индексы*

*5. Рассмотреть нормализацию адресов и данных родителей*

*6. Добавить ограничения NOT NULL где необходимо*

## Полученные диаграммы:
### ER-диаграмма 

<img width="1623" height="568" alt="Untitled (5)" src="https://github.com/user-attachments/assets/0fccc931-949b-4ff4-91fa-66d2609f3aa9" />

### Логическая модель в виде Диаграммы классов UML-2.4

<img width="3920" height="2356" alt="deepseek_mermaid_20251013_502a06" src="https://github.com/user-attachments/assets/3a6e3393-fd98-4420-ad85-c97004447d03" />

### Физическая модель 


<img width="5036" height="3066" alt="deepseek_mermaid_20251027_968b9a" src="https://github.com/user-attachments/assets/927bb42d-9039-45df-9213-d6905fa1c493" />


