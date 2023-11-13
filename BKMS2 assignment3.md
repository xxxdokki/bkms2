# BKMS2 assignment3

복습 여부: No

김도윤

2023-22378

- ChatGPT 4 사용
- https://github.com/xxxdokki/bkms2/blob/fbaa8db3d5d6fecb1d429917d7a3a37bbfd8b77e/BKMS2%20assignment3.md -> 권장
- [https://www.notion.so/BKMS2-assignment3-8386851b79f748e8a87597312a4ccfa4?pvs=4](https://www.notion.so/BKMS2-assignment3-8386851b79f748e8a87597312a4ccfa4?pvs=21) 에서 더 간단히 살펴볼 수 있음.

### 첫 번째 문제

- Find the department that have the highest average salary
    1. `Ask ChatGPT to generate a SQL query **without giving any schema information**.`
        
        제시한 프롬프트는 다음과 같음.
- 프롬프트 시작 부분
        
        > ****1. Creating Tables****
        Tables are the basic unit of data storage in an Oracle Database. Data is stored in rows and columns. You define a table with a table name, such as employees, and a set of columns. You give each column a column name, such as employee_id, last_name, and job_id; a datatype, such as VARCHAR2, DATE, or NUMBER; and a width. The width can be predetermined by the datatype, as in DATE. If columns are of the NUMBER datatype, define precision and scale instead of width. A row is a collection of column information corresponding to a single record.
        You can specify rules for each column of a table. These rules are called integrity constraints. One example is a NOT NULL integrity constraint. This constraint forces the column to contain a value in every row.
        For example:
        `create table DEPARTMENTS (  
          deptno        number,  
          name          varchar2(50) not null,  
          location      varchar2(50),  
          constraint pk_departments primary key (deptno)  
        );`
         Insert into EditorCode Block 1
        
        Tables can declarative specify relationships between tables, typically referred to as referential integrity. To see how this works we can create a "child" table of the DEPARTMENTS table by including a foreign key in the EMPLOYEES table that references the DEPARTMENTS table. For example:
        `create table EMPLOYEES (  
          empno             number,  
          name              varchar2(50) not null,  
          job               varchar2(50),  
          manager           number,  
          hiredate          date,  
          salary            number(7,2),  
          commission        number(7,2),  
          deptno           number,  
          constraint pk_employees primary key (empno),  
          constraint fk_employees_deptno foreign key (deptno) 
              references DEPARTMENTS (deptno)  
        );`
         Insert into EditorCode Block 2
        
        Foreign keys must reference primary keys, so to create a "child" table the "parent" table must have a primary key for the foreign key to reference.
        ****2. Creating Triggers****
        Triggers are procedures that are stored in the database and are implicitly run, or fired, when something happens. Traditionally, triggers supported the execution of a procedural code, in Oracle procedural SQL is called a PL/SQL block. PL stands for procedural language. When an INSERT, UPDATE, or DELETE occurred on a table or view. Triggers support system and other data events on DATABASE and SCHEMA.
        Triggers are frequently used to automatically populate table primary keys, the trigger examples below show an example trigger to do just this. We will use a built in function to obtain a globallally unique identifier or GUID.
        `create or replace trigger  DEPARTMENTS_BIU
            before insert or update on DEPARTMENTS
            for each row
        begin
            if inserting and :new.deptno is null then
                :new.deptno := to_number(sys_guid(), 
                  'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /
        
        create or replace trigger EMPLOYEES_BIU
            before insert or update on EMPLOYEES
            for each row
        begin
            if inserting and :new.empno is null then
                :new.empno := to_number(sys_guid(), 
                    'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /`
         Insert into EditorCode Block 3
        
        ****3. Inserting Data****
        Now that we have tables created, and we have triggers to automatically populate our primary keys, we can add data to our tables. Because we have a parent child relationship, with the DEPARTMENTS table as the parent table, and the EMPLOYEES table as the child we will first INSERT a row into the DEPARTMENTS table.
        `insert into departments (name, location) values
           ('Finance','New York');
        
        insert into departments (name, location) values
           ('Development','San Jose');`
         Insert into EditorCode Block 4
        
        Lets verify that the insert was successful by running a SQL SELECT statement to query all columns and all rows of our table.
        `select * from departments;`
         Insert into EditorCode Block 5
        
        You can see that an ID will have been automatically generated. You can now insert into the EMPLOYEES table a new row but you will need to put the generated DEPTID value into your SQL INSERT statement. The examples below show how we can do this using a SQL query, but you could simply enter the department number directly.
        `insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Sam Smith','Programmer', 
            5000, 
          (select deptno 
          from departments 
          where name = 'Development'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Mara Martin','Analyst', 
           6000, 
           (select deptno 
           from departments 
           where name = 'Finance'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Yun Yates','Analyst', 
           5500, 
           (select deptno 
           from departments 
           where name = 'Development'));`
         Insert into EditorCode Block 6
        
        ****4. Indexing Columns****
        Typically developers index columns for three major reasons:
        1. To enforce unique values within a column
        2. To improve data access performance
        3. To prevent lock escalation when updating rows of tables that use declarative referential integrity
        When a table is created and a PRIMARY KEY is specified an index is automatically created to enforce the primary key constraint. If you specific UNIQUE for a column when creating a column a unique index is also created. To see the indexes that already exist for a given table you can run the following dictionary query.
        `select table_name "Table", 
               index_name "Index", 
               column_name "Column", 
               column_position "Position"
        from  user_ind_columns 
        where table_name = 'EMPLOYEES' or 
              table_name = 'DEPARTMENTS'
        order by table_name, column_name, column_position`
         Insert into EditorCode Block 7
        
        It is typically good form to index foreign keys, foreign keys are columns in a table that reference another table. In our EMPLOYEES and DEPARTMENTS table example the DEPTNO column in the EMPLOYEE table references the primary key of the DEPARTMENTS table.
        `create index employee_dept_no_fk_idx 
        on employees (deptno)`
         Insert into EditorCode Block 8
        
        We may also determine that the EMPLOYEE table will be frequently searched by the NAME column. To improve the performance searches and to ensure uniqueness we can create a unique index on the EMPLOYEE table NAME column.
        `create unique index employee_ename_idx
        on employees (name)`
         Insert into EditorCode Block 9
        
        Oracle provides many other indexing technologies including function based indexes which can index expressions, such as an upper function, text indexes which can index free form text, bitmapped indexes useful in data warehousing. You can also create indexed organized tables, you can use partition indexes and more. Sometimes it is best to have fewer indexes and take advantage of in memory capabilities. All of these topics are beyond the scope of this basic introduction.
        ****5. Querying Data****
        To select data from a single table it is reasonably easy, simply use the SELECT ... FROM ... WHERE ... ORDER BY ... syntax.
        `select * from employees;`
         Insert into EditorCode Block 10
        
        To query data from two related tables you can join the data
        `select e.name employee,
                   d.name department,
                   e.job,
                   d.location
        from departments d, employees e
        where d.deptno = e.deptno(+)
        order by e.name;`
         Insert into EditorCode Block 11
        
        As an alternative to a join you can use an inline select to query data.
        `select e.name employee,
                  (select name 
                   from departments d 
                   where d.deptno = e.deptno) department,
                   e.job
        from employees e
        order by e.name;`
         Insert into EditorCode Block 12
        
        ****6. Adding Columns****
        You can add additional columns after you have created your table using the ALTER TABLE ... ADD ... syntax. For example:
        `alter table EMPLOYEES 
        add country_code varchar2(2);`
         Insert into EditorCode Block 13
        
        ****7. Querying the Oracle Data Dictionary****
        Table meta data is accessible from the Oracle data dictionary. The following queries show how you can query the data dictionary tables.
        `select table_name, tablespace_name, status
        from user_tables
        where table_Name = 'EMPLOYEES';
        
        select column_id, column_name , data_type
        from user_tab_columns
        where table_Name = 'EMPLOYEES'
        order by column_id;`
         Insert into EditorCode Block 14
        
        ****8. Updating Data****
        You can use SQL to update values in your table, to do this we will use the update clause
        `update employees
        set country_code = 'US';`
         Insert into EditorCode Block 15
        
        The query above will update all rows of the employee table and set the value of country code to US. You can also selectively update just a specific row.
        `update employees
        set commission = 2000
        where  name = 'Sam Smith';`
         Insert into EditorCode Block 16
        
        Lets run a Query to see what our data looks like
        `select name, country_code, salary, commission
        from employees
        order by name;`
         Insert into EditorCode Block 17
        
        ****9. Aggregate Queries****
        You can sum data in tables using aggregate functions. We will use column aliases to rename columns for readability, we will also use the null value function (NVL) to allow us to properly sum columns with null values.
        `select 
              count(*) employee_count,
              sum(salary) total_salary,
              sum(commission) total_commission,
              min(salary + nvl(commission,0)) min_compensation,
              max(salary + nvl(commission,0)) max_compensation
        from employees;`
         Insert into EditorCode Block 18
        
        ****10. Compressing Data****
        As your database grows in size to gigabytes or terabytes and beyond, consider using table compression. Table compression saves disk space and reduces memory use in the buffer cache. Table compression can also speed up query execution during reads. There is, however, a cost in CPU overhead for data loading and DML. Table compression is completely transparent to applications. It is especially useful in online analytical processing (OLAP) systems, where there are lengthy read-only operations, but can also be used in online transaction processing (OLTP) systems.
        You specify table compression with the COMPRESS clause of the CREATE TABLE statement. You can enable compression for an existing table by using this clause in an ALTER TABLE statement. In this case, the only data that is compressed is the data inserted or updated after compression is enabled. Similarly, you can disable table compression for an existing compressed table with the ALTER TABLE...NOCOMPRESS statement. In this case, all data the was already compressed remains compressed, and new data is inserted uncompressed.
        To enable compression for future data use the following syntax.
        `alter table EMPLOYEES compress for oltp; 
        alter table DEPARTMENTS compress for oltp;` 
         Insert into EditorCode Block 19
        
        ****11. Deleting Data****
        You can delete one or more rows from a table using the DELETE syntax. For example to delete a specific row:
        `delete from employees 
        where name = 'Sam Smith';`
         Insert into EditorCode Block 20
        
        ****12. Dropping Tables****
        You can drop tables using the SQL DROP command. Dropping a table will remove all of the rows and drop sub-objects including indexes and triggers. The following DROP statements will drop the departments and employees tables. The optional cascade constraints clause will drop remove constraints thus allowing you to drop database tables in any order.
        `drop table departments cascade constraints;
        drop table employees cascade constraints;`
         Insert into EditorCode Block 21
        
        ****13. Un-dropping Tables****
        If the RECYCLEBIN initialization parameter is set to ON (the default in 10g), then dropping this table will place it in the recycle bin. To see if you can undrop a table run the following data dictionary query:
        `select object_name, 
               original_name, 
               type, 
               can_undrop, 
               can_purge
        from recyclebin;`
         Insert into EditorCode Block 22
        
        To undrop tables we use the flashback command, for example:
        `flashback table DEPARTMENTS to before drop;
        flashback table EMPLOYEES to before drop;
        select count(*) departments 
        from departments;
        select count(*) employees
        from employees;`
         Insert into EditorCode Block 23
        > 
        > 
        > 
        > -> 자 이제  "Find the department that has the highest average salary" 문제를 해결하는 SQL 쿼리를 생성해줘
        >
  
- 프롬프트 끝

  
- 결과
        
        SELECT d.name AS department_name, AVG(e.salary) AS average_salary
        FROM departments d
        INNER JOIN employees e ON d.deptno = e.deptno
        GROUP BY d.name
        ORDER BY AVG(e.salary) DESC
        FETCH FIRST 1 ROW ONLY;
        

---

2. `Ask ChatGPT to generate a SQL query **with schema information**.`

- 프롬프트 시작

  
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > create table classroom
        > (building		varchar(15),
        > room_number		varchar(7),
        > capacity		numeric(4,0),
        > primary key (building, room_number)
        > );
        > 
        > create table department
        > (dept_name		varchar(20),
        > building		varchar(15),
        > budget		        numeric(12,2) check (budget > 0),
        > primary key (dept_name)
        > );
        > 
        > create table course
        > (course_id		varchar(8),
        > title			varchar(50),
        > dept_name		varchar(20),
        > credits		numeric(2,0) check (credits > 0),
        > primary key (course_id),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table instructor
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > salary			numeric(8,2) check (salary > 29000),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table section
        > (course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6)
        > check (semester in ('Fall', 'Winter', 'Spring', 'Summer')),
        > year			numeric(4,0) check (year > 1701 and year < 2100),
        > building		varchar(15),
        > room_number		varchar(7),
        > time_slot_id		varchar(4),
        > primary key (course_id, sec_id, semester, year),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (building, room_number) references classroom (building, room_number)
        > on delete set null
        > );
        > 
        > create table teaches
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references instructor (ID)
        > on delete cascade
        > );
        > 
        > create table student
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > tot_cred		numeric(3,0) check (tot_cred >= 0),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table takes
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > grade		        varchar(2),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table advisor
        > (s_ID			varchar(5),
        > i_ID			varchar(5),
        > primary key (s_ID),
        > foreign key (i_ID) references instructor (ID)
        > on delete set null,
        > foreign key (s_ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table time_slot
        > (time_slot_id		varchar(4),
        > day			varchar(1),
        > start_hr		numeric(2) check (start_hr >= 0 and start_hr < 24),
        > start_min		numeric(2) check (start_min >= 0 and start_min < 60),
        > end_hr			numeric(2) check (end_hr >= 0 and end_hr < 24),
        > end_min		numeric(2) check (end_min >= 0 and end_min < 60),
        > primary key (time_slot_id, day, start_hr, start_min)
        > );
        > 
        > create table prereq
        > (course_id		varchar(8),
        > prereq_id		varchar(8),
        > primary key (course_id, prereq_id),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (prereq_id) references course (course_id)
        > ); 
        > 
        > step-by-step의 생각 과정은 다음과 같아. 
        > 
        > 1. **목적 파악:** 우선 "가장 높은 평균 급여를 가진 부서 찾기"라는 문제의 해결 목적을 명확히 합니다. 이 경우에는 '부서별 평균 급여 계산'이라는 중간 목표를 가지고 '가장 높은 급여를 제공하는 부서'를 최종적으로 찾아야 합니다.
        > 2. **데이터 선택:** 관련 데이터를 포함하고 있는 테이블과 필요한 열(컬럼)을 식별합니다. 
        > 3. **집계 함수 사용:** 평균 급여를 계산하기 위해 SQL의 집계 함수 **`AVG`**를 사용합니다. **`AVG(salary)`** 함수는 각 부서의 평균 급여를 계산합니다.
        > 4. **그룹화:** **`GROUP BY`** 절을 사용하여 부서별로 결과를 그룹화하고, 각 부서의 평균 급여를 계산합니다. 이러한 그룹화는 집계 함수가 적용되는 범위를 부서별로 한정짓습니다.
        > 5. **결과 정렬:** 계산된 평균 급여를 기준으로 결과를 정렬하기 위해 **`ORDER BY`** 절을 사용합니다. 평균 급여가 가장 높은 부서를 찾기 위해 평균 급여를 내림차순으로 정렬합니다(**`DESC`**).
        > 6. **상위 결과 제한:** **`FETCH FIRST 1 ROW ONLY`** (또는 일부 SQL 방언에서는 **`LIMIT 1`**)를 사용하여 결과 집합의 첫 번째 행만 가져오도록 제한합니다. 이렇게 하면 가장 높은 평균 급여를 가진 단일 부서만 결과로 반환됩니다.
        > 
        > → 자 이제  "Find the department that has the highest average salary" 문제를 해결하는 SQL 쿼리를 생성해줘
        > 

- 프롬프트 종료

- 결과
    
        SELECT d.dept_name, AVG(i.salary) as avg_salary
        FROM instructor i
        JOIN department d ON i.dept_name = d.dept_name
        GROUP BY d.dept_name
        ORDER BY avg_salary DESC
        FETCH FIRST ROW ONLY;
    

---

3.  `Give **some additional information**(i.e., explanation of table schema, PK/FK relationship, clause you want to use etc.) to ChatGPT to make the SQL query more accurate.`

- 프롬프트 시작

        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > create table classroom
        > (building		varchar(15),
        > room_number		varchar(7),
        > capacity		numeric(4,0),
        > primary key (building, room_number)
        > );
        > 
        > create table department
        > (dept_name		varchar(20),
        > building		varchar(15),
        > budget		        numeric(12,2) check (budget > 0),
        > primary key (dept_name)
        > );
        > 
        > create table course
        > (course_id		varchar(8),
        > title			varchar(50),
        > dept_name		varchar(20),
        > credits		numeric(2,0) check (credits > 0),
        > primary key (course_id),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table instructor
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > salary			numeric(8,2) check (salary > 29000),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table section
        > (course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6)
        > check (semester in ('Fall', 'Winter', 'Spring', 'Summer')),
        > year			numeric(4,0) check (year > 1701 and year < 2100),
        > building		varchar(15),
        > room_number		varchar(7),
        > time_slot_id		varchar(4),
        > primary key (course_id, sec_id, semester, year),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (building, room_number) references classroom (building, room_number)
        > on delete set null
        > );
        > 
        > create table teaches
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references instructor (ID)
        > on delete cascade
        > );
        > 
        > create table student
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > tot_cred		numeric(3,0) check (tot_cred >= 0),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table takes
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > grade		        varchar(2),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table advisor
        > (s_ID			varchar(5),
        > i_ID			varchar(5),
        > primary key (s_ID),
        > foreign key (i_ID) references instructor (ID)
        > on delete set null,
        > foreign key (s_ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table time_slot
        > (time_slot_id		varchar(4),
        > day			varchar(1),
        > start_hr		numeric(2) check (start_hr >= 0 and start_hr < 24),
        > start_min		numeric(2) check (start_min >= 0 and start_min < 60),
        > end_hr			numeric(2) check (end_hr >= 0 and end_hr < 24),
        > end_min		numeric(2) check (end_min >= 0 and end_min < 60),
        > primary key (time_slot_id, day, start_hr, start_min)
        > );
        > 
        > create table prereq
        > (course_id		varchar(8),
        > prereq_id		varchar(8),
        > primary key (course_id, prereq_id),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (prereq_id) references course (course_id)
        > ); 
        > 
        > step-by-step의 생각 과정은 다음과 같아. 
        > 
        > 1. **목적 파악:** 우선 "가장 높은 평균 급여를 가진 부서 찾기"라는 문제의 해결 목적을 명확히 합니다. 이 경우에는 '부서별 평균 급여 계산'이라는 중간 목표를 가지고 '가장 높은 급여를 제공하는 부서'를 최종적으로 찾아야 합니다.
        > 2. **데이터 선택:** 관련 데이터를 포함하고 있는 테이블과 필요한 열(컬럼)을 식별합니다. 
        > 3. **집계 함수 사용:** 평균 급여를 계산하기 위해 SQL의 집계 함수 **`AVG`**를 사용합니다. **`AVG(salary)`** 함수는 각 부서의 평균 급여를 계산합니다.
        > 4. **그룹화:** **`GROUP BY`** 절을 사용하여 부서별로 결과를 그룹화하고, 각 부서의 평균 급여를 계산합니다. 이러한 그룹화는 집계 함수가 적용되는 범위를 부서별로 한정짓습니다.
        > 5. **결과 정렬:** 계산된 평균 급여를 기준으로 결과를 정렬하기 위해 **`ORDER BY`** 절을 사용합니다. 평균 급여가 가장 높은 부서를 찾기 위해 평균 급여를 내림차순으로 정렬합니다(**`DESC`**).
        > 6. **상위 결과 제한:** **`FETCH FIRST 1 ROW ONLY`** (또는 일부 SQL 방언에서는 **`LIMIT 1`**)를 사용하여 결과 집합의 첫 번째 행만 가져오도록 제한합니다. 이렇게 하면 가장 높은 평균 급여를 가진 단일 부서만 결과로 반환됩니다.
        > 
        > → 자 이제  "Find the department that has the highest average salary" 문제를 해결하는 SQL 쿼리를 생성해줘
        > 

- 프롬프트 끝

  
- 결과
    
        -- Step 1: Calculate the average salary for each department
        SELECT dept_name, AVG(salary) AS avg_salary
        FROM instructor
        GROUP BY dept_name
        
        -- Step 2: Order the results by the average salary in descending order and limit to the top result
        ORDER BY avg_salary DESC
        FETCH FIRST 1 ROW ONLY;


    
    ***분석***
    
    <aside>
    💡 분석
    
    1. **평균 급여 계산**: 각 부서별로 'instructor' 테이블의 급여 평균을 'dept_name'으로 그룹화하여 계산합니다.
    2. **결과 정렬 및 제한**: 계산된 평균 급여를 기준으로 내림차순으로 정렬하고, 상위 결과 하나만 가져옵니다.
    </aside>




    

### 두 번째 문제

- Find the average total credits over all prior years.
    1. `Ask ChatGPT to generate a SQL query **without giving any schema information**.`
        
        제시한 쿼리는 다음과 같음.
- 프롬프트 시작

  
        > ****1. Creating Tables****
        Tables are the basic unit of data storage in an Oracle Database. Data is stored in rows and columns. You define a table with a table name, such as employees, and a set of columns. You give each column a column name, such as employee_id, last_name, and job_id; a datatype, such as VARCHAR2, DATE, or NUMBER; and a width. The width can be predetermined by the datatype, as in DATE. If columns are of the NUMBER datatype, define precision and scale instead of width. A row is a collection of column information corresponding to a single record.
        You can specify rules for each column of a table. These rules are called integrity constraints. One example is a NOT NULL integrity constraint. This constraint forces the column to contain a value in every row.
        For example:
        `create table DEPARTMENTS (  
          deptno        number,  
          name          varchar2(50) not null,  
          location      varchar2(50),  
          constraint pk_departments primary key (deptno)  
        );`
         Insert into EditorCode Block 1
        
        Tables can declarative specify relationships between tables, typically referred to as referential integrity. To see how this works we can create a "child" table of the DEPARTMENTS table by including a foreign key in the EMPLOYEES table that references the DEPARTMENTS table. For example:
        `create table EMPLOYEES (  
          empno             number,  
          name              varchar2(50) not null,  
          job               varchar2(50),  
          manager           number,  
          hiredate          date,  
          salary            number(7,2),  
          commission        number(7,2),  
          deptno           number,  
          constraint pk_employees primary key (empno),  
          constraint fk_employees_deptno foreign key (deptno) 
              references DEPARTMENTS (deptno)  
        );`
         Insert into EditorCode Block 2
        
        Foreign keys must reference primary keys, so to create a "child" table the "parent" table must have a primary key for the foreign key to reference.
        ****2. Creating Triggers****
        Triggers are procedures that are stored in the database and are implicitly run, or fired, when something happens. Traditionally, triggers supported the execution of a procedural code, in Oracle procedural SQL is called a PL/SQL block. PL stands for procedural language. When an INSERT, UPDATE, or DELETE occurred on a table or view. Triggers support system and other data events on DATABASE and SCHEMA.
        Triggers are frequently used to automatically populate table primary keys, the trigger examples below show an example trigger to do just this. We will use a built in function to obtain a globallally unique identifier or GUID.
        `create or replace trigger  DEPARTMENTS_BIU
            before insert or update on DEPARTMENTS
            for each row
        begin
            if inserting and :new.deptno is null then
                :new.deptno := to_number(sys_guid(), 
                  'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /
        
        create or replace trigger EMPLOYEES_BIU
            before insert or update on EMPLOYEES
            for each row
        begin
            if inserting and :new.empno is null then
                :new.empno := to_number(sys_guid(), 
                    'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /`
         Insert into EditorCode Block 3
        
        ****3. Inserting Data****
        Now that we have tables created, and we have triggers to automatically populate our primary keys, we can add data to our tables. Because we have a parent child relationship, with the DEPARTMENTS table as the parent table, and the EMPLOYEES table as the child we will first INSERT a row into the DEPARTMENTS table.
        `insert into departments (name, location) values
           ('Finance','New York');
        
        insert into departments (name, location) values
           ('Development','San Jose');`
         Insert into EditorCode Block 4
        
        Lets verify that the insert was successful by running a SQL SELECT statement to query all columns and all rows of our table.
        `select * from departments;`
         Insert into EditorCode Block 5
        
        You can see that an ID will have been automatically generated. You can now insert into the EMPLOYEES table a new row but you will need to put the generated DEPTID value into your SQL INSERT statement. The examples below show how we can do this using a SQL query, but you could simply enter the department number directly.
        `insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Sam Smith','Programmer', 
            5000, 
          (select deptno 
          from departments 
          where name = 'Development'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Mara Martin','Analyst', 
           6000, 
           (select deptno 
           from departments 
           where name = 'Finance'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Yun Yates','Analyst', 
           5500, 
           (select deptno 
           from departments 
           where name = 'Development'));`
         Insert into EditorCode Block 6
        
        ****4. Indexing Columns****
        Typically developers index columns for three major reasons:
        1. To enforce unique values within a column
        2. To improve data access performance
        3. To prevent lock escalation when updating rows of tables that use declarative referential integrity
        When a table is created and a PRIMARY KEY is specified an index is automatically created to enforce the primary key constraint. If you specific UNIQUE for a column when creating a column a unique index is also created. To see the indexes that already exist for a given table you can run the following dictionary query.
        `select table_name "Table", 
               index_name "Index", 
               column_name "Column", 
               column_position "Position"
        from  user_ind_columns 
        where table_name = 'EMPLOYEES' or 
              table_name = 'DEPARTMENTS'
        order by table_name, column_name, column_position`
         Insert into EditorCode Block 7
        
        It is typically good form to index foreign keys, foreign keys are columns in a table that reference another table. In our EMPLOYEES and DEPARTMENTS table example the DEPTNO column in the EMPLOYEE table references the primary key of the DEPARTMENTS table.
        `create index employee_dept_no_fk_idx 
        on employees (deptno)`
         Insert into EditorCode Block 8
        
        We may also determine that the EMPLOYEE table will be frequently searched by the NAME column. To improve the performance searches and to ensure uniqueness we can create a unique index on the EMPLOYEE table NAME column.
        `create unique index employee_ename_idx
        on employees (name)`
         Insert into EditorCode Block 9
        
        Oracle provides many other indexing technologies including function based indexes which can index expressions, such as an upper function, text indexes which can index free form text, bitmapped indexes useful in data warehousing. You can also create indexed organized tables, you can use partition indexes and more. Sometimes it is best to have fewer indexes and take advantage of in memory capabilities. All of these topics are beyond the scope of this basic introduction.
        ****5. Querying Data****
        To select data from a single table it is reasonably easy, simply use the SELECT ... FROM ... WHERE ... ORDER BY ... syntax.
        `select * from employees;`
         Insert into EditorCode Block 10
        
        To query data from two related tables you can join the data
        `select e.name employee,
                   d.name department,
                   e.job,
                   d.location
        from departments d, employees e
        where d.deptno = e.deptno(+)
        order by e.name;`
         Insert into EditorCode Block 11
        
        As an alternative to a join you can use an inline select to query data.
        `select e.name employee,
                  (select name 
                   from departments d 
                   where d.deptno = e.deptno) department,
                   e.job
        from employees e
        order by e.name;`
         Insert into EditorCode Block 12
        
        ****6. Adding Columns****
        You can add additional columns after you have created your table using the ALTER TABLE ... ADD ... syntax. For example:
        `alter table EMPLOYEES 
        add country_code varchar2(2);`
         Insert into EditorCode Block 13
        
        ****7. Querying the Oracle Data Dictionary****
        Table meta data is accessible from the Oracle data dictionary. The following queries show how you can query the data dictionary tables.
        `select table_name, tablespace_name, status
        from user_tables
        where table_Name = 'EMPLOYEES';
        
        select column_id, column_name , data_type
        from user_tab_columns
        where table_Name = 'EMPLOYEES'
        order by column_id;`
         Insert into EditorCode Block 14
        
        ****8. Updating Data****
        You can use SQL to update values in your table, to do this we will use the update clause
        `update employees
        set country_code = 'US';`
         Insert into EditorCode Block 15
        
        The query above will update all rows of the employee table and set the value of country code to US. You can also selectively update just a specific row.
        `update employees
        set commission = 2000
        where  name = 'Sam Smith';`
         Insert into EditorCode Block 16
        
        Lets run a Query to see what our data looks like
        `select name, country_code, salary, commission
        from employees
        order by name;`
         Insert into EditorCode Block 17
        
        ****9. Aggregate Queries****
        You can sum data in tables using aggregate functions. We will use column aliases to rename columns for readability, we will also use the null value function (NVL) to allow us to properly sum columns with null values.
        `select 
              count(*) employee_count,
              sum(salary) total_salary,
              sum(commission) total_commission,
              min(salary + nvl(commission,0)) min_compensation,
              max(salary + nvl(commission,0)) max_compensation
        from employees;`
         Insert into EditorCode Block 18
        
        ****10. Compressing Data****
        As your database grows in size to gigabytes or terabytes and beyond, consider using table compression. Table compression saves disk space and reduces memory use in the buffer cache. Table compression can also speed up query execution during reads. There is, however, a cost in CPU overhead for data loading and DML. Table compression is completely transparent to applications. It is especially useful in online analytical processing (OLAP) systems, where there are lengthy read-only operations, but can also be used in online transaction processing (OLTP) systems.
        You specify table compression with the COMPRESS clause of the CREATE TABLE statement. You can enable compression for an existing table by using this clause in an ALTER TABLE statement. In this case, the only data that is compressed is the data inserted or updated after compression is enabled. Similarly, you can disable table compression for an existing compressed table with the ALTER TABLE...NOCOMPRESS statement. In this case, all data the was already compressed remains compressed, and new data is inserted uncompressed.
        To enable compression for future data use the following syntax.
        `alter table EMPLOYEES compress for oltp; 
        alter table DEPARTMENTS compress for oltp;` 
         Insert into EditorCode Block 19
        
        ****11. Deleting Data****
        You can delete one or more rows from a table using the DELETE syntax. For example to delete a specific row:
        `delete from employees 
        where name = 'Sam Smith';`
         Insert into EditorCode Block 20
        
        ****12. Dropping Tables****
        You can drop tables using the SQL DROP command. Dropping a table will remove all of the rows and drop sub-objects including indexes and triggers. The following DROP statements will drop the departments and employees tables. The optional cascade constraints clause will drop remove constraints thus allowing you to drop database tables in any order.
        `drop table departments cascade constraints;
        drop table employees cascade constraints;`
         Insert into EditorCode Block 21
        
        ****13. Un-dropping Tables****
        If the RECYCLEBIN initialization parameter is set to ON (the default in 10g), then dropping this table will place it in the recycle bin. To see if you can undrop a table run the following data dictionary query:
        `select object_name, 
               original_name, 
               type, 
               can_undrop, 
               can_purge
        from recyclebin;`
         Insert into EditorCode Block 22
        
        To undrop tables we use the flashback command, for example:
        `flashback table DEPARTMENTS to before drop;
        flashback table EMPLOYEES to before drop;
        select count(*) departments 
        from departments;
        select count(*) employees
        from employees;`
         Insert into EditorCode Block 23
        > 
        > 
        > 
        > -> 다음의 문법을 바탕으로 Find the average total credits over all prior years. 의 결과를 출력하는 쿼리를 생각해봐 그리고 각각의 과정이 어떻게 이뤄지는지 확인해봐
        > 

- 프롬프트 끝

- 결과
        
        SELECT AVG(tot_cred) AS avg_total_credits
        FROM student
        WHERE ID NOT IN (
            SELECT DISTINCT ID
            FROM takes
            WHERE year = EXTRACT(YEAR FROM CURRENT_DATE)
        );
        
2. `Ask ChatGPT to generate a SQL query **with schema information**.`

- 프롬프트 시작
  
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > create table classroom
        > (building		varchar(15),
        > room_number		varchar(7),
        > capacity		numeric(4,0),
        > primary key (building, room_number)
        > );
        > 
        > create table department
        > (dept_name		varchar(20),
        > building		varchar(15),
        > budget		        numeric(12,2) check (budget > 0),
        > primary key (dept_name)
        > );
        > 
        > create table course
        > (course_id		varchar(8),
        > title			varchar(50),
        > dept_name		varchar(20),
        > credits		numeric(2,0) check (credits > 0),
        > primary key (course_id),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table instructor
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > salary			numeric(8,2) check (salary > 29000),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table section
        > (course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6)
        > check (semester in ('Fall', 'Winter', 'Spring', 'Summer')),
        > year			numeric(4,0) check (year > 1701 and year < 2100),
        > building		varchar(15),
        > room_number		varchar(7),
        > time_slot_id		varchar(4),
        > primary key (course_id, sec_id, semester, year),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (building, room_number) references classroom (building, room_number)
        > on delete set null
        > );
        > 
        > create table teaches
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references instructor (ID)
        > on delete cascade
        > );
        > 
        > create table student
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > tot_cred		numeric(3,0) check (tot_cred >= 0),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table takes
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > grade		        varchar(2),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table advisor
        > (s_ID			varchar(5),
        > i_ID			varchar(5),
        > primary key (s_ID),
        > foreign key (i_ID) references instructor (ID)
        > on delete set null,
        > foreign key (s_ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table time_slot
        > (time_slot_id		varchar(4),
        > day			varchar(1),
        > start_hr		numeric(2) check (start_hr >= 0 and start_hr < 24),
        > start_min		numeric(2) check (start_min >= 0 and start_min < 60),
        > end_hr			numeric(2) check (end_hr >= 0 and end_hr < 24),
        > end_min		numeric(2) check (end_min >= 0 and end_min < 60),
        > primary key (time_slot_id, day, start_hr, start_min)
        > );
        > 
        > create table prereq
        > (course_id		varchar(8),
        > prereq_id		varchar(8),
        > primary key (course_id, prereq_id),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (prereq_id) references course (course_id)
        > ); 
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **목적 파악:** 평균 총 학점을 모든 과거 연도에 대해 계산해야 합니다. 이는 "prior years"라는 표현을 고려할 때 현재 연도를 제외하고 계산해야 한다는 것을 의미합니다.
        > 2. **데이터 선택:** 관련 데이터가 어느 열에 있는지 어떤 데이터를 선택해야 하는지 확인해.
        > 3. **시간 제한 설정:** 관련된 테이블을 참조하여 과거 연도의 데이터를 필터링해야 합니다. ' 수강한 과목의 연도 정보를 포함하고 있는 테이블을 참조해야 해.
        > 4. **집계 함수 사용:** 'tot_cred'의 총 합을 계산한 후, 학생 수로 나누어 평균을 구합니다. 이를 위해 SQL의 **`SUM`**과 **`COUNT`** 집계 함수를 사용해.
        > 5. **쿼리 작성:** 모든 학생의 'tot_cred' 값을 합산하고, 그 수로 나누어 평균 총 학점을 구합니다.
        > 
        > → 자 이제 Find the average total credits over all prior years. 의 결과를 출력하는 쿼리를 작성해줘
        > 

- 프롬프트 끝

  
- 결과
    
        SELECT AVG(tot_cred) as average_total_credits
        FROM student
        WHERE ID NOT IN (
            SELECT DISTINCT ID
            FROM takes
            WHERE year = EXTRACT(YEAR FROM SYSDATE)
        );
    
3.  `Give **some additional information**(i.e., explanation of table schema, PK/FK relationship, clause you want to use etc.) to ChatGPT to make the SQL query more accurate.`

- 프롬프트 시작
  
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > create table classroom
        > (building		varchar(15),
        > room_number		varchar(7),
        > capacity		numeric(4,0),
        > primary key (building, room_number)
        > );
        > 
        > create table department
        > (dept_name		varchar(20),
        > building		varchar(15),
        > budget		        numeric(12,2) check (budget > 0),
        > primary key (dept_name)
        > );
        > 
        > create table course
        > (course_id		varchar(8),
        > title			varchar(50),
        > dept_name		varchar(20),
        > credits		numeric(2,0) check (credits > 0),
        > primary key (course_id),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table instructor
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > salary			numeric(8,2) check (salary > 29000),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table section
        > (course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6)
        > check (semester in ('Fall', 'Winter', 'Spring', 'Summer')),
        > year			numeric(4,0) check (year > 1701 and year < 2100),
        > building		varchar(15),
        > room_number		varchar(7),
        > time_slot_id		varchar(4),
        > primary key (course_id, sec_id, semester, year),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (building, room_number) references classroom (building, room_number)
        > on delete set null
        > );
        > 
        > create table teaches
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references instructor (ID)
        > on delete cascade
        > );
        > 
        > create table student
        > (ID			varchar(5),
        > name			varchar(20) not null,
        > dept_name		varchar(20),
        > tot_cred		numeric(3,0) check (tot_cred >= 0),
        > primary key (ID),
        > foreign key (dept_name) references department (dept_name)
        > on delete set null
        > );
        > 
        > create table takes
        > (ID			varchar(5),
        > course_id		varchar(8),
        > sec_id			varchar(8),
        > semester		varchar(6),
        > year			numeric(4,0),
        > grade		        varchar(2),
        > primary key (ID, course_id, sec_id, semester, year),
        > foreign key (course_id, sec_id, semester, year) references section (course_id, sec_id, semester, year)
        > on delete cascade,
        > foreign key (ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table advisor
        > (s_ID			varchar(5),
        > i_ID			varchar(5),
        > primary key (s_ID),
        > foreign key (i_ID) references instructor (ID)
        > on delete set null,
        > foreign key (s_ID) references student (ID)
        > on delete cascade
        > );
        > 
        > create table time_slot
        > (time_slot_id		varchar(4),
        > day			varchar(1),
        > start_hr		numeric(2) check (start_hr >= 0 and start_hr < 24),
        > start_min		numeric(2) check (start_min >= 0 and start_min < 60),
        > end_hr			numeric(2) check (end_hr >= 0 and end_hr < 24),
        > end_min		numeric(2) check (end_min >= 0 and end_min < 60),
        > primary key (time_slot_id, day, start_hr, start_min)
        > );
        > 
        > create table prereq
        > (course_id		varchar(8),
        > prereq_id		varchar(8),
        > primary key (course_id, prereq_id),
        > foreign key (course_id) references course (course_id)
        > on delete cascade,
        > foreign key (prereq_id) references course (course_id)
        > ); 
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **목적 파악:** 평균 총 학점을 모든 과거 연도에 대해 계산해야 합니다. 이는 "prior years"라는 표현을 고려할 때 현재 연도를 제외하고 계산해야 한다는 것을 의미합니다.
        > 2. **데이터 선택:** 관련 데이터가 어느 열에 있는지 어떤 데이터를 선택해야 하는지 확인해.
        > 3. **시간 제한 설정:** 관련된 테이블을 참조하여 과거 연도의 데이터를 필터링해야 합니다. ' 수강한 과목의 연도 정보를 포함하고 있는 테이블을 참조해야 해.
        > 4. **집계 함수 사용:** 'tot_cred'의 총 합을 계산한 후, 학생 수로 나누어 평균을 구합니다. 이를 위해 SQL의 **`SUM`**과 **`COUNT`** 집계 함수를 사용해.
        > 5. **쿼리 작성:** 모든 학생의 'tot_cred' 값을 합산하고, 그 수로 나누어 평균 총 학점을 구합니다.
        > 
        > → 
        > 
        > *주의 사항
        > 
        > 1. 오류가 나지 않게 주의해
        > 2. **To get the current year in Oracle,** you use **`EXTRACT(YEAR FROM SYSDATE)`**.
        > 3. **To calculate the total credits for prior years,** we first need to filter out the current year's data. However, the **`takes`** table includes individual course enrollments, not total credits, and **`HAVING`** clause is used after **`GROUP BY`** to filter groups, not to filter out individual records for aggregation in a subquery.
        > 
        > → 이걸 주의하면서 Find the average total credits over all prior years. 의 결과를 출력하는 쿼리를 작성해줘. 쿼리 결과만 주면 돼.
        > 

- 프롬프트 끝

  
- 결과
    
        SELECT AVG(tot_cred) AS average_total_credits
        FROM student
        WHERE ID NOT IN (
          SELECT ID
          FROM takes
          WHERE year = EXTRACT(YEAR FROM CURRENT_DATE)
        );
    
    ***분석***
    
    <aside>
    💡 분석
    
    1. 현재 연도에 과목을 수강하지 않은 학생들의 전체 학점 평균을 계산합니다. 
    2. 'student' 테이블에서 'ID'를 선택하여 현재 연도에 'takes' 테이블에 등록되지 않은 학생들만을 필터링하고, 
    3. 이 그룹의 'tot_cred' (총 학점)의 평균을 'average_total_credits'로 표시합니다.
    </aside>
    

### 세 번째 문제

- List the last 25% of employees when ranking by salary. (Evaluation data will consist of 60 tuples.)
    1. `Ask ChatGPT to generate a SQL query **without giving any schema information**.`

  
        제시한 쿼리는 다음과 같음.

- 프롬프트 시작
  
        > ****1. Creating Tables****
        Tables are the basic unit of data storage in an Oracle Database. Data is stored in rows and columns. You define a table with a table name, such as employees, and a set of columns. You give each column a column name, such as employee_id, last_name, and job_id; a datatype, such as VARCHAR2, DATE, or NUMBER; and a width. The width can be predetermined by the datatype, as in DATE. If columns are of the NUMBER datatype, define precision and scale instead of width. A row is a collection of column information corresponding to a single record.
        You can specify rules for each column of a table. These rules are called integrity constraints. One example is a NOT NULL integrity constraint. This constraint forces the column to contain a value in every row.
        For example:
        `create table DEPARTMENTS (  
          deptno        number,  
          name          varchar2(50) not null,  
          location      varchar2(50),  
          constraint pk_departments primary key (deptno)  
        );`
         Insert into EditorCode Block 1
        
        Tables can declarative specify relationships between tables, typically referred to as referential integrity. To see how this works we can create a "child" table of the DEPARTMENTS table by including a foreign key in the EMPLOYEES table that references the DEPARTMENTS table. For example:
        `create table EMPLOYEES (  
          empno             number,  
          name              varchar2(50) not null,  
          job               varchar2(50),  
          manager           number,  
          hiredate          date,  
          salary            number(7,2),  
          commission        number(7,2),  
          deptno           number,  
          constraint pk_employees primary key (empno),  
          constraint fk_employees_deptno foreign key (deptno) 
              references DEPARTMENTS (deptno)  
        );`
         Insert into EditorCode Block 2
        
        Foreign keys must reference primary keys, so to create a "child" table the "parent" table must have a primary key for the foreign key to reference.
        ****2. Creating Triggers****
        Triggers are procedures that are stored in the database and are implicitly run, or fired, when something happens. Traditionally, triggers supported the execution of a procedural code, in Oracle procedural SQL is called a PL/SQL block. PL stands for procedural language. When an INSERT, UPDATE, or DELETE occurred on a table or view. Triggers support system and other data events on DATABASE and SCHEMA.
        Triggers are frequently used to automatically populate table primary keys, the trigger examples below show an example trigger to do just this. We will use a built in function to obtain a globallally unique identifier or GUID.
        `create or replace trigger  DEPARTMENTS_BIU
            before insert or update on DEPARTMENTS
            for each row
        begin
            if inserting and :new.deptno is null then
                :new.deptno := to_number(sys_guid(), 
                  'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /
        
        create or replace trigger EMPLOYEES_BIU
            before insert or update on EMPLOYEES
            for each row
        begin
            if inserting and :new.empno is null then
                :new.empno := to_number(sys_guid(), 
                    'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /`
         Insert into EditorCode Block 3
        
        ****3. Inserting Data****
        Now that we have tables created, and we have triggers to automatically populate our primary keys, we can add data to our tables. Because we have a parent child relationship, with the DEPARTMENTS table as the parent table, and the EMPLOYEES table as the child we will first INSERT a row into the DEPARTMENTS table.
        `insert into departments (name, location) values
           ('Finance','New York');
        
        insert into departments (name, location) values
           ('Development','San Jose');`
         Insert into EditorCode Block 4
        
        Lets verify that the insert was successful by running a SQL SELECT statement to query all columns and all rows of our table.
        `select * from departments;`
         Insert into EditorCode Block 5
        
        You can see that an ID will have been automatically generated. You can now insert into the EMPLOYEES table a new row but you will need to put the generated DEPTID value into your SQL INSERT statement. The examples below show how we can do this using a SQL query, but you could simply enter the department number directly.
        `insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Sam Smith','Programmer', 
            5000, 
          (select deptno 
          from departments 
          where name = 'Development'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Mara Martin','Analyst', 
           6000, 
           (select deptno 
           from departments 
           where name = 'Finance'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Yun Yates','Analyst', 
           5500, 
           (select deptno 
           from departments 
           where name = 'Development'));`
         Insert into EditorCode Block 6
        
        ****4. Indexing Columns****
        Typically developers index columns for three major reasons:
        1. To enforce unique values within a column
        2. To improve data access performance
        3. To prevent lock escalation when updating rows of tables that use declarative referential integrity
        When a table is created and a PRIMARY KEY is specified an index is automatically created to enforce the primary key constraint. If you specific UNIQUE for a column when creating a column a unique index is also created. To see the indexes that already exist for a given table you can run the following dictionary query.
        `select table_name "Table", 
               index_name "Index", 
               column_name "Column", 
               column_position "Position"
        from  user_ind_columns 
        where table_name = 'EMPLOYEES' or 
              table_name = 'DEPARTMENTS'
        order by table_name, column_name, column_position`
         Insert into EditorCode Block 7
        
        It is typically good form to index foreign keys, foreign keys are columns in a table that reference another table. In our EMPLOYEES and DEPARTMENTS table example the DEPTNO column in the EMPLOYEE table references the primary key of the DEPARTMENTS table.
        `create index employee_dept_no_fk_idx 
        on employees (deptno)`
         Insert into EditorCode Block 8
        
        We may also determine that the EMPLOYEE table will be frequently searched by the NAME column. To improve the performance searches and to ensure uniqueness we can create a unique index on the EMPLOYEE table NAME column.
        `create unique index employee_ename_idx
        on employees (name)`
         Insert into EditorCode Block 9
        
        Oracle provides many other indexing technologies including function based indexes which can index expressions, such as an upper function, text indexes which can index free form text, bitmapped indexes useful in data warehousing. You can also create indexed organized tables, you can use partition indexes and more. Sometimes it is best to have fewer indexes and take advantage of in memory capabilities. All of these topics are beyond the scope of this basic introduction.
        ****5. Querying Data****
        To select data from a single table it is reasonably easy, simply use the SELECT ... FROM ... WHERE ... ORDER BY ... syntax.
        `select * from employees;`
         Insert into EditorCode Block 10
        
        To query data from two related tables you can join the data
        `select e.name employee,
                   d.name department,
                   e.job,
                   d.location
        from departments d, employees e
        where d.deptno = e.deptno(+)
        order by e.name;`
         Insert into EditorCode Block 11
        
        As an alternative to a join you can use an inline select to query data.
        `select e.name employee,
                  (select name 
                   from departments d 
                   where d.deptno = e.deptno) department,
                   e.job
        from employees e
        order by e.name;`
         Insert into EditorCode Block 12
        
        ****6. Adding Columns****
        You can add additional columns after you have created your table using the ALTER TABLE ... ADD ... syntax. For example:
        `alter table EMPLOYEES 
        add country_code varchar2(2);`
         Insert into EditorCode Block 13
        
        ****7. Querying the Oracle Data Dictionary****
        Table meta data is accessible from the Oracle data dictionary. The following queries show how you can query the data dictionary tables.
        `select table_name, tablespace_name, status
        from user_tables
        where table_Name = 'EMPLOYEES';
        
        select column_id, column_name , data_type
        from user_tab_columns
        where table_Name = 'EMPLOYEES'
        order by column_id;`
         Insert into EditorCode Block 14
        
        ****8. Updating Data****
        You can use SQL to update values in your table, to do this we will use the update clause
        `update employees
        set country_code = 'US';`
         Insert into EditorCode Block 15
        
        The query above will update all rows of the employee table and set the value of country code to US. You can also selectively update just a specific row.
        `update employees
        set commission = 2000
        where  name = 'Sam Smith';`
         Insert into EditorCode Block 16
        
        Lets run a Query to see what our data looks like
        `select name, country_code, salary, commission
        from employees
        order by name;`
         Insert into EditorCode Block 17
        
        ****9. Aggregate Queries****
        You can sum data in tables using aggregate functions. We will use column aliases to rename columns for readability, we will also use the null value function (NVL) to allow us to properly sum columns with null values.
        `select 
              count(*) employee_count,
              sum(salary) total_salary,
              sum(commission) total_commission,
              min(salary + nvl(commission,0)) min_compensation,
              max(salary + nvl(commission,0)) max_compensation
        from employees;`
         Insert into EditorCode Block 18
        
        ****10. Compressing Data****
        As your database grows in size to gigabytes or terabytes and beyond, consider using table compression. Table compression saves disk space and reduces memory use in the buffer cache. Table compression can also speed up query execution during reads. There is, however, a cost in CPU overhead for data loading and DML. Table compression is completely transparent to applications. It is especially useful in online analytical processing (OLAP) systems, where there are lengthy read-only operations, but can also be used in online transaction processing (OLTP) systems.
        You specify table compression with the COMPRESS clause of the CREATE TABLE statement. You can enable compression for an existing table by using this clause in an ALTER TABLE statement. In this case, the only data that is compressed is the data inserted or updated after compression is enabled. Similarly, you can disable table compression for an existing compressed table with the ALTER TABLE...NOCOMPRESS statement. In this case, all data the was already compressed remains compressed, and new data is inserted uncompressed.
        To enable compression for future data use the following syntax.
        `alter table EMPLOYEES compress for oltp; 
        alter table DEPARTMENTS compress for oltp;` 
         Insert into EditorCode Block 19
        
        ****11. Deleting Data****
        You can delete one or more rows from a table using the DELETE syntax. For example to delete a specific row:
        `delete from employees 
        where name = 'Sam Smith';`
         Insert into EditorCode Block 20
        
        ****12. Dropping Tables****
        You can drop tables using the SQL DROP command. Dropping a table will remove all of the rows and drop sub-objects including indexes and triggers. The following DROP statements will drop the departments and employees tables. The optional cascade constraints clause will drop remove constraints thus allowing you to drop database tables in any order.
        `drop table departments cascade constraints;
        drop table employees cascade constraints;`
         Insert into EditorCode Block 21
        
        ****13. Un-dropping Tables****
        If the RECYCLEBIN initialization parameter is set to ON (the default in 10g), then dropping this table will place it in the recycle bin. To see if you can undrop a table run the following data dictionary query:
        `select object_name, 
               original_name, 
               type, 
               can_undrop, 
               can_purge
        from recyclebin;`
         Insert into EditorCode Block 22
        
        To undrop tables we use the flashback command, for example:
        `flashback table DEPARTMENTS to before drop;
        flashback table EMPLOYEES to before drop;
        select count(*) departments 
        from departments;
        select count(*) employees
        from employees;`
         Insert into EditorCode Block 23
        > 
        > 
        > 
        > -> 다음의 문법을 바탕으로  List the last 25% of employees when ranking by salary. (Evaluation data will consist of 60 tuples.)의 결과를 출력하는 쿼리를 생각해봐 그리고 각각의 과정이 어떻게 이뤄지는지 확인해봐
        > 

- 프롬프트 끝

- 결과
        
        SELECT *
        FROM (
          SELECT e.*, ROW_NUMBER() OVER (ORDER BY salary DESC) as salary_rank
          FROM employees e
        )
        WHERE salary_rank > (0.75 * 60); -- Calculate the position after which the last 25% begins
        

---

2. `Ask ChatGPT to generate a SQL query **with schema information**.`

- 프롬프트 시작
      
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE DEPT (
        > DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        > DNAME VARCHAR2(14) ,
        > LOC VARCHAR2(13) ) ;
        > 
        > CREATE TABLE EMP(
        > EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        > ENAME VARCHAR2(10),
        > JOB VARCHAR2(9),
        > MGR NUMBER(4),
        > HIREDATE DATE,
        > SAL NUMBER(7,2),
        > COMM NUMBER(7,2),
        > DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
        > 
        > CREATE TABLE BONUS (
        > ENAME VARCHAR2(10) ,
        > JOB VARCHAR2(9)  ,
        > SAL NUMBER,
        > COMM NUMBER
        > ) ;
        > 
        > CREATE TABLE SALGRADE (
        > GRADE NUMBER,
        > LOSAL NUMBER,
        > HISAL NUMBER
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **결과 정의:** 문제를 이해하고, 어떤 결과가 필요한지 정의합니다. 이 경우, 모든 직원을 급여 순으로 정렬하고, 하위 25%의 목록을 얻어야 합니다.
        > 2. **데이터셋 크기 고려:** 전체 데이터셋의 크기가 주어졌을 때(60명의 직원), 하위 25%를 찾기 위해 전체 직원 수의 75% 위치를 계산합니다.
        > 3. **정렬 메커니즘 선택:** 직원을 급여에 따라 정렬하기 위해 **`ORDER BY`** 절과 함께 **`ROW_NUMBER()`** 윈도우 함수를 사용합니다. **`ROW_NUMBER()`**는 각 행에 순차적 번호를 할당하여 급여 순위를 생성합니다.
        > 4. **필터링 조건 설정:** 하위 25%에 해당하는 직원만 선택하기 위해 **`WHERE`** 절을 사용하여 75% 위치보다 높은 급여 순위를 가진 직원을 필터링합니다.
        > 5. **쿼리 구조화:** 위의 생각 과정을 바탕으로, 서브쿼리를 사용하여 각 직원에 대한 전체 데이터와 그들의 급여 순위를 선택하고, 그 결과를 외부 쿼리에서 필터링하여 최종 결과를 얻습니다.
        > 6. **쿼리 최적화 고려:** 쿼리 효율성을 고려하여 필요한 최소한의 컬럼만을 선택하거나 적절한 인덱스 사용을 고려할 수 있습니다.
        > 7. **동적 데이터셋 적응성:** 데이터셋의 크기가 변할 수 있는 경우를 대비하여, 직원 수를 동적으로 계산하는 방법을 고려합니다. 이 예에서는 고정된 숫자(60)를 사용했지만, 직원 수가 변동될 경우 **`COUNT(*) OVER()`** 함수를 사용하여 전체 직원 수를 계산한 후 이를 기준으로 25%를 계산할 수 있습니다.
        > 
        > → 자 이제 List the last 25% of employees when ranking by salary. (Evaluation data will consist of 60 tuples.)의 결과를 출력하는 쿼리를 작성해줘 
        > 

- 프롬프트 끝

  
- 결과
    
        SELECT ENAME, SAL
        FROM (
          SELECT ENAME, SAL, ROW_NUMBER() OVER (ORDER BY SAL) as SAL_RANK, COUNT(*) OVER () as TOTAL_COUNT
          FROM EMP
        ) WHERE SAL_RANK > (0.75 * TOTAL_COUNT)
        ORDER BY SAL;
        

---

3.  `Give **some additional information**(i.e., explanation of table schema, PK/FK relationship, clause you want to use etc.) to ChatGPT to make the SQL query more accurate.`

- 프롬프트 시작
      
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE DEPT (
        > DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        > DNAME VARCHAR2(14) ,
        > LOC VARCHAR2(13) ) ;
        > 
        > CREATE TABLE EMP(
        > EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        > ENAME VARCHAR2(10),
        > JOB VARCHAR2(9),
        > MGR NUMBER(4),
        > HIREDATE DATE,
        > SAL NUMBER(7,2),
        > COMM NUMBER(7,2),
        > DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
        > 
        > CREATE TABLE BONUS (
        > ENAME VARCHAR2(10) ,
        > JOB VARCHAR2(9)  ,
        > SAL NUMBER,
        > COMM NUMBER
        > ) ;
        > 
        > CREATE TABLE SALGRADE (
        > GRADE NUMBER,
        > LOSAL NUMBER,
        > HISAL NUMBER
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **결과 정의:** 문제를 이해하고, 어떤 결과가 필요한지 정의합니다. 이 경우, 모든 직원을 급여 순으로 정렬하고, 하위 25%의 목록을 얻어야 합니다.
        > 2. **데이터셋 크기 고려:** 전체 데이터셋의 크기가 주어졌을 때(60명의 직원), 하위 25%를 찾기 위해 전체 직원 수의 75% 위치를 계산합니다.
        > 3. **정렬 메커니즘 선택:** 직원을 급여에 따라 정렬하기 위해 **`ORDER BY`** 절과 함께 **`ROW_NUMBER()`** 윈도우 함수를 사용합니다. **`ROW_NUMBER()`**는 각 행에 순차적 번호를 할당하여 급여 순위를 생성합니다.
        > 4. **필터링 조건 설정:** 하위 25%에 해당하는 직원만 선택하기 위해 **`WHERE`** 절을 사용하여 75% 위치보다 높은 급여 순위를 가진 직원을 필터링합니다.
        > 5. **쿼리 구조화:** 위의 생각 과정을 바탕으로, 서브쿼리를 사용하여 각 직원에 대한 전체 데이터와 그들의 급여 순위를 선택하고, 그 결과를 외부 쿼리에서 필터링하여 최종 결과를 얻습니다.
        > 6. **쿼리 최적화 고려:** 쿼리 효율성을 고려하여 필요한 최소한의 컬럼만을 선택하거나 적절한 인덱스 사용을 고려할 수 있습니다.
        > 7. **동적 데이터셋 적응성:** 데이터셋의 크기가 변할 수 있는 경우를 대비하여, 직원 수를 동적으로 계산하는 방법을 고려합니다. 이 예에서는 고정된 숫자(60)를 사용했지만, 직원 수가 변동될 경우 **`COUNT(*) OVER()`** 함수를 사용하여 전체 직원 수를 계산한 후 이를 기준으로 25%를 계산할 수 있습니다.
        > 
        > → 자 이제 List the last 25% of employees when ranking by salary. (Evaluation data will consist of 60 tuples.)의 결과를 출력하는 쿼리를 작성해줘 
        > 
        > *주의 사항
        > 
        > 1. 오류가 나지 않게 주의해
        > 2. It will rank employees by their salary in descending order using the **`ROW_NUMBER()`** function.
        > 3. It will count the total number of employees using the **`COUNT(*) OVER()`** function without partitioning, which will result in the total number being the same for each row, representing the total number of employees.
        > 4. The **`WHERE`** clause filters the results to only include those rows where the rank is greater than the threshold for the last 25% of the total count (which is calculated as **`0.75 * total_count`**).
        > 5. The **`ORDER BY`** clause at the end of the outer query makes sure to list the employees in descending order of their salaries, even though you are only returning the bottom quartile.
        > 
        > → 이걸 주의하면서 자 이제 List the last 25% of employees when ranking by salary. (Evaluation data will consist of 60 tuples.)의 결과를 출력하는 쿼리를 작성해줘  쿼리 결과만 주면 돼.
        > 

- 프롬프트 끝
  
- 결과
    
        SELECT ENAME, SAL
        FROM (
          SELECT ENAME, SAL, ROW_NUMBER() OVER (ORDER BY SAL DESC) AS rn, COUNT(*) OVER() AS total_count
          FROM EMP
        ) WHERE rn > (0.75 * total_count)
        ORDER BY SAL DESC;


    ***분석***
    
    <aside>
    💡 분석
    
    1. 이 쿼리는 'EMP' 테이블에서 직원의 이름('ENAME')과 급여('SAL')를 선택하고, 
    2. 내림차순으로 급여 기준으로 정렬하여 각 행에 대한 순위를 매긴 후('ROW_NUMBER() OVER (ORDER BY SAL DESC)'), 
    3. 전체 행의 수를 계산합니다('COUNT(*) OVER()'). 
    4. 그런 다음 이 서브쿼리의 결과를 사용하여 전체 직원 중 급여가 하위 25%에 해당하는 직원만 필터링하여('WHERE rn > (0.75 * total_count)') 
    5. 이들의 급여를 내림차순으로 나열합니다('ORDER BY SAL DESC').
    </aside>
    

### 네 번째 문제

- Suppose we have only 4 departments(SALES, ACCOUNT, RESEARCH, OPERATION) in DEPT table. Show [1] the total salaries in the "SALES" and "ACCOUNT" deparments combined and [2] in the "RESEARCH" and "OPERATION" departments combined.
    1. `Ask ChatGPT to generate a SQL query **without giving any schema information**.`
        
        제시한 쿼리는 다음과 같음.

- 프롬프트 시작
  
        > ****1. Creating Tables****
        Tables are the basic unit of data storage in an Oracle Database. Data is stored in rows and columns. You define a table with a table name, such as employees, and a set of columns. You give each column a column name, such as employee_id, last_name, and job_id; a datatype, such as VARCHAR2, DATE, or NUMBER; and a width. The width can be predetermined by the datatype, as in DATE. If columns are of the NUMBER datatype, define precision and scale instead of width. A row is a collection of column information corresponding to a single record.
        You can specify rules for each column of a table. These rules are called integrity constraints. One example is a NOT NULL integrity constraint. This constraint forces the column to contain a value in every row.
        For example:
        `create table DEPARTMENTS (  
          deptno        number,  
          name          varchar2(50) not null,  
          location      varchar2(50),  
          constraint pk_departments primary key (deptno)  
        );`
         Insert into EditorCode Block 1
        
        Tables can declarative specify relationships between tables, typically referred to as referential integrity. To see how this works we can create a "child" table of the DEPARTMENTS table by including a foreign key in the EMPLOYEES table that references the DEPARTMENTS table. For example:
        `create table EMPLOYEES (  
          empno             number,  
          name              varchar2(50) not null,  
          job               varchar2(50),  
          manager           number,  
          hiredate          date,  
          salary            number(7,2),  
          commission        number(7,2),  
          deptno           number,  
          constraint pk_employees primary key (empno),  
          constraint fk_employees_deptno foreign key (deptno) 
              references DEPARTMENTS (deptno)  
        );`
         Insert into EditorCode Block 2
        
        Foreign keys must reference primary keys, so to create a "child" table the "parent" table must have a primary key for the foreign key to reference.
        ****2. Creating Triggers****
        Triggers are procedures that are stored in the database and are implicitly run, or fired, when something happens. Traditionally, triggers supported the execution of a procedural code, in Oracle procedural SQL is called a PL/SQL block. PL stands for procedural language. When an INSERT, UPDATE, or DELETE occurred on a table or view. Triggers support system and other data events on DATABASE and SCHEMA.
        Triggers are frequently used to automatically populate table primary keys, the trigger examples below show an example trigger to do just this. We will use a built in function to obtain a globallally unique identifier or GUID.
        `create or replace trigger  DEPARTMENTS_BIU
            before insert or update on DEPARTMENTS
            for each row
        begin
            if inserting and :new.deptno is null then
                :new.deptno := to_number(sys_guid(), 
                  'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /
        
        create or replace trigger EMPLOYEES_BIU
            before insert or update on EMPLOYEES
            for each row
        begin
            if inserting and :new.empno is null then
                :new.empno := to_number(sys_guid(), 
                    'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /`
         Insert into EditorCode Block 3
        
        ****3. Inserting Data****
        Now that we have tables created, and we have triggers to automatically populate our primary keys, we can add data to our tables. Because we have a parent child relationship, with the DEPARTMENTS table as the parent table, and the EMPLOYEES table as the child we will first INSERT a row into the DEPARTMENTS table.
        `insert into departments (name, location) values
           ('Finance','New York');
        
        insert into departments (name, location) values
           ('Development','San Jose');`
         Insert into EditorCode Block 4
        
        Lets verify that the insert was successful by running a SQL SELECT statement to query all columns and all rows of our table.
        `select * from departments;`
         Insert into EditorCode Block 5
        
        You can see that an ID will have been automatically generated. You can now insert into the EMPLOYEES table a new row but you will need to put the generated DEPTID value into your SQL INSERT statement. The examples below show how we can do this using a SQL query, but you could simply enter the department number directly.
        `insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Sam Smith','Programmer', 
            5000, 
          (select deptno 
          from departments 
          where name = 'Development'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Mara Martin','Analyst', 
           6000, 
           (select deptno 
           from departments 
           where name = 'Finance'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Yun Yates','Analyst', 
           5500, 
           (select deptno 
           from departments 
           where name = 'Development'));`
         Insert into EditorCode Block 6
        
        ****4. Indexing Columns****
        Typically developers index columns for three major reasons:
        1. To enforce unique values within a column
        2. To improve data access performance
        3. To prevent lock escalation when updating rows of tables that use declarative referential integrity
        When a table is created and a PRIMARY KEY is specified an index is automatically created to enforce the primary key constraint. If you specific UNIQUE for a column when creating a column a unique index is also created. To see the indexes that already exist for a given table you can run the following dictionary query.
        `select table_name "Table", 
               index_name "Index", 
               column_name "Column", 
               column_position "Position"
        from  user_ind_columns 
        where table_name = 'EMPLOYEES' or 
              table_name = 'DEPARTMENTS'
        order by table_name, column_name, column_position`
         Insert into EditorCode Block 7
        
        It is typically good form to index foreign keys, foreign keys are columns in a table that reference another table. In our EMPLOYEES and DEPARTMENTS table example the DEPTNO column in the EMPLOYEE table references the primary key of the DEPARTMENTS table.
        `create index employee_dept_no_fk_idx 
        on employees (deptno)`
         Insert into EditorCode Block 8
        
        We may also determine that the EMPLOYEE table will be frequently searched by the NAME column. To improve the performance searches and to ensure uniqueness we can create a unique index on the EMPLOYEE table NAME column.
        `create unique index employee_ename_idx
        on employees (name)`
         Insert into EditorCode Block 9
        
        Oracle provides many other indexing technologies including function based indexes which can index expressions, such as an upper function, text indexes which can index free form text, bitmapped indexes useful in data warehousing. You can also create indexed organized tables, you can use partition indexes and more. Sometimes it is best to have fewer indexes and take advantage of in memory capabilities. All of these topics are beyond the scope of this basic introduction.
        ****5. Querying Data****
        To select data from a single table it is reasonably easy, simply use the SELECT ... FROM ... WHERE ... ORDER BY ... syntax.
        `select * from employees;`
         Insert into EditorCode Block 10
        
        To query data from two related tables you can join the data
        `select e.name employee,
                   d.name department,
                   e.job,
                   d.location
        from departments d, employees e
        where d.deptno = e.deptno(+)
        order by e.name;`
         Insert into EditorCode Block 11
        
        As an alternative to a join you can use an inline select to query data.
        `select e.name employee,
                  (select name 
                   from departments d 
                   where d.deptno = e.deptno) department,
                   e.job
        from employees e
        order by e.name;`
         Insert into EditorCode Block 12
        
        ****6. Adding Columns****
        You can add additional columns after you have created your table using the ALTER TABLE ... ADD ... syntax. For example:
        `alter table EMPLOYEES 
        add country_code varchar2(2);`
         Insert into EditorCode Block 13
        
        ****7. Querying the Oracle Data Dictionary****
        Table meta data is accessible from the Oracle data dictionary. The following queries show how you can query the data dictionary tables.
        `select table_name, tablespace_name, status
        from user_tables
        where table_Name = 'EMPLOYEES';
        
        select column_id, column_name , data_type
        from user_tab_columns
        where table_Name = 'EMPLOYEES'
        order by column_id;`
         Insert into EditorCode Block 14
        
        ****8. Updating Data****
        You can use SQL to update values in your table, to do this we will use the update clause
        `update employees
        set country_code = 'US';`
         Insert into EditorCode Block 15
        
        The query above will update all rows of the employee table and set the value of country code to US. You can also selectively update just a specific row.
        `update employees
        set commission = 2000
        where  name = 'Sam Smith';`
         Insert into EditorCode Block 16
        
        Lets run a Query to see what our data looks like
        `select name, country_code, salary, commission
        from employees
        order by name;`
         Insert into EditorCode Block 17
        
        ****9. Aggregate Queries****
        You can sum data in tables using aggregate functions. We will use column aliases to rename columns for readability, we will also use the null value function (NVL) to allow us to properly sum columns with null values.
        `select 
              count(*) employee_count,
              sum(salary) total_salary,
              sum(commission) total_commission,
              min(salary + nvl(commission,0)) min_compensation,
              max(salary + nvl(commission,0)) max_compensation
        from employees;`
         Insert into EditorCode Block 18
        
        ****10. Compressing Data****
        As your database grows in size to gigabytes or terabytes and beyond, consider using table compression. Table compression saves disk space and reduces memory use in the buffer cache. Table compression can also speed up query execution during reads. There is, however, a cost in CPU overhead for data loading and DML. Table compression is completely transparent to applications. It is especially useful in online analytical processing (OLAP) systems, where there are lengthy read-only operations, but can also be used in online transaction processing (OLTP) systems.
        You specify table compression with the COMPRESS clause of the CREATE TABLE statement. You can enable compression for an existing table by using this clause in an ALTER TABLE statement. In this case, the only data that is compressed is the data inserted or updated after compression is enabled. Similarly, you can disable table compression for an existing compressed table with the ALTER TABLE...NOCOMPRESS statement. In this case, all data the was already compressed remains compressed, and new data is inserted uncompressed.
        To enable compression for future data use the following syntax.
        `alter table EMPLOYEES compress for oltp; 
        alter table DEPARTMENTS compress for oltp;` 
         Insert into EditorCode Block 19
        
        ****11. Deleting Data****
        You can delete one or more rows from a table using the DELETE syntax. For example to delete a specific row:
        `delete from employees 
        where name = 'Sam Smith';`
         Insert into EditorCode Block 20
        
        ****12. Dropping Tables****
        You can drop tables using the SQL DROP command. Dropping a table will remove all of the rows and drop sub-objects including indexes and triggers. The following DROP statements will drop the departments and employees tables. The optional cascade constraints clause will drop remove constraints thus allowing you to drop database tables in any order.
        `drop table departments cascade constraints;
        drop table employees cascade constraints;`
         Insert into EditorCode Block 21
        
        ****13. Un-dropping Tables****
        If the RECYCLEBIN initialization parameter is set to ON (the default in 10g), then dropping this table will place it in the recycle bin. To see if you can undrop a table run the following data dictionary query:
        `select object_name, 
               original_name, 
               type, 
               can_undrop, 
               can_purge
        from recyclebin;`
         Insert into EditorCode Block 22
        
        To undrop tables we use the flashback command, for example:
        `flashback table DEPARTMENTS to before drop;
        flashback table EMPLOYEES to before drop;
        select count(*) departments 
        from departments;
        select count(*) employees
        from employees;`
         Insert into EditorCode Block 23
        > 
        > 
        > 
        > -> 다음의 문법을 바탕으로  Suppose we have only 4 departments(SALES, ACCOUNT, RESEARCH, OPERATION) in DEPT table. Show [1] the total salaries in the "SALES" and "ACCOUNT" deparments combined and [2] in the "RESEARCH" and "OPERATION" departments combined.의 결과를 출력하는 쿼리를 생각해봐 그리고 각각의 과정이 어떻게 이뤄지는지 확인해봐
        > 

- 프롬프트 끝


- 결과
        
        SELECT 
          'SALES & ACCOUNT' AS Department_Group,
          SUM(SAL) AS Total_Salaries
        FROM EMP
        WHERE DEPTNO IN (SELECT DEPTNO FROM DEPT WHERE DNAME IN ('SALES', 'ACCOUNT'))
        UNION ALL
        SELECT 
          'RESEARCH & OPERATION' AS Department_Group,
          SUM(SAL) AS Total_Salaries
        FROM EMP
        WHERE DEPTNO IN (SELECT DEPTNO FROM DEPT WHERE DNAME IN ('RESEARCH', 'OPERATION'));
        

---

2. `Ask ChatGPT to generate a SQL query **with schema information**.`

- 프롬프트 시작


        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE DEPT (
        > DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        > DNAME VARCHAR2(14) ,
        > LOC VARCHAR2(13) ) ;
        > 
        > CREATE TABLE EMP(
        > EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        > ENAME VARCHAR2(10),
        > JOB VARCHAR2(9),
        > MGR NUMBER(4),
        > HIREDATE DATE,
        > SAL NUMBER(7,2),
        > COMM NUMBER(7,2),
        > DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
        > 
        > CREATE TABLE BONUS (
        > ENAME VARCHAR2(10) ,
        > JOB VARCHAR2(9)  ,
        > SAL NUMBER,
        > COMM NUMBER
        > ) ;
        > 
        > CREATE TABLE SALGRADE (
        > GRADE NUMBER,
        > LOSAL NUMBER,
        > HISAL NUMBER
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **Identify the Objective:** You're tasked with finding the total salaries for two groups of departments, 'SALES & ACCOUNT' and 'RESEARCH & OPERATION'. This requires summing salaries in each group.
        > 2. **Determine the Necessary Data:** You need to select salary information from the 'EMP' table and department names from the 'DEPT' table, requiring a way to correlate the two.
        > 3. **Understand Data Relations:** Since department names are given, and you need to calculate based on department numbers, you'll need to correlate department names with their respective numbers, likely through a join or subquery.
        > 4. **Decide on Grouping Strategy:** You need to group salaries by department but only for the specified pairs. This implies a need for conditional grouping, which can be achieved by filtering departments within the 'WHERE' clause or using a 'CASE' expression within the 'SUM' function.
        > 5. **Aggregate Salary Data:** Using the 'SUM' function, you can calculate the total salary for each department group. You might consider using subqueries to filter for 'DEPTNO' corresponding to each named group before aggregation.
        > 6. **Craft the Query with Unions:** Since you have two distinct department groups, you'll construct two select queries, one for each group, and combine them using 'UNION ALL' to get a complete picture in one result set.
        > 7. **Optimize for Readability and Performance:** Ensure the query is readable by clearly labeling results with 'AS' for column aliases. Additionally, optimize performance by only querying necessary columns and using 'UNION ALL' instead of 'UNION' to avoid unnecessary sorting and duplicate elimination.
        > 
        > → 자 이제 Suppose we have only 4 departments(SALES, ACCOUNT, RESEARCH, OPERATION) in DEPT table. Show [1] the total salaries in the "SALES" and "ACCOUNT" deparments combined and [2] in the "RESEARCH" and "OPERATION" departments combined.의 결과를 출력하는 쿼리를 작성해줘.
        > 

- 프롬프트 끝

  
    결과
    
        SELECT 'SALES & ACCOUNT' AS Department_Group, SUM(SAL) AS Total_Salary
        FROM EMP
        JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO
        WHERE DNAME IN ('SALES', 'ACCOUNT')
        UNION ALL
        SELECT 'RESEARCH & OPERATION' AS Department_Group, SUM(SAL) AS Total_Salary
        FROM EMP
        JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO
        WHERE DNAME IN ('RESEARCH', 'OPERATION');
        

---

3.  `Give **some additional information**(i.e., explanation of table schema, PK/FK relationship, clause you want to use etc.) to ChatGPT to make the SQL query more accurate.`

- 프롬프트 시작


        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE DEPT (
        > DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        > DNAME VARCHAR2(14) ,
        > LOC VARCHAR2(13) ) ;
        > 
        > CREATE TABLE EMP(
        > EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        > ENAME VARCHAR2(10),
        > JOB VARCHAR2(9),
        > MGR NUMBER(4),
        > HIREDATE DATE,
        > SAL NUMBER(7,2),
        > COMM NUMBER(7,2),
        > DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
        > 
        > CREATE TABLE BONUS (
        > ENAME VARCHAR2(10) ,
        > JOB VARCHAR2(9)  ,
        > SAL NUMBER,
        > COMM NUMBER
        > ) ;
        > 
        > CREATE TABLE SALGRADE (
        > GRADE NUMBER,
        > LOSAL NUMBER,
        > HISAL NUMBER
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > *주의 사항
        > 
        > 1. 오류가 나지 않게 주의해
        > 2. **Understand the Schema:** The schema provided has a 'DEPT' table with department names and numbers and an 'EMP' table with employee details, including their department numbers and salaries.
        > 3. **Link Departments to Salaries:** You need to join 'EMP' and 'DEPT' tables to match employees to their departments. The 'deptno' column in both tables serves as the link between them.
        > 4. **Isolate the Desired Departments:** Filter out employees working in the 'SALES,' 'ACCOUNT,' 'RESEARCH,' and 'OPERATION' departments.
        > 5. **Sum Salaries by Department Group:** Aggregate the salaries for each of the desired department groups. This will require summing 'sal' from 'EMP' for each department, grouped according to the department names you are interested in.
        > 6. **Combine the Results:** Since you are asked for combined totals for 'SALES & ACCOUNT' and 'RESEARCH & OPERATION,' you may use a 'UNION ALL' to bring these two groups' totals into one result.
        > 
        > → 이걸 주의하면서 자 이제 Suppose we have only 4 departments(SALES, ACCOUNT, RESEARCH, OPERATION) in DEPT table. Show [1] the total salaries in the "SALES" and "ACCOUNT" deparments combined and [2] in the "RESEARCH" and "OPERATION" departments combined.의 결과를 출력하는 쿼리를 작성해줘  쿼리 결과만 주면 돼.
        > 


- 프롬프트 끝
  
- 결과
    
        SELECT 'SALES & ACCOUNT' AS Department_Group, SUM(SAL) AS Total_Salary
        FROM EMP E
        JOIN DEPT D ON E.DEPTNO = D.DEPTNO
        WHERE DNAME IN ('SALES', 'ACCOUNT')
        UNION ALL
        SELECT 'RESEARCH & OPERATION' AS Department_Group, SUM(SAL) AS Total_Salary
        FROM EMP E
        JOIN DEPT D ON E.DEPTNO = D.DEPTNO
        WHERE DNAME IN ('RESEARCH', 'OPERATION');
        
    ***분석***
    
    <aside>
    💡 분석
    
    1. 이 쿼리는 'EMP' 테이블과 'DEPT' 테이블을 조인하여 
    2. 'SALES'와 'ACCOUNT' 부서의 직원들의 총 급여를 계산하고, 
    3. 이 결과를 'SALES & ACCOUNT'라는 그룹 이름과 함께 표시합니다. 
    4. 이어서 같은 방식으로 'RESEARCH'와 'OPERATION' 부서의 직원들의 총 급여를 'RESEARCH & OPERATION'이라는 그룹 이름으로 계산하고, 
    5. 두 그룹의 결과를 'UNION ALL'을 통해 하나의 결과 집합으로 결합합니다. 
    </aside>
    

### 다섯 번째 문제

- Show [1] the order date, [2] the number of orders for that date, [3] the number of books ordered, [4] the total price of the orders, [5] the running total of books for the month, [6] the number of books from the same day last week, and [7] all of these totals (from [1] to [6]) should also show for each month and overall.
    1. `Ask ChatGPT to generate a SQL query **without giving any schema information**.`
        
        제시한 쿼리는 다음과 같음.

- 프롬프트 시작
      
        > ****1. Creating Tables****
        Tables are the basic unit of data storage in an Oracle Database. Data is stored in rows and columns. You define a table with a table name, such as employees, and a set of columns. You give each column a column name, such as employee_id, last_name, and job_id; a datatype, such as VARCHAR2, DATE, or NUMBER; and a width. The width can be predetermined by the datatype, as in DATE. If columns are of the NUMBER datatype, define precision and scale instead of width. A row is a collection of column information corresponding to a single record.
        You can specify rules for each column of a table. These rules are called integrity constraints. One example is a NOT NULL integrity constraint. This constraint forces the column to contain a value in every row.
        For example:
        `create table DEPARTMENTS (  
          deptno        number,  
          name          varchar2(50) not null,  
          location      varchar2(50),  
          constraint pk_departments primary key (deptno)  
        );`
         Insert into EditorCode Block 1
        
        Tables can declarative specify relationships between tables, typically referred to as referential integrity. To see how this works we can create a "child" table of the DEPARTMENTS table by including a foreign key in the EMPLOYEES table that references the DEPARTMENTS table. For example:
        `create table EMPLOYEES (  
          empno             number,  
          name              varchar2(50) not null,  
          job               varchar2(50),  
          manager           number,  
          hiredate          date,  
          salary            number(7,2),  
          commission        number(7,2),  
          deptno           number,  
          constraint pk_employees primary key (empno),  
          constraint fk_employees_deptno foreign key (deptno) 
              references DEPARTMENTS (deptno)  
        );`
         Insert into EditorCode Block 2
        
        Foreign keys must reference primary keys, so to create a "child" table the "parent" table must have a primary key for the foreign key to reference.
        ****2. Creating Triggers****
        Triggers are procedures that are stored in the database and are implicitly run, or fired, when something happens. Traditionally, triggers supported the execution of a procedural code, in Oracle procedural SQL is called a PL/SQL block. PL stands for procedural language. When an INSERT, UPDATE, or DELETE occurred on a table or view. Triggers support system and other data events on DATABASE and SCHEMA.
        Triggers are frequently used to automatically populate table primary keys, the trigger examples below show an example trigger to do just this. We will use a built in function to obtain a globallally unique identifier or GUID.
        `create or replace trigger  DEPARTMENTS_BIU
            before insert or update on DEPARTMENTS
            for each row
        begin
            if inserting and :new.deptno is null then
                :new.deptno := to_number(sys_guid(), 
                  'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /
        
        create or replace trigger EMPLOYEES_BIU
            before insert or update on EMPLOYEES
            for each row
        begin
            if inserting and :new.empno is null then
                :new.empno := to_number(sys_guid(), 
                    'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX');
            end if;
        end;
        /`
         Insert into EditorCode Block 3
        
        ****3. Inserting Data****
        Now that we have tables created, and we have triggers to automatically populate our primary keys, we can add data to our tables. Because we have a parent child relationship, with the DEPARTMENTS table as the parent table, and the EMPLOYEES table as the child we will first INSERT a row into the DEPARTMENTS table.
        `insert into departments (name, location) values
           ('Finance','New York');
        
        insert into departments (name, location) values
           ('Development','San Jose');`
         Insert into EditorCode Block 4
        
        Lets verify that the insert was successful by running a SQL SELECT statement to query all columns and all rows of our table.
        `select * from departments;`
         Insert into EditorCode Block 5
        
        You can see that an ID will have been automatically generated. You can now insert into the EMPLOYEES table a new row but you will need to put the generated DEPTID value into your SQL INSERT statement. The examples below show how we can do this using a SQL query, but you could simply enter the department number directly.
        `insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Sam Smith','Programmer', 
            5000, 
          (select deptno 
          from departments 
          where name = 'Development'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Mara Martin','Analyst', 
           6000, 
           (select deptno 
           from departments 
           where name = 'Finance'));
        
        insert into EMPLOYEES 
           (name, job, salary, deptno) 
           values
           ('Yun Yates','Analyst', 
           5500, 
           (select deptno 
           from departments 
           where name = 'Development'));`
         Insert into EditorCode Block 6
        
        ****4. Indexing Columns****
        Typically developers index columns for three major reasons:
        1. To enforce unique values within a column
        2. To improve data access performance
        3. To prevent lock escalation when updating rows of tables that use declarative referential integrity
        When a table is created and a PRIMARY KEY is specified an index is automatically created to enforce the primary key constraint. If you specific UNIQUE for a column when creating a column a unique index is also created. To see the indexes that already exist for a given table you can run the following dictionary query.
        `select table_name "Table", 
               index_name "Index", 
               column_name "Column", 
               column_position "Position"
        from  user_ind_columns 
        where table_name = 'EMPLOYEES' or 
              table_name = 'DEPARTMENTS'
        order by table_name, column_name, column_position`
         Insert into EditorCode Block 7
        
        It is typically good form to index foreign keys, foreign keys are columns in a table that reference another table. In our EMPLOYEES and DEPARTMENTS table example the DEPTNO column in the EMPLOYEE table references the primary key of the DEPARTMENTS table.
        `create index employee_dept_no_fk_idx 
        on employees (deptno)`
         Insert into EditorCode Block 8
        
        We may also determine that the EMPLOYEE table will be frequently searched by the NAME column. To improve the performance searches and to ensure uniqueness we can create a unique index on the EMPLOYEE table NAME column.
        `create unique index employee_ename_idx
        on employees (name)`
         Insert into EditorCode Block 9
        
        Oracle provides many other indexing technologies including function based indexes which can index expressions, such as an upper function, text indexes which can index free form text, bitmapped indexes useful in data warehousing. You can also create indexed organized tables, you can use partition indexes and more. Sometimes it is best to have fewer indexes and take advantage of in memory capabilities. All of these topics are beyond the scope of this basic introduction.
        ****5. Querying Data****
        To select data from a single table it is reasonably easy, simply use the SELECT ... FROM ... WHERE ... ORDER BY ... syntax.
        `select * from employees;`
         Insert into EditorCode Block 10
        
        To query data from two related tables you can join the data
        `select e.name employee,
                   d.name department,
                   e.job,
                   d.location
        from departments d, employees e
        where d.deptno = e.deptno(+)
        order by e.name;`
         Insert into EditorCode Block 11
        
        As an alternative to a join you can use an inline select to query data.
        `select e.name employee,
                  (select name 
                   from departments d 
                   where d.deptno = e.deptno) department,
                   e.job
        from employees e
        order by e.name;`
         Insert into EditorCode Block 12
        
        ****6. Adding Columns****
        You can add additional columns after you have created your table using the ALTER TABLE ... ADD ... syntax. For example:
        `alter table EMPLOYEES 
        add country_code varchar2(2);`
         Insert into EditorCode Block 13
        
        ****7. Querying the Oracle Data Dictionary****
        Table meta data is accessible from the Oracle data dictionary. The following queries show how you can query the data dictionary tables.
        `select table_name, tablespace_name, status
        from user_tables
        where table_Name = 'EMPLOYEES';
        
        select column_id, column_name , data_type
        from user_tab_columns
        where table_Name = 'EMPLOYEES'
        order by column_id;`
         Insert into EditorCode Block 14
        
        ****8. Updating Data****
        You can use SQL to update values in your table, to do this we will use the update clause
        `update employees
        set country_code = 'US';`
         Insert into EditorCode Block 15
        
        The query above will update all rows of the employee table and set the value of country code to US. You can also selectively update just a specific row.
        `update employees
        set commission = 2000
        where  name = 'Sam Smith';`
         Insert into EditorCode Block 16
        
        Lets run a Query to see what our data looks like
        `select name, country_code, salary, commission
        from employees
        order by name;`
         Insert into EditorCode Block 17
        
        ****9. Aggregate Queries****
        You can sum data in tables using aggregate functions. We will use column aliases to rename columns for readability, we will also use the null value function (NVL) to allow us to properly sum columns with null values.
        `select 
              count(*) employee_count,
              sum(salary) total_salary,
              sum(commission) total_commission,
              min(salary + nvl(commission,0)) min_compensation,
              max(salary + nvl(commission,0)) max_compensation
        from employees;`
         Insert into EditorCode Block 18
        
        ****10. Compressing Data****
        As your database grows in size to gigabytes or terabytes and beyond, consider using table compression. Table compression saves disk space and reduces memory use in the buffer cache. Table compression can also speed up query execution during reads. There is, however, a cost in CPU overhead for data loading and DML. Table compression is completely transparent to applications. It is especially useful in online analytical processing (OLAP) systems, where there are lengthy read-only operations, but can also be used in online transaction processing (OLTP) systems.
        You specify table compression with the COMPRESS clause of the CREATE TABLE statement. You can enable compression for an existing table by using this clause in an ALTER TABLE statement. In this case, the only data that is compressed is the data inserted or updated after compression is enabled. Similarly, you can disable table compression for an existing compressed table with the ALTER TABLE...NOCOMPRESS statement. In this case, all data the was already compressed remains compressed, and new data is inserted uncompressed.
        To enable compression for future data use the following syntax.
        `alter table EMPLOYEES compress for oltp; 
        alter table DEPARTMENTS compress for oltp;` 
         Insert into EditorCode Block 19
        
        ****11. Deleting Data****
        You can delete one or more rows from a table using the DELETE syntax. For example to delete a specific row:
        `delete from employees 
        where name = 'Sam Smith';`
         Insert into EditorCode Block 20
        
        ****12. Dropping Tables****
        You can drop tables using the SQL DROP command. Dropping a table will remove all of the rows and drop sub-objects including indexes and triggers. The following DROP statements will drop the departments and employees tables. The optional cascade constraints clause will drop remove constraints thus allowing you to drop database tables in any order.
        `drop table departments cascade constraints;
        drop table employees cascade constraints;`
         Insert into EditorCode Block 21
        
        ****13. Un-dropping Tables****
        If the RECYCLEBIN initialization parameter is set to ON (the default in 10g), then dropping this table will place it in the recycle bin. To see if you can undrop a table run the following data dictionary query:
        `select object_name, 
               original_name, 
               type, 
               can_undrop, 
               can_purge
        from recyclebin;`
         Insert into EditorCode Block 22
        
        To undrop tables we use the flashback command, for example:
        `flashback table DEPARTMENTS to before drop;
        flashback table EMPLOYEES to before drop;
        select count(*) departments 
        from departments;
        select count(*) employees
        from employees;`
         Insert into EditorCode Block 23
        > 
        > 
        > 
        > -> 다음의 문법을 바탕으로  Show [1] the order date, [2] the number of orders for that date, [3] the number of books ordered, [4] the total price of the orders, [5] the running total of books for the month, [6] the number of books from the same day last week, and [7] all of these totals (from [1] to [6]) should also show for each month and overall의 결과를 출력하는 쿼리를 생각해봐 그리고 각각의 과정이 어떻게 이뤄지는지 확인해봐
        >
- 프롬프트 끝
        
- 결과
        
        SELECT 
          order_date, 
          COUNT(order_id) AS num_orders, 
          SUM(quantity) AS books_ordered, 
          SUM(quantity * price) AS total_order_price,
          SUM(SUM(quantity)) OVER (PARTITION BY EXTRACT(MONTH FROM order_date)) AS running_monthly_total,
          (SELECT SUM(quantity) FROM orders WHERE order_date = o.order_date - INTERVAL '7' DAY) AS last_week_same_day_books
        FROM 
          orders o
        GROUP BY 
          order_date
        ORDER BY 
          order_date;
        

---

2. `Ask ChatGPT to generate a SQL query **with schema information**.`

- 프롬프트 시작
  
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE author (
        > author_id NUMBER,
        > author_name VARCHAR2(400),
        > CONSTRAINT pk_author PRIMARY KEY (author_id)
        > );
        > 
        > CREATE TABLE publisher (
        > publisher_id NUMBER,
        > publisher_name VARCHAR2(400),
        > CONSTRAINT pk_publisher PRIMARY KEY (publisher_id)
        > );
        > 
        > CREATE TABLE book_language (
        > language_id NUMBER,
        > language_code VARCHAR2(8),
        > language_name VARCHAR2(50),
        > CONSTRAINT pk_language PRIMARY KEY (language_id)
        > );
        > 
        > CREATE TABLE book (
        > book_id NUMBER,
        > title VARCHAR2(400),
        > isbn13 VARCHAR2(13),
        > language_id NUMBER,
        > num_pages NUMBER,
        > publication_date DATE,
        > publisher_id NUMBER,
        > CONSTRAINT pk_book PRIMARY KEY (book_id),
        > CONSTRAINT fk_book_lang FOREIGN KEY (language_id) REFERENCES book_language (language_id),
        > CONSTRAINT fk_book_pub FOREIGN KEY (publisher_id) REFERENCES publisher (publisher_id)
        > );
        > 
        > CREATE TABLE book_author (
        > book_id NUMBER,
        > author_id NUMBER,
        > CONSTRAINT pk_bookauthor PRIMARY KEY (book_id, author_id),
        > CONSTRAINT fk_ba_book FOREIGN KEY (book_id) REFERENCES book (book_id),
        > CONSTRAINT fk_ba_author FOREIGN KEY (author_id) REFERENCES author (author_id)
        > );
        > 
        > CREATE TABLE address_status (
        > status_id NUMBER,
        > address_status VARCHAR2(30),
        > CONSTRAINT pk_addr_status PRIMARY KEY (status_id)
        > );
        > 
        > CREATE TABLE country (
        > country_id NUMBER,
        > country_name VARCHAR2(200),
        > CONSTRAINT pk_country PRIMARY KEY (country_id)
        > );
        > 
        > CREATE TABLE address (
        > address_id NUMBER,
        > street_number VARCHAR2(10),
        > street_name VARCHAR2(200),
        > city VARCHAR2(100),
        > country_id NUMBER,
        > CONSTRAINT pk_address PRIMARY KEY (address_id),
        > CONSTRAINT fk_addr_ctry FOREIGN KEY (country_id) REFERENCES country (country_id)
        > );
        > 
        > CREATE TABLE customer (
        > customer_id NUMBER,
        > first_name VARCHAR2(200),
        > last_name VARCHAR2(200),
        > email VARCHAR2(350),
        > CONSTRAINT pk_customer PRIMARY KEY (customer_id)
        > );
        > 
        > CREATE TABLE customer_address (
        > customer_id NUMBER,
        > address_id NUMBER,
        > status_id NUMBER,
        > CONSTRAINT pk_custaddr PRIMARY KEY (customer_id, address_id),
        > CONSTRAINT fk_ca_cust FOREIGN KEY (customer_id) REFERENCES customer (customer_id),
        > CONSTRAINT fk_ca_addr FOREIGN KEY (address_id) REFERENCES address (address_id)
        > );
        > 
        > CREATE TABLE shipping_method (
        > method_id NUMBER,
        > method_name VARCHAR2(100),
        > cost DECIMAL(6, 2),
        > CONSTRAINT pk_shipmethod PRIMARY KEY (method_id)
        > );
        > 
        > CREATE SEQUENCE seq_custorder;
        > 
        > CREATE TABLE cust_order (
        > order_id NUMBER,
        > order_date DATE,
        > customer_id NUMBER,
        > shipping_method_id NUMBER,
        > dest_address_id NUMBER,
        > CONSTRAINT pk_custorder PRIMARY KEY (order_id),
        > CONSTRAINT fk_order_cust FOREIGN KEY (customer_id) REFERENCES customer (customer_id),
        > CONSTRAINT fk_order_ship FOREIGN KEY (shipping_method_id) REFERENCES shipping_method (method_id),
        > CONSTRAINT fk_order_addr FOREIGN KEY (dest_address_id) REFERENCES address (address_id)
        > );
        > 
        > CREATE TABLE order_status (
        > status_id NUMBER,
        > status_value VARCHAR2(20),
        > CONSTRAINT pk_orderstatus PRIMARY KEY (status_id)
        > );
        > 
        > CREATE SEQUENCE seq_orderline;
        > 
        > CREATE TABLE order_line (
        > line_id NUMBER,
        > order_id NUMBER,
        > book_id NUMBER,
        > price NUMBER(5, 2),
        > CONSTRAINT pk_orderline PRIMARY KEY (line_id),
        > CONSTRAINT fk_ol_order FOREIGN KEY (order_id) REFERENCES cust_order (order_id),
        > CONSTRAINT fk_ol_book FOREIGN KEY (book_id) REFERENCES book (book_id)
        > );
        > 
        > CREATE SEQUENCE seq_orderhist;
        > 
        > CREATE TABLE order_history (
        > history_id NUMBER,
        > order_id NUMBER,
        > status_id NUMBER,
        > status_date DATE,
        > CONSTRAINT pk_orderhist PRIMARY KEY (history_id),
        > CONSTRAINT fk_oh_order FOREIGN KEY (order_id) REFERENCES cust_order (order_id),
        > CONSTRAINT fk_oh_status FOREIGN KEY (status_id) REFERENCES order_status (status_id)
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **dentify Date and Order Counts:** Select the order date and count the number of orders for that date using **`COUNT()`** aggregated by the order date.
        > 2. **Sum Book Quantities and Total Prices:** Calculate the total number of books ordered and the total price for orders on each date using **`SUM()`**.
        > 3. **Calculate Running Totals:** For the running total of books for the month, use a window function like **`SUM()`** with the **`OVER()`** clause partitioned by the month.
        > 4. **Compare with Previous Week Data:** To get the number of books from the same day last week, you can use a subquery with a date filter subtracting 7 days from the order date.
        > 5. **Aggregate Monthly and Overall Totals:** Use **`GROUP BY`** with rollup or grouping sets to include subtotals for each month and a grand total.
        > 6. **Combine All Results:** Use a combination of GROUP BY and window functions to bring all these data points together.
        > 
        > → 자 이제 Show [1] the order date, [2] the number of orders for that date, [3] the number of books ordered, [4] the total price of the orders, [5] the running total of books for the month, [6] the number of books from the same day last week, and [7] all of these totals (from [1] to [6]) should also show for each month and overall의 결과를 출력하는 쿼리를 작성해줘.
        > 

- 프롬프트 끝

  
- 결과

        WITH OrderDetails AS (
          SELECT 
            o.order_id, 
            o.order_date, 
            ol.book_id, 
            ol.price, 
            EXTRACT(MONTH FROM o.order_date) AS order_month,
            EXTRACT(YEAR FROM o.order_date) AS order_year,
            LAG(o.order_date, 7) OVER (ORDER BY o.order_date) AS last_week_date
          FROM 
            cust_order o
            JOIN order_line ol ON o.order_id = ol.order_id
        ),
        BookTotals AS (
          SELECT 
            order_date, 
            COUNT(DISTINCT order_id) AS num_orders, 
            COUNT(book_id) AS num_books, 
            SUM(price) AS total_price,
            SUM(COUNT(book_id)) OVER (PARTITION BY order_month, order_year ORDER BY order_date) AS running_total_books,
            (SELECT COUNT(book_id) FROM OrderDetails WHERE order_date = last_week_date) AS last_week_books
          FROM 
            OrderDetails
          GROUP BY 
            order_date, 
            order_month, 
            order_year, 
            last_week_date
        )
        SELECT 
          order_date, 
          num_orders, 
          num_books, 
          total_price,
          running_total_books,
          last_week_books
        FROM 
          BookTotals
        ORDER BY 
          order_date;
    

---

3.  `Give **some additional information**(i.e., explanation of table schema, PK/FK relationship, clause you want to use etc.) to ChatGPT to make the SQL query more accurate.`

- 프롬프트 시작
  
        > 아까와 똑같은 가이드라인으로 쿼리를 작성할거야. 이제는 스키마를 알려줄건데, 여기서 주의할 점은 !!!!!!!!!!!!!!!!!!!!!! 무조건 주어진 스키마들만 사용해야 한다는거야!!!!!!!!!!!!!!!!!!!!!!!!!!
        > 
        > 
        > CREATE TABLE DEPT (
        > DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        > DNAME VARCHAR2(14) ,
        > LOC VARCHAR2(13) ) ;
        > 
        > CREATE TABLE EMP(
        > EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        > ENAME VARCHAR2(10),
        > JOB VARCHAR2(9),
        > MGR NUMBER(4),
        > HIREDATE DATE,
        > SAL NUMBER(7,2),
        > COMM NUMBER(7,2),
        > DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
        > 
        > CREATE TABLE BONUS (
        > ENAME VARCHAR2(10) ,
        > JOB VARCHAR2(9)  ,
        > SAL NUMBER,
        > COMM NUMBER
        > ) ;
        > 
        > CREATE TABLE SALGRADE (
        > GRADE NUMBER,
        > LOSAL NUMBER,
        > HISAL NUMBER
        > );
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 너가 쿼리를 짜기위해 step-by-step으로 생각할 과정은 다음과 같아.
        > 
        > 1. **dentify Date and Order Counts:** Select the order date and count the number of orders for that date using **`COUNT()`** aggregated by the order date.
        > 2. **Sum Book Quantities and Total Prices:** Calculate the total number of books ordered and the total price for orders on each date using **`SUM()`**.
        > 3. **Calculate Running Totals:** For the running total of books for the month, use a window function like **`SUM()`** with the **`OVER()`** clause partitioned by the month.
        > 4. **Compare with Previous Week Data:** To get the number of books from the same day last week, you can use a subquery with a date filter subtracting 7 days from the order date.
        > 5. **Aggregate Monthly and Overall Totals:** Use **`GROUP BY`** with rollup or grouping sets to include subtotals for each month and a grand total.
        > 6. **Combine All Results:** Use a combination of GROUP BY and window functions to bring all these data points together.
        > 
        > *주의 사항
        > 
        > 1. 오류가 나지 않게 주의해
        > 2. **Subquery Selection**: The inner subquery selects **`order_date`** and **`order_id`** from **`cust_order`** and joins with **`order_line`** to get the book quantities and total prices per order.
        > 3. **Date Filtering**: A WHERE clause filters orders to a specific range, such as the last year in the example provided with **`ADD_MONTHS(TRUNC(SYSDATE, 'YY'), -12)`**.
        > 4. **Aggregation**: Within the subquery, **`GROUP BY`** is used to aggregate data per order date and order ID to get distinct counts and sums.
        > 5. **Running Total Calculation**: The **`SUM() OVER()`** window function, partitioned by month and ordered by date, calculates the running total of books for each month.
        > 6. **Previous Week Comparison**: The **`LAG()`** function retrieves the number of books from the same day of the previous week.
        > 7. **Grouping and Rollup**: **`GROUP BY ROLLUP`** allows for the creation of subtotal and total rows for each month and the entire result set, respectively.
        > 8. **Final Selection**: The outer query selects the month, order date, number of orders, books ordered, total price, running total, and books from the last week's same day.
        > 9. **Ordering Results**: The **`ORDER BY`** clause at the end ensures the results are ordered by month and date in descending order to present the data cohesively.
        > 
        > → 이걸 주의하면서 Show [1] the order date, [2] the number of orders for that date, [3] the number of books ordered, [4] the total price of the orders, [5] the running total of books for the month, [6] the number of books from the same day last week, and [7] all of these totals (from [1] to [6]) should also show for each month and overall의 결과를 출력하는 쿼리를 작성해줘. 쿼리 결과만 주면 돼. 특히 오류가 나지 않도록 조심해!
        > 

- 프롬프트 끝
  
- 결과
    
        SELECT COALESCE(TO_CHAR(order_date, 'YYYY-MM'), 'Total') AS Order_Month,
               order_date,
               COUNT(DISTINCT order_id) AS Number_of_Orders,
               SUM(quantity) AS Number_of_Books_Ordered,
               SUM(total_price) AS Total_Price_of_Orders,
               SUM(SUM(quantity)) OVER (PARTITION BY TO_CHAR(order_date, 'YYYY-MM') ORDER BY order_date) AS Running_Total_for_Month,
               LAG(SUM(quantity), 7) OVER (ORDER BY order_date) AS Books_Last_Week_Same_Day
        FROM (SELECT o.order_date, 
                     o.order_id, 
                     COUNT(l.book_id) AS quantity, 
                     SUM(l.price) AS total_price
              FROM cust_order o
              LEFT JOIN order_line l ON o.order_id = l.order_id
              WHERE o.order_date >= ADD_MONTHS(TRUNC(SYSDATE, 'YY'), -12) -- Considering last one year for sample
              GROUP BY o.order_date, o.order_id) 
        GROUP BY ROLLUP (TO_CHAR(order_date, 'YYYY-MM'), order_date)
        ORDER BY Order_Month DESC, order_date DESC;
    
    ***분석***
    
    <aside>
    💡 분석
    
    1. **`COALESCE`**와 **`TO_CHAR`**을 사용하여 주문 날짜를 연-월 형식으로 변환하고, 연월 별로 데이터를 집계하며, 결과에 'Total' 열을 포함시킵니다.
    2. **`COUNT(DISTINCT order_id)`**를 사용하여 각 날짜에 해당하는 고유 주문 수를 셉니다.
    3. **`SUM(quantity)`**는 주문된 책의 총 수를 계산하고 **`SUM(total_price)`**는 주문의 총 가격을 계산합니다.
    4. **`SUM(SUM(quantity)) OVER (...)`**를 사용하여 각 월의 주문된 책의 누적 합계를 계산합니다.
    5. **`LAG(SUM(quantity), 7) OVER (...)`**를 사용하여 지난주 동일 요일의 주문된 책의 수를 가져옵니다.
    6. **`GROUP BY ROLLUP`**을 사용하여 월별 및 전체 데이터에 대한 소계와 총계를 생성합니다.
    7. 마지막으로, **`ORDER BY`**를 사용하여 결과를 월과 날짜별로 내림차순 정렬하여 전체 데이터를 깔끔하게 표시합니다.
    </aside>
    

### Conclusion

- CoT를 사용하는 경우 성능이 훨씬 좋아짐을 확인함.
- 프롬프팅이 정말 중요하다는 것을 확인함.
- 자연어로 명령하는 쿼리의 발전 가능성이 무궁무진 할 것이라 생각함.
- 특히 나의 연구 주제인 LLM에서는 hallucination을 방지하기 위해 DB와의 grounding을 중요시하는데, 이러한 부분과 크게 연결되어 연구될 수 있는 발판을 이 과제를 통해 체험하였음.
- 또한 데이터를 다양한 각도에서 분석하고, 의미 있는 정보를 추출하여 비즈니스 인텔리전스와 결정 지원에 어떻게 활용할 수 있는지가 데이터사이언티스트로서 중요한 능력임을 확인함.
