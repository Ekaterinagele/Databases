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
```sql
-- Справочник типов записей в журнале
CREATE TABLE record_types (
    type_id SERIAL PRIMARY KEY,
    type_name VARCHAR(20) NOT NULL UNIQUE -- Например: 'grade', 'absence'
);

-- Справочник причин пропусков (опционально, для детализации)
CREATE TABLE absence_types (
    absence_type_id SERIAL PRIMARY KEY,
    absence_type_name VARCHAR(50) NOT NULL UNIQUE -- Например: 'Болезнь', 'Семейные обстоятельства'
);

-- Таблица Учеников
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    last_name TEXT NOT NULL,
    first_name TEXT NOT NULL,
    middle_name TEXT,
    address VARCHAR(100),
    phone_number VARCHAR(20),
    -- Данные об отце
    father_last_name TEXT,
    father_first_name TEXT,
    father_middle_name TEXT,
    -- Данные о матери
    mother_last_name TEXT,
    mother_first_name TEXT,
    mother_middle_name TEXT,
    mother_phone_number VARCHAR(20)
);

-- Таблица Предметов
CREATE TABLE subjects (
    subject_id SERIAL PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE
);

-- Основная журнальная таблица (Оценки и Пропуски)
CREATE TABLE journal (
    record_id SERIAL PRIMARY KEY,
    student_id INTEGER NOT NULL,
    subject_id INTEGER NOT NULL,
    date DATE NOT NULL,
    -- Поля, которые могут быть NULL в зависимости от типа записи
    grade_value INTEGER, -- Оценка (2-5). NULL, если запись - это пропуск.
    absence_type_id INTEGER, -- Причина пропуска. NULL, если запись - это оценка.
    -- Тип записи: обязательно указывает, что это - оценка или пропуск.
    record_type_id INTEGER NOT NULL,

    -- Ограничения целостности
    CONSTRAINT fk_journal_student FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    CONSTRAINT fk_journal_subject FOREIGN KEY (subject_id) REFERENCES subjects(subject_id) ON DELETE CASCADE,
    CONSTRAINT fk_journal_record_type FOREIGN KEY (record_type_id) REFERENCES record_types(type_id),
    CONSTRAINT fk_journal_absence_type FOREIGN KEY (absence_type_id) REFERENCES absence_types(absence_type_id),

    -- Проверка на корректность оценки
    CONSTRAINT chk_grade_value CHECK (grade_value IS NULL OR (grade_value >= 2 AND grade_value <= 5)),

    -- Обеспечиваем, что запись не может быть и оценкой, и пропуском одновременно.
    -- Если это оценка (record_type_id соответствует 'grade'), то absence_type_id должен быть NULL.
    -- Если это пропуск (record_type_id соответствует 'absence'), то grade_value должен быть NULL.
);

-- Уникальный индекс, чтобы предотвратить дублирование оценок за один день по одному предмету.
-- Для пропусков это ограничение может не действовать (ученик может болеть несколько дней подряд по одному предмету),
-- но для оценок оно полезно.
CREATE UNIQUE INDEX idx_journal_unique_grade ON journal (student_id, subject_id, date)
WHERE grade_value IS NOT NULL; -- Частичный индекс только для записей с оценками
```

## Проверка нормальных форм: 


