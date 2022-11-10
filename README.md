# Implementing Table Relations in SQL

## Learning Goals

- Implement a one-to-many relationship using an FOREIGN KEY constraint.
- Implement a many-to-many relationship using FOREIGN KEY constraints along with a composite PRIMARY KEY constraint.
- Implement a many-to-many relationship using FOREIGN KEY constraints, a single column PRIMARY KEY constraint, and
  a composite UNIQUE constraint.
- Implement a one-to-one relationship using a FOREIGN KEY and a UNIQUE constraint.


## Introduction

In this lesson, we use SQL to implement the various relationships between tables.

## Implementing One-To-Many Relationships in SQL

![employee department ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/dept_emp_erd.png
)

We will see how to use SQL to implement the one-to-many relationship between `employee` and `department`.
Recall our sample data:

<table>
<tr>
<th>department</th>
<th>employee</th>
</tr>

<tr>

<td>

| id  | name      | location   |
|-----|-----------|------------|
| 1   | Payroll   | Building A |
| 2   | Marketing | Building B |

</td>

<td>

| id  | first_name | last_name  | salary | department_id |
|-----|------------|------------|--------|---------------|
| 1   | Kiran      | Willow     | 76000  | 2             |
| 2   | Dani       | Elm        | 99000  | 2             |
| 3   | Hao        | Pine       | 45000  | 2             |
| 4   | Tal        | Oak        | 88000  | 1             |
| 5   | Yuri       | Birch      | 150000 | 1             |

</td>


</tr>
</table>

- The  `department_id` column stores a single integer, thus each employee works in one department.
- The `department_id` column is **not** unique. Multiple rows in the `employee` table may have the same value in the `department_id` column,
  thus a department may have multiple employees.

We implement the relationship using a `FOREIGN KEY` constraint on the `department_id` column in the `employee` table:

```sql
DROP TABLE IF EXISTS department CASCADE;

CREATE TABLE department (
  id INTEGER PRIMARY KEY,
  name TEXT,
  location TEXT
);

DROP TABLE IF EXISTS employee;

CREATE TABLE employee (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  salary INTEGER,
  department_id INTEGER NOT NULL,
  CONSTRAINT department_fk FOREIGN KEY (department_id) REFERENCES department (id)
);
```

The foreign key forces the database to check that each value in
the `department_id` column corresponds to a value in the `id` column
of the `department` table.  Thus, the following statement would
result in a foreign key constrain violation as the `department` table
does not contain a row with `id` 10.  The DBMS would prevent the row
from being inserted:

```SQL
INSERT INTO employee (id, first_name, last_name, salary, department_id)
VALUES (6, 'Lee', 'Maple', 75000, 10); 
```

Foreign keys introduce a challenge when it come to dropping tables.
Normally, a DBMS won't let us drop a table if another table
references any of  its rows using a foreign key.
The statement `DROP TABLE IF EXISTS department CASCADE;` tells the DBMS that
dropping the `department` table should result in dropping associated employee
rows from the `employee` table.  There are other techniques to manage
the deletion of rows and tables in the presence of foreign keys.  For simplicity,
we will use the `CASCADE` keyword in the `DROP TABLE` statement.


An alternative approach to define a foreign key is to leave the constraint
out of the `CREATE TABLE` statement and instead use an `ALTER TABLE`
statement to add the foreign key:

```sql
DROP TABLE IF EXISTS department CASCADE;

CREATE TABLE department (
  id INTEGER PRIMARY KEY,
  name TEXT,
  location TEXT
);

DROP TABLE IF EXISTS employee;
CREATE TABLE employee (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  salary INTEGER,
  department_id INTEGER NOT NULL
);

ALTER TABLE employee ADD FOREIGN KEY (department_id) REFERENCES department (id);
```

## Many-To-Many Relationships

We will see how to use SQL to implement the one-to-many relationship between `employee` and `department`.
Recall our sample data:

![author book ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/author_book_erd.png)

<table>
<tr>
<th>Book table</th>
<th>Author table</th>
</tr>

<tr>

<td>

| id  | title                            | year | ISBN           |
|-----|----------------------------------|------|----------------|
| 1   | The C Programming Language       | 1978 | 978-0131101630 |
| 2   | The Unix Programming Environment | 1983 | 978-0139376818 |
| 3   | Unix, A History and Memoir       | 2019 | 978-1695978553 |

</td>

<td>

| id  | author              |
|-----|---------------------|
| 1   | Brian W. Kernighan  |
| 2   | Dennis M. Ritchie   |
| 3   | Rob Pike            | 

</td>
</tr>
</table>


There are two basic techniques for designing a primary key for many-to-many relationships.

<table>
<tr>
<th>Technique #1</th>
<th>Technique #2</th>
</tr>

<tr>
<td>Composite primary key (book_id, author_id) </td>
<td>Primary key (id).  Unique constraint (book_id, author_id) </td>
</tr>

<tr>

<td>
 
| book_id | author_id |
|---------|-----------|
| 1       | 1         |
| 1       | 2         |
| 2       | 1         |
| 2       | 3         |
| 3       | 1         |

</td>

<td>


| id  | book_id | author_id  |
|-----|---------|------------|
| 1   | 1       | 1          |
| 2   | 1       | 2          |
| 3   | 2       | 1          |
| 4   | 2       | 3          |
| 5   | 3       | 1          |


</td>

</tr>

</table>

For technique #1, the two columns `book_id` and `author_id` together form the primary key.

For technique #2, we use an additional `id` column for the primary key, and add a 
constraint that no two rows can contain the same combination of `book_id` and `author_id`.

## Implementing Many-To-Many Relationships in SQL

The SQL statements to create the many-to-many relationship using a composite primary key `(book_id, author_id)`
is shown below.  Both columns are also foreign keys to ensure the relationship references
an existing book and author.

```sql
DROP TABLE IF EXISTS book CASCADE;

CREATE TABLE book (
  id INTEGER PRIMARY KEY,
  title TEXT,
  year INTEGER,
  isbn TEXT
);

DROP TABLE IF EXISTS author CASCADE;

CREATE TABLE author (
  id INTEGER PRIMARY KEY,
  name TEXT
);

DROP TABLE IF EXISTS book_author;

CREATE TABLE book_author (
  book_id INTEGER,
  author_id INTEGER,
  PRIMARY KEY (book_id, author_id),
  CONSTRAINT book_fk FOREIGN KEY (book_id) REFERENCES book (id),
  CONSTRAINT author_fk FOREIGN KEY (author_id) REFERENCES author (id)
);
```

An alternative implementation of the `book_author` relationship table
uses an `id` column for the primary key, along
with a unique constraint for the composite set of columns `(book_id, author_id)`:

```sql
DROP TABLE IF EXISTS book CASCADE;

CREATE TABLE book (
  id INTEGER PRIMARY KEY,
  title TEXT,
  year INTEGER,
  isbn TEXT
);

DROP TABLE IF EXISTS author CASCADE;

CREATE TABLE author (
  id INTEGER PRIMARY KEY,
  name TEXT
);

DROP TABLE IF EXISTS book_author;

CREATE TABLE book_author (
  id INTEGER PRIMARY KEY,
  book_id INTEGER,
  author_id INTEGER,
  CONSTRAINT book_fk FOREIGN KEY (book_id) REFERENCES book (id),
  CONSTRAINT author_fk FOREIGN KEY (author_id) REFERENCES author (id),
  CONSTRAINT book_author_unique UNIQUE (book_id, author_id)
);
```

## Implementing One-To-One Relationships in SQL

Finally, let's see how to implement the one-to-one relationship between `pet` and `fish_physiology`.
The `pet_id` column in the `fish_physiology` table is a foreign key to the `id` column in `pet`.
We limit the cardinality to **one-to-one** by adding a unique constraint on `pet_id`, which
prevents two rows from having the same value in the `pet_id` column.

![fish ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/fish_erd.png)


```sql
DROP TABLE IF EXISTS pet CASCADE;

CREATE TABLE pet (
  id INTEGER PRIMARY KEY,
  name TEXT,
  species TEXT,
  age INTEGER
);

CREATE TABLE fish_physiology (
  id INTEGER PRIMARY KEY,
  habitat TEXT,
  temperature_min INTEGER,
  temperature_max INTEGER,
  pet_id INTEGER UNIQUE, 
  CONSTRAINT pet_fk FOREIGN KEY (pet_id) REFERENCES pet (id)
);
```

***

## Conclusion

The SQL `FOREIGN KEY` constraint is used to control the relationship between tables.
The SQL `UNIQUE` constraint ensures the relationship is one-to-one and not one-to-many.

***


## Resources

- [PostgreSQL CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html)    
- [PostgreSQL DROP TABLE](https://www.postgresql.org/docs/current/sql-droptable.html)   
- [PostgreSQL Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)       
- [PostgreSQL FOREIGN KEY Tutorial](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-foreign-key/)    
- [PostgreSQL UNIQUE Tutorial](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-unique-constraint/)   

