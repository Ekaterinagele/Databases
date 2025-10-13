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


