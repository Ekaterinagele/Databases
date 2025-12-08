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

<img width="1332" height="568" alt="dbdiagram.io" src="https://github.com/user-attachments/assets/610bee37-cadc-4455-8b2f-bb13a3ae6cb7" />

### Логическая модель в виде Диаграммы классов UML-2.4

<img width="3920" height="2356" alt="Логическая модель" src="https://github.com/user-attachments/assets/3a6e3393-fd98-4420-ad85-c97004447d03" />

### Физическая модель 
<img width="4973" height="2643" alt="Физическая модель" src="https://github.com/user-attachments/assets/e3b6a736-e46f-4948-98ad-096e4719a71e" />

# Лабораторная работа 2
## Создание DDL-запросов для PostgreSQL
### Создана структура БД в pgAdmin

### Таблица students 

<img width="1107" height="681" alt="students" src="https://github.com/user-attachments/assets/06736f22-8ab1-4b2c-9a8f-257b4d560e22" />

### Таблица subjects

<img width="1115" height="677" alt="subjects" src="https://github.com/user-attachments/assets/e7dfabdf-c769-4d16-a719-53a2d288dd95" />

### Таблица record_types

<img width="1121" height="681" alt="record_types" src="https://github.com/user-attachments/assets/79ccf799-d1af-4fe1-9040-27860229d661" />

### Таблица addresses

<img width="1118" height="677" alt="addresses" src="https://github.com/user-attachments/assets/6b3f13f6-876e-48ca-abe2-c402a7be4823" />

### Таблица grades

<img width="1120" height="681" alt="grades" src="https://github.com/user-attachments/assets/36f2f56c-35c2-45bf-a96c-a77863f09871" />

### Таблица parents

<img width="1119" height="682" alt="parents" src="https://github.com/user-attachments/assets/83787486-2241-4104-8ecf-95897756b824" />

## Таблицы заполнены данными

### Таблица parents

<img width="975" height="1182" alt="1" src="https://github.com/user-attachments/assets/7f467ee4-1b9c-410c-a979-0b15e1579fa2" />

### Таблица addresses

<img width="976" height="1106" alt="2" src="https://github.com/user-attachments/assets/16a890e8-6193-42d1-b949-a52d2a5f0a82" />

### Таблица students

<img width="1183" height="1063" alt="22" src="https://github.com/user-attachments/assets/d206a0fe-b58b-4526-a842-df550a283fdc" />

### Таблица subjects

<img width="1216" height="1034" alt="23" src="https://github.com/user-attachments/assets/d31d6fa6-2515-404c-a9bb-24c73829bb41" />

### Таблица record_types

<img width="1168" height="1086" alt="555555" src="https://github.com/user-attachments/assets/ae25fae5-9664-40b8-9132-7ac330489040" />


### Таблица grades

<img width="977" height="1415" alt="students_1" src="https://github.com/user-attachments/assets/6b6f868e-b290-45b9-bc74-88232b6fde0b" />

## Выполнены SELECT-запросы c JOIN

### Запрос 1.
**Выведем список всех учеников, которые получили пятерки по математике**.

Запрос объединяет таблицы **students, grades, subjects**.
В результате получим таблицу состоящую из трех строк, содержащих фамилию, имя ученика, предмет и оценку. 

<img width="861" height="569" alt="запрос 2" src="https://github.com/user-attachments/assets/806ccf7b-1128-4f8b-8bdc-d87d527127d2" />

### Запрос 2. 
**Выдать список учеников, получивших двойки или пропустивших занятия в определенный интервал дат**.

Запрос объединяет таблицы **students, grades, record_types**.
В результате получим таблицу состоящую из трех строк, содержащих фамилию, имя и отчество ученика.

![1](https://github.com/user-attachments/assets/d58a8145-0b6e-41e2-b986-420b02d35118)

### Запрос 3.
**Выдать список учеников и предметов, по которым число пятерок больше заданного, отсортированный ученикам и предметам.**

Запрос объединяет таблицы **students, grades, subjects**.
В результате получим таблицу состоящую из девяти строк, содержащих фамилию, имя и отчество ученика, предмет и количество пятерок.

![3](https://github.com/user-attachments/assets/1ccfc3c1-b7c4-41b9-93fa-30ba18f265ff)


# Лабораторная работа 3
## Представления и процедуры
### Создано VIEW для запроса

![view](https://github.com/user-attachments/assets/28ab450d-0a20-4c30-b74b-bc8dedc22882)

**Проверка работы представления**

<img width="685" height="380" alt="проверка1" src="https://github.com/user-attachments/assets/25157b8a-5db0-47c3-81b9-7f6bc98ebd1e" />

### Создана функция get_students_with_min_fives

Создана функиция для поиска учеников с количеством пятерок больше заданного

<img width="835" height="667" alt="function" src="https://github.com/user-attachments/assets/3eeef5ce-1df7-4457-b1df-5a8108523574" />

**Проверка работы функции**

<img width="690" height="447" alt="проверка2" src="https://github.com/user-attachments/assets/6b62510f-99d0-45ed-8204-6d628691b1aa" />

### Созданы процедуры для редактирования 

<img width="964" height="772" alt="процедуры" src="https://github.com/user-attachments/assets/b81ed293-7c98-4ab0-a4cd-95b047a5a938" />

# Лабораторная работа 4
## Анализ производительности
### Созданы генераторы данных, заполняющие таблицы 20000 записями

## Генеретор для таблицы *addresses*

<img width="1179" height="564" alt="lab4 1" src="https://github.com/user-attachments/assets/7af4e494-f439-4c7a-ab44-2b591ab8f5a4" />

## **Проверка работы генератора**

<img width="1023" height="225" alt="lab4 1 1" src="https://github.com/user-attachments/assets/054a2da4-9f1c-404f-bb11-7eb7588245e3" />

## Генератор для таблицы *parents*

<img width="1049" height="805" alt="lab4 2" src="https://github.com/user-attachments/assets/03450655-e0f5-463c-b7d8-bdfc1f801c4e" />

## **Проверка работы генератора**

<img width="1233" height="213" alt="lab4 2 2" src="https://github.com/user-attachments/assets/2de0c384-4d4d-4c65-86e4-37ce34e9763e" />

## Генератор для таблицы *students*

<img width="1196" height="802" alt="lab4 3" src="https://github.com/user-attachments/assets/676ce710-4d0f-4e8a-a507-a37064faf8e2" />

## **Проверка работы генератора**

<img width="1329" height="333" alt="lab4 3 2png" src="https://github.com/user-attachments/assets/86eba20d-aaf2-4786-a1d0-a6739e4ee837" />

## Генератор для таблицы *subjects*

<img width="1198" height="462" alt="lab4 4" src="https://github.com/user-attachments/assets/d7a71d12-267c-47e1-ab5e-a23ac38bccd0" />

## **Проверка работы генератора**

<img width="1067" height="351" alt="lab4 4 1" src="https://github.com/user-attachments/assets/d1c09606-ede0-4cd8-b287-48c7723641db" />
