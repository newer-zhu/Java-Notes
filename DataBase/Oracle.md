1. 一个用户相当于一个数据库

2. 表空间

   表空间是用于存储表、索引和其他数据库对象的逻辑存储结构。每个表空间由一个或多个数据文件组成，这些文件可以位于不同的物理磁盘上。

   ```sql
   CREATE TABLESPACE example_tbs
   DATAFILE '/u01/app/oracle/oradata/example/example01.dbf'
   SIZE 100M
   AUTOEXTEND ON
   NEXT 50M
   MAXSIZE 500M;
   ```

3. 修改列名的语法

   ```sql
   oracle
   ALTER TABLE table_name RENAME COLUMN old_column_name TO new_column_name;
   mysql
   ALTER TABLE table_name CHANGE COLUMN old_column_name new_column_name datatype;
   ```

4. sysydate 表示当前日期

5. 在Oracle数据库中，ROWID 是一个伪列，用于标识表中的行。每个 ROWID 可以用于快速定位表中的行。

   ROWID 值是一个字符串，由以下三个部分组成：

   数据对象的编号（Data Object Number，也称为文件标识符）
   数据块的编号（Data Block Number）
   在数据块中的行号（Row Number）
   例如，以下 ROWID 值表示 employees 表中的一行：

   ```sql
   AAAR3zAAEAAAAVvAAA
   ```

   其中，前六个字符 AAAR3z 是数据对象的编号，接下来的六个字符 AAEAAA 是数据块的编号，最后的三个字符 VVAAA 是在数据块中的行号。

   ROWID 值是基于物理存储结构计算的，并且可能会因为表的重建、分区或其他操作而发生变化。因此，在使用 ROWID 时需要格外小心，并且尽可能使用其他方法来定位表中的行。

6. 在 Oracle 数据库中，ROWNUM 是一个伪列，用于给查询结果中的行分配行号。ROWNUM 的值是一个整数，从 1 开始递增，表示查询结果中的第几行。

   以下是使用 ROWNUM 的基本语法：

   ```sql
   SELECT ROWNUM, employee_id, first_name, last_name
   FROM employees
   WHERE ROWNUM <= 10;
   ```

   返回名为 employees 表中前 10 行数据，并为每行分配一个行号

   需要注意的是，ROWNUM 是在查询结果返回之后才分配的行号，因此不能在查询过程中使用它来筛选数据。例如，以下查询不会返回任何数据：

   ```sql
   SELECT ROWNUM, employee_id, first_name, last_name
   FROM employees
   WHERE ROWNUM > 10;
   ```


   这是因为 ROWNUM 的值在结果返回之前就已经被计算了，而 WHERE 子句只能筛选已经返回的结果。如果要筛选结果集中的行号，可以使用子查询或通用表表达式（CTE）等方法。

   ```sql
   第 11 到 20 行数据
   SELECT *
   FROM (
       SELECT t.*, ROWNUM rn
       FROM my_table t
       WHERE ROWNUM <= 20
   )
   WHERE rn >= 11;
   按照id升序排序后获取第 11 到 20 行数据
   SELECT *
   FROM (
       SELECT t.*, ROWNUM rn
       FROM (
           SELECT *
           FROM my_table
           ORDER BY id
       ) t
       WHERE ROWNUM <= 20
   )
   WHERE rn >= 11;
   
   ```

7. NVL 函数接受两个参数，如果第一个参数不为空，则返回第一个参数的值；否则返回第二个参数的值。

   ```sql
   SELECT NVL(name, 'N/A') AS name, NVL(age, 0) AS age
   FROM my_table;
   ```

   NVL2 函数接受三个参数，如果第一个参数不为空，则返回第二个参数的值；否则返回第三个参数的值。

   ```sql
   SELECT NVL2(name, 'Name is not empty', 'Name is empty') AS result
   FROM my_table;
   ```

8. DECODE 函数接受多个参数，第一个参数是要比较的字段或表达式，后面的参数是一组值和对应的结果。如果要比较的字段或表达式等于某个值，则返回该值对应的结果；否则返回最后一个参数作为默认结果。

   ```sql
   SELECT DECODE(gender, 'M', 'Male', 'F', 'Female', 'Unknown') AS result
   FROM my_table;
   ===============
   如果 gender 字段等于 'M'，则返回字符串 "Male"；如果 gender 字段等于 'F'，则返回字符串 "Female"；否则返回字符串 "Unknown"。
   ```

9. CASE 表达式根据 gender 字段的值，在查询结果中添加一个名为 gender_desc 的新列。如果 gender 字段等于 'M'，则返回字符串 "Male"；如果 gender 字段等于 'F'，则返回字符串 "Female"；否则返回字符串 "Unknown"。

```sql
SELECT name, age, gender,
    CASE gender
        WHEN 'M' THEN 'Male'
        WHEN 'F' THEN 'Female'
        ELSE 'Unknown'
    END AS gender_desc
FROM my_table;
```

10. ```sql
    SELECT name, score, RANK() OVER (ORDER BY score DESC) AS rank
    FROM my_table;
    ```

    RANK() 函数根据 score 字段的值为每个行分配一个排名，并将排名保存在名为 rank 的新列中。ORDER BY 子句指定了按照 score 字段的值降序排序。如果有多个行具有相同的值，则它们将被分配相同的排名，并且下一个排名将被跳过。dense_rank()则连续，row_number()连续排名无论值是否相同。

11. 视图

    ```sql
    CREATE [OR REPLACE] VIEW view_name [(column1, column2, ...)]
    AS SELECT statement;
    ```

    CREATE VIEW 用于创建一个新的视图，OR REPLACE 用于替换一个已经存在的同名视图（如果存在），view_name 是视图的名称，column1, column2, ... 是视图中包含的列名，SELECT statement 是用于创建视图的查询语句。

    需要注意的是，在 Oracle 数据库中，视图可以包含复杂的查询和窗口函数等高级特性，而在 MySQL 数据库中，视图的功能相对较为简单。此外，在 Oracle 数据库中，视图可以使用 WITH CHECK OPTION 子句来限制对视图的修改操作；而在 MySQL 数据库中，视图不支持 WITH CHECK OPTION 子句。

    视图和表的数据是相互绑定的。修改视图数据时只会影响键保留表里的数据，也就是主键来自的那张表。

12. 物化视图：物化视图是一个包含了预先计算好的数据的表，这些数据可以是聚合数据、连接查询或者其他复杂查询的结果。当查询需要访问这些数据时，数据库可以直接从物化视图中读取数据，而不必重新计算查询结果。

    ```sql
    CREATE MATERIALIZED VIEW view_name
    BUILD [IMMEDIATE|DEFERRED]
    REFRESH [FAST|COMPLETE|FORCE] [START WITH date] [NEXT date]
    [on commit]
    AS SELECT statement
    [LOGGING;]
    
    手动刷新
    begin
    EXEC DBMS_SNAPSHOT.REFRESH('my_view', 'C');
    end;
    ```

    CREATE MATERIALIZED VIEW 用于创建一个新的物化视图，BUILD IMMEDIATE 或 BUILD DEFERRED 用于指定何时开始构建物化视图，REFRESH FAST、REFRESH COMPLETE 或 REFRESH FORCE 用于指定何时刷新物化视图，START WITH 和 NEXT 用于指定刷新计划，view_name 是物化视图的名称，SELECT statement 是用于创建物化视图的查询语句。LOGGING 选项表示启用日志记录功能，用于记录增量数据

    在 MySQL 数据库中，创建物化视图的语法与普通视图类似，但是 MySQL 不支持像 Oracle 那样的 REFRESH 选项和 START WITH/NEXT 选项。

13. 序列：

    ```sql
    CREATE SEQUENCE my_seq
    START WITH 1
    INCREMENT BY 1
    MAXVALUE 999999999
    NOCACHE
    NOORDER;
    ```

    my_seq 是序列的名称，START WITH 1 表示起始值为 1，INCREMENT BY 1 表示递增量为 1，MAXVALUE 999999999 表示最大值为 999999999，NOCACHE 表示不缓存序列值，NOORDER 表示不保证序列值的顺序。

    需要注意的是，序列是数据库级别的对象，可以被多个表共享。

14. 在MySQL中，用户定义变量只在当前会话中有效。而在Oracle中，PL/SQL变量的作用域可以是整个程序或子程序。

    ```sql
    --mysql
    SET @my_var = 10;
    SELECT CONCAT('My variable is ', @my_var);
    
    --oracle
    DECLARE
      my_var NUMBER;
    BEGIN
      my_var := 10;
      DBMS_OUTPUT.PUT_LINE('My variable is ' || my_var);
    END;
    --如果不知道v_last_name数据类型但知道其来源列
    DECLARE
      v_last_name employees.last_name%TYPE;
    BEGIN
      SELECT last_name INTO v_last_name FROM employees WHERE employee_id = 100;
      DBMS_OUTPUT.PUT_LINE('The last name is ' || v_last_name);
    END;
    --如果不知道v_employee数据类型但知道其来源行
    DECLARE
      v_employee employees%ROWTYPE;
    BEGIN
      SELECT * INTO v_employee FROM employees WHERE employee_id = 100;
      DBMS_OUTPUT.PUT_LINE('The employee name is ' || v_employee.first_name || ' ' || v_employee.last_name);
    END;
    
    ```

15. 异常处理

    ```plsql
    BEGIN
      -- 此处是正常的 PL/SQL 代码
      ...
    EXCEPTION
      WHEN exception1 THEN
        -- 异常处理代码1
        ...
      WHEN exception2 THEN
        -- 异常处理代码2
        ...
    END;
    ============================================================
    BEGIN
      SELECT column1 INTO variable1 FROM table1 WHERE column2 = value;
    
    EXCEPTION
      WHEN NO_DATA_FOUND THEN
        -- 处理 No Data Found 异常
        variable1 := NULL;
    END;
    ```

16. 循环

    ```sql
    --oracle
    FOR i IN 1..10 LOOP
      -- 执行循环体中的代码
    END LOOP;
    
    WHILE condition LOOP
      -- 执行循环体中的代码
    END LOOP;
    ```

17. 分区，可以把表根据某种规则分区，插入数据时无感知。查询可以根据分区的规则查询

    ```sql
    CREATE TABLE SALES
    (
      SALES_ID NUMBER,
      SALE_DATE DATE,
      CUSTOMER_ID NUMBER,
      PRODUCT_ID NUMBER,
      AMOUNT NUMBER
    )
    PARTITION BY RANGE (SALE_DATE)
    (
      PARTITION SALES_Q1 VALUES LESS THAN (TO_DATE('01-APR-2023', 'DD-MON-YYYY')),
      PARTITION SALES_Q2 VALUES LESS THAN (TO_DATE('01-JUL-2023', 'DD-MON-YYYY')),
      PARTITION SALES_Q3 VALUES LESS THAN (TO_DATE('01-OCT-2023', 'DD-MON-YYYY')),
      PARTITION SALES_Q4 VALUES LESS THAN (TO_DATE('01-JAN-2024', 'DD-MON-YYYY'))
    );
    
    SELECT * FROM SALES PARTITION (SALES_Q1);
    SELECT * FROM SALES PARTITION (SALES_Q2);
    SELECT * FROM SALES PARTITION (SALES_Q3);
    SELECT * FROM SALES PARTITION (SALES_Q4);
    ```

    