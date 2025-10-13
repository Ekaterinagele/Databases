# Лабораторные работы по БД

## Постановка задачи:

**Классное руководство (18 вариант)**

**Сущности:**

    Ученики (student_id, ФИО, Имя, Отчество, Адрес, Номер телефона, ФИО отца, ФИО матери, Номер телефона матери),
    Предметы (student_id, Название предмета), 
    Оценки (student_id, Оценка, Дата, Тип записи).

**Процессы:** (Отношения)

    Накапливаются сведения об успеваемости учеников по предметам с указанием оценок, дат и сведений о пропусках.

**Входные документы:**
   - Выдать список учеников, получивших двойки или пропустивших занятия в определенный интервал дат.

   - Выдать список учеников и предметов, по которым число пятерок больше заданного, отсортированный ученикам и предметам.
# Лабораторная работа 1

### Базовые сущности 

    Ученики (student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, 
    father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number), первичный ключ - student_id
    Предметы (subject_id, subject_name), первичный ключ - subject_id
    Оценки (grade_id, student_id, subject_id, grade_value, date), первичный ключ - grade_id, внешние ключи - student_id subject_id
### Отношения 

[Ученики]-1,Required------------------N, Required-[Оценки]   
[Предметы]-1,Required------------------N, Required-[Оценки]

## Логическая модель 
Используя правило 1, Required -- N, Required получаем две *таблицы*:
- ```students(student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number)```, primary key - student_id

- ```subjects(subject_id, subject_name)```, primary key - subject_id

Используя правило N,* -- M,* получаем итого три *таблицы*:

- ```students(student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number)```, primary key - student_id

- ```subjects(subject_id, subject_name)```, primary key - subject_id

- ```grades(grade_id, student_id, subject_id, grade_value, date)```, secondary key - (student_id, subject_id, date)

## Физическая модель
  - ```student_id::integer```
  - ```last_name::text```
  - ```first_name::text```
  - ```middle_name::text```
  - ```address::varchar(100)```
  - ```phone_number::varchar(20)```
  - ```father_last_name::text```
  - ```father_first_name::text```
  - ```father_middle_name::text```
  - ```mother_last_name::text```
  - ```mother_first_name::text```
  - ```mother_middle_name::text```
  - ```mother_phone_number::varchar(20)```
  - ```subject_id::integer```
  - ```subject_name::varchar(100)```
  - ```type_id::integer```
  - ```type_name::varchar(20)```
  - ```grade_id::integer```
  - ```student_id::integer```
  - ```subject_id::integer```
  - ```grade_value::integer```
  - ```date::date```
  - ```type_id::integer```

## Проверка нормальных форм: 
```text 
Отличная начальная работа! Вы хорошо структурировали задачу и продумали основные сущности. Однако в модели есть несколько критических ошибок и мест для улучшения.

Давайте разберем все по порядку.

Критические ошибки и замечания
Отсутствие сущности для "Типа записи" (пропуск/оценка):

Проблема: В исходной постановке есть "Тип записи" (оценка или пропуск). В вашей модели grades есть поле grade_value::integer, но куда девать пропуски? Хранить их в том же поле как NULL или магическое число (например, 0) — плохая практика. Это усложнит запросы и может привести к ошибкам.

Решение: Создать отдельную таблицу grade_types.

Некорректная логика связи "Ученик-Предмет":

Проблема: Связь между Учениками и Предметами на диаграмме показана как напрямую через Оценки. Это верно, но в тексте вы пишете: "Предметы (student_id, Название предмета)". Это серьезная ошибка. Это означало бы, что каждый предмет "принадлежит" конкретному ученику, что бессмысленно. Предметы должны быть независимой сущностью.

Решение: Вы правильно исправили это в логической модели, создав таблицу subjects. Просто убедитесь, что начальное описание исправлено.

Терминология "Secondary Key":

Проблема: В реляционной теории нет стандартного понятия "вторичный ключ" (secondary key). То, что вы описали (student_id, subject_id, date) — это UNIQUE-ограничение или альтернативный ключ (Alternate Key), который гарантирует, что у одного ученика по одному предмету не может быть двух оценок за одну и ту же дату. Это очень хорошая и правильная мысль, но назовите ее верно.

Избыточность в таблице students:

Проблема: Хранение полного ФИО отца и матери в каждой строке ученика не нормализовано. Если у двух детей одни родители, их данные придется дублировать, что может привести к несогласованности.

Решение (на будущее): Для настоящей нормализации можно было бы создать отдельную таблицу parents и таблицу связи student_parents. Однако, для учебной лабораторной работы и учитывая простоту задачи, ваше решение допустимо. Но будьте готовы обосновать этот выбор.

Рекомендации по улучшению
Поле address: varchar(100) может быть слишком мало для полного адреса. Рассмотрите text или varchar(200).

Поле phone_number: Лучше всегда использовать text или varchar для номеров телефонов, так как они могут начинаться с +, иметь пробелы, скобки и т.д. Ваше решение varchar(20) вполне адекватно.

Поле grade_value: Какие оценки допустимы? Только 2, 3, 4, 5? Или также "зачет/незачет"? Рекомендуется использовать CHECK ограничение, например: CHECK (grade_value BETWEEN 2 AND 5).

Исправленная и дополненная модель
Вот переработанный вариант, который устраняет указанные ошибки.

Логическая модель
students (student_id, last_name, first_name, middle_name, address, phone_number, father_last_name, father_first_name, father_middle_name, mother_last_name, mother_first_name, mother_middle_name, mother_phone_number)

Первичный ключ: student_id

subjects (subject_id, subject_name)

Первичный ключ: subject_id

grade_types (type_id, type_name) // НОВАЯ СУЩНОСТЬ

Первичный ключ: type_id

Примеры записей: (1, 'Оценка'), (2, 'Пропуск')

grades (grade_id, student_id, subject_id, type_id, value, date)

Первичный ключ: grade_id

Внешние ключи:

student_id -> students(student_id)

subject_id -> subjects(subject_id)

type_id -> grade_types(type_id)

UNIQUE-ограничение: (student_id, subject_id, date) // Чтобы избежать дублирования записей за день.

Логика:

Если type_id соответствует "Оценка", то поле value содержит цифру (2, 3, 4, 5).

Если type_id соответствует "Пропуск", то поле value может содержать NULL или, например, количество пропущенных уроков (1, 2...). Это зависит от постановки. Часто для пропусков создают отдельную таблицу, но для простоты можно и так.

Физическая модель (DDL для PostgreSQL)
sql
CREATE TABLE students (
    student_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Современный способ автоинкремента
    last_name TEXT NOT NULL,
    first_name TEXT NOT NULL,
    middle_name TEXT,
    address VARCHAR(150),
    phone_number VARCHAR(20),
    father_last_name TEXT,
    father_first_name TEXT,
    father_middle_name TEXT,
    mother_last_name TEXT,
    mother_first_name TEXT,
    mother_middle_name TEXT,
    mother_phone_number VARCHAR(20)
);

CREATE TABLE subjects (
    subject_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE -- Название предмета должно быть уникальным
);

CREATE TABLE grade_types (
    type_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE
);

CREATE TABLE grades (
    grade_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    student_id INTEGER NOT NULL,
    subject_id INTEGER NOT NULL,
    type_id INTEGER NOT NULL,
    value INTEGER, -- Может быть NULL для пропусков, если не храним кол-во уроков
    date DATE NOT NULL DEFAULT CURRENT_DATE,

    -- Внешние ключи
    CONSTRAINT fk_grades_student
        FOREIGN KEY(student_id)
        REFERENCES students(student_id)
        ON DELETE CASCADE,

    CONSTRAINT fk_grades_subject
        FOREIGN KEY(subject_id)
        REFERENCES subjects(subject_id),

    CONSTRAINT fk_grades_grade_type
        FOREIGN KEY(type_id)
        REFERENCES grade_types(type_id),

    -- Ограничение на уникальность оценки ученика по предмету за дату
    CONSTRAINT unique_grade_entry
        UNIQUE (student_id, subject_id, date),


    -- Проверка на значение оценки (только если это запись типа "Оценка")
    -- Это упрощенный вариант. Более строгий потребовал бы триггеров.
    CONSTRAINT chk_grade_value
        CHECK (
            (type_id = (SELECT type_id FROM grade_types WHERE type_name = 'Оценка') AND value BETWEEN 2 AND 5)
            OR
            (type_id = (SELECT type_id FROM grade_types WHERE type_name = 'Пропуск') AND value IS NULL)
            -- Или, если храним кол-во уроков для пропуска: (type_id = ... AND value > 0)
        )
);

-- Вставка базовых типов записей
INSERT INTO grade_types (type_name) VALUES ('Оценка'), ('Пропуск');
Проверка нормальных форм (для исправленной модели)
1NF: Все атрибуты атомарны. Повторяющихся групп нет.

2NF: Все таблицы имеют простой первичный ключ. Нетривиальные функциональные зависимости от составного ключа в grades (value зависит от всего ключа {grade_id}, а date и type_id тоже являются частью ключа или зависят от него). В целом, соответствует.

3NF: Есть транзитивные зависимости в таблице students (например, mother_phone_number функционально зависит от mother_first_name, mother_last_name, которые, в свою очередь, зависят от student_id). Это нарушение 3NF. Как обсуждалось выше, для полной нормализации нужно выносить родителей в отдельную таблицу. Ваша текущая модель не соответствует 3NF, и это ее главный теоретический недостаток.

Итог
Ваша основная ошибка была в отсутствии сущности для типа записи. После ее исправления модель становится работоспособной и логичной для поставленных задач. Нарушение 3NF в таблице студентов является компромиссом между простотой и строгой нормализацией, который на данном этапе может быть допустим.

Рекомендую: представить эту исправленную модель и в пояснительной записке обсудить, что для полного соответствия 3NF таблицу students следует разбить на students и parents.
```


