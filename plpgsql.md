# PL/pgSQL

**PL/pgSQL** (Procedural Language/PostgreSQL Structured Query Language) — это процедурный язык, встроенный в PostgreSQL, который позволяет писать хранимые процедуры и триггеры с использованием управляющих структур (циклы, условия) и логики на стороне базы данных. PL/pgSQL расширяет стандартные SQL-возможности, предоставляя такие инструменты, как переменные, условия и циклы.

**1. Функции (Functions)**
Функции в PL/pgSQL позволяют инкапсулировать логику и многократно её использовать. Функции могут принимать параметры и возвращать значения.

Пример: Функция без параметров

```sql
CREATE OR REPLACE FUNCTION hello_world() RETURNS TEXT AS $$
BEGIN
    RETURN 'Hello, world!';
END;
$$ LANGUAGE plpgsql;
```

Вызов функции:

```sql
SELECT hello_world();
```

Пример: Функция с параметрами

```sql
CREATE OR REPLACE FUNCTION get_employee_salary(emp_id INT) RETURNS DECIMAL AS $$
DECLARE
    emp_salary DECIMAL;
BEGIN
    SELECT salary INTO emp_salary
    FROM employees
    WHERE id = emp_id;

    RETURN emp_salary;
END;
$$ LANGUAGE plpgsql;
```

Вызов функции:

```sql
SELECT get_employee_salary(1);
```

Здесь функция get_employee_salary принимает параметр emp_id и возвращает зарплату сотрудника с указанным идентификатором.

**2. Процедуры (Procedures)**
Процедуры в PostgreSQL (с версии 11) похожи на функции, но они не возвращают значения и могут использовать транзакции (COMMIT/ROLLBACK) внутри себя.

Пример процедуры:

```sql
CREATE OR REPLACE PROCEDURE raise_salary(emp_id INT, raise_percent DECIMAL) AS $$
BEGIN
    UPDATE employees
    SET salary = salary * (1 + raise_percent / 100)
    WHERE id = emp_id;
END;
$$ LANGUAGE plpgsql;
```

Вызов процедуры:

```sql
CALL raise_salary(1, 10); -- Повышение зарплаты на 10% для сотрудника с id = 1
```

**3. Переменные (Variables)**
В PL/pgSQL можно объявлять и использовать переменные для хранения промежуточных значений.

Пример использования переменных:

```sql
CREATE OR REPLACE FUNCTION calculate_bonus(emp_id INT) RETURNS DECIMAL AS $$
DECLARE
    emp_salary DECIMAL;
    bonus DECIMAL;
BEGIN
    -- Получаем зарплату сотрудника
    SELECT salary INTO emp_salary
    FROM employees
    WHERE id = emp_id;

    -- Рассчитываем бонус как 10% от зарплаты
    bonus := emp_salary * 0.1;

    RETURN bonus;
END;
$$ LANGUAGE plpgsql;
```

**4. Условные операторы (IF, ELSE)**
PL/pgSQL поддерживает использование условных операторов для выполнения различных блоков кода в зависимости от условий.

Пример: Условное выполнение кода

```sql
CREATE OR REPLACE FUNCTION check_salary(emp_id INT) RETURNS TEXT AS $$
DECLARE
    emp_salary DECIMAL;
BEGIN
    SELECT salary INTO emp_salary
    FROM employees
    WHERE id = emp_id;

    IF emp_salary > 50000 THEN
        RETURN 'High salary';
    ELSE
        RETURN 'Low salary';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

**5. Циклы (Loops)**
PL/pgSQL поддерживает различные виды циклов, такие как LOOP, WHILE, и FOR.

Пример: Цикл с FOR

```sql
CREATE OR REPLACE FUNCTION increase_salary_for_all() RETURNS VOID AS $$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN SELECT id, salary FROM employees LOOP
        UPDATE employees
        SET salary = salary * 1.05
        WHERE id = rec.id;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

Этот пример увеличивает зарплату всех сотрудников на 5%.

**6. Триггеры (Triggers)**
Триггеры — это специальные функции, которые автоматически выполняются при вставке, обновлении или удалении данных в таблицах.

Пример: Триггер для обновления даты изменения
Создадим триггер для таблицы employees, который будет автоматически обновлять столбец last_updated при изменении зарплаты сотрудника:

Сначала добавим столбец для хранения даты последнего обновления:

```sql
ALTER TABLE employees ADD COLUMN last_updated TIMESTAMP;
```

Теперь создадим функцию-триггер:

```sql
CREATE OR REPLACE FUNCTION update_last_modified() RETURNS TRIGGER AS $$
BEGIN
    NEW.last_updated := CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

Далее создадим сам триггер:

```sql
CREATE TRIGGER trigger_update_last_modified
BEFORE UPDATE ON employees
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary)
EXECUTE FUNCTION update_last_modified();
```

Теперь, когда зарплата сотрудника изменяется, поле last_updated будет автоматически обновлено текущей датой и временем.

**7. Обработка ошибок (Exception Handling)**
PL/pgSQL поддерживает обработку ошибок с помощью блока EXCEPTION. Это позволяет перехватывать и обрабатывать ошибки, возникающие во время выполнения процедуры или функции.

Пример: Обработка деления на ноль

```sql
CREATE OR REPLACE FUNCTION safe_divide(a DECIMAL, b DECIMAL) RETURNS DECIMAL AS $$
DECLARE
    result DECIMAL;
BEGIN
    result := a / b;
    RETURN result;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Cannot divide by zero';
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

Этот пример демонстрирует, как можно перехватить ошибку деления на ноль и вернуть NULL вместо аварийного завершения.

**8. Пакетная обработка данных (Cursors)**
Курсоры позволяют обрабатывать большие наборы данных по частям, что полезно, если данные слишком большие для единовременной обработки.

Пример: Использование курсора

```sql
CREATE OR REPLACE FUNCTION process_large_dataset() RETURNS VOID AS $$
DECLARE
    emp_cursor CURSOR FOR SELECT id, name FROM employees;
    emp_record RECORD;
BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO emp_record;
        EXIT WHEN NOT FOUND;

        -- Здесь можно обработать запись, например, вывести данные
        RAISE NOTICE 'Processing employee: %', emp_record.name;
    END LOOP;

    CLOSE emp_cursor;
END;
$$ LANGUAGE plpgsql;
```

Этот пример показывает использование курсора для поочередной обработки каждой строки из таблицы employees.
