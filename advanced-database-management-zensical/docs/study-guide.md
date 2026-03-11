# IS480 Midterm Study Guide

This guide covers **everything** from Weeks 1–6: database theory (Chapters 2–4) and PL/SQL programming. Use it to review concepts, test yourself, and practice code.

---

## Part 1: Database Theory

### Chapter 2 — Modeling Data in the Organization (E-R Model)

#### Key Concepts

**The E-R Model** is a graphical way to represent data requirements. It shows **entities** (things we store data about), **attributes** (properties of entities), and **relationships** (how entities connect).

**Business Rules** drive the model. They come from interviews, policies, and domain knowledge. Every constraint in your schema traces back to a business rule.

#### Entities

| Term | Definition | Example |
|------|-----------|---------|
| **Entity type** | A class of persons, places, objects, events, or concepts | `Student`, `Course`, `Order` |
| **Entity instance** | One specific occurrence | Student #101 "Andy" |
| **Strong entity** | Has its own primary key, exists independently | `Student` |
| **Weak entity** | Cannot be uniquely identified without its owner entity's key | `Dependent` (needs `Employee` PK) |

!!! tip "How to identify entities"
    Look for nouns in business rules. If something has attributes you need to track and multiple instances exist, it's likely an entity.

#### Attributes

| Type | Description | Example |
|------|-----------|---------|
| **Simple** | Cannot be decomposed | `Age`, `GPA` |
| **Composite** | Can be broken into sub-parts | `Address` → Street, City, State, ZIP |
| **Multivalued** | Can have multiple values | `PhoneNumbers` (a person may have several) |
| **Derived** | Calculated from other attributes | `Age` derived from `DateOfBirth` |
| **Required** | Must have a value (NOT NULL) | `StudentID` |
| **Optional** | May be null | `MiddleName` |
| **Identifier (Key)** | Uniquely identifies each instance | `snum` in Students |

#### Relationships

| Cardinality | Notation | Example |
|-------------|----------|---------|
| **One-to-One (1:1)** | Each A has at most one B | Employee – ParkingSpot |
| **One-to-Many (1:M)** | Each A has many B, each B has one A | Department – Employee |
| **Many-to-Many (M:N)** | Each A has many B and vice versa | Student – Course (via Enrollment) |
| **Unary** | Entity relates to itself | Employee manages Employee |
| **Binary** | Between two entity types | Student – Course |
| **Ternary** | Among three entity types | Doctor – Patient – Treatment |

**Minimum cardinality** = mandatory (1) or optional (0) participation:

- `0` = optional participation (some instances may not participate)
- `1` = mandatory participation (every instance must participate)

**Maximum cardinality** = one (1) or many (M)

**Associative entity** = resolves a M:N relationship into two 1:M relationships. Enrollments is an associative entity between Students and SchClasses.

#### Time-Dependent Data

When you need history (e.g., salary changes over time), add a **time stamp** attribute or create a separate history entity.

#### Self-Test Questions — Chapter 2

??? question "1. What is the difference between an entity type and an entity instance?"
    An **entity type** is the category/class (e.g., Student). An **entity instance** is one specific occurrence (e.g., Student #101 "Andy"). The type is the template; the instance is the data.

??? question "2. A university tracks courses. Each course may have 0 or more sections. Each section belongs to exactly 1 course. What is the cardinality?"
    **One-to-Many (1:M)** from Course to Section. Minimum cardinality: Course has 0 (optional) sections; each Section must have 1 (mandatory) Course.

??? question "3. When should you use an associative entity?"
    When you have a **many-to-many relationship** that needs its own attributes. For example, Enrollment between Student and SchClass needs a `grade` attribute — that attribute belongs to neither Student nor SchClass alone, so it goes on the associative entity.

??? question "4. What makes an entity 'weak'?"
    A weak entity **cannot be uniquely identified by its own attributes alone** — it requires the primary key of its owner (strong) entity. Example: `OrderLine` (identified by `OrderID` + `LineNumber`) depends on `Order`.

??? question "5. A patient has a Name (First, Middle, Last). What type of attribute is Name?"
    **Composite attribute** — it can be decomposed into simpler sub-attributes.

---

### Chapter 3 — The Enhanced E-R Model (EER)

#### Key Concepts

The EER model adds **supertypes, subtypes, and inheritance** to the basic E-R model. This allows you to model entities that share common attributes but also have specialized attributes.

#### Supertypes and Subtypes

```
        ┌────────────┐
        │   PERSON    │  ← Supertype (shared: Name, DOB, Address)
        │             │
        └──────┬──────┘
        ┌──────┼──────┐
   ┌────┴───┐  │  ┌───┴────┐
   │ STUDENT│  │  │EMPLOYEE│  ← Subtypes (specialized attributes)
   │ GPA    │  │  │ Salary │
   │ Major  │  │  │ HireDate│
   └────────┘  │  └────────┘
          ┌────┴────┐
          │ ALUMNUS │
          │ GradYear│
          └─────────┘
```

**Attribute inheritance:** Subtypes automatically receive ALL attributes of their supertype. A Student has Name, DOB, Address (from Person) PLUS GPA, Major (its own).

#### Completeness Constraints

| Constraint | Meaning | Notation |
|-----------|---------|----------|
| **Total** | Every supertype instance MUST be a member of at least one subtype | Double line |
| **Partial** | A supertype instance MAY exist without belonging to any subtype | Single line |

#### Disjointness Constraints

| Constraint | Meaning | Notation |
|-----------|---------|----------|
| **Disjoint** | An instance can belong to AT MOST one subtype | "d" in circle |
| **Overlap** | An instance can belong to MULTIPLE subtypes simultaneously | "o" in circle |

#### Subtype Discriminator

An attribute that determines which subtype an instance belongs to:

- `EmployeeType = 'H'` → Hourly subtype
- `EmployeeType = 'S'` → Salaried subtype

For **overlap**, you may need composite or Boolean discriminators.

#### When to Create Subtypes

Create subtypes when:

1. Some instances have attributes that don't apply to all (e.g., only Hourly employees have HourlyRate)
2. Some instances participate in relationships that don't apply to all
3. Different business rules apply to different categories

**Don't** create subtypes if all instances share the same attributes and relationships.

#### Self-Test Questions — Chapter 3

??? question "1. What does attribute inheritance mean in an EER model?"
    A subtype automatically receives ALL attributes (and typically the identifier) of its supertype. You don't redeclare them — they're inherited. Student inherits PersonID, Name, DOB from Person.

??? question "2. A hospital has PATIENT as a supertype with INPATIENT and OUTPATIENT as subtypes. Every patient must be one or the other, never both. What are the constraints?"
    **Total** (every patient must be in a subtype) and **Disjoint** (can't be both). Notation: double line + "d".

??? question "3. A university PERSON can be a STUDENT, an EMPLOYEE, or both. Not every person is necessarily either. What are the constraints?"
    **Partial** (can be neither) and **Overlap** (can be both). Notation: single line + "o".

??? question "4. What is a subtype discriminator? Give an example."
    An attribute on the supertype that determines subtype membership. Example: `PersonType` where 'S' = Student, 'E' = Employee. For overlap cases with Boolean discriminators: `IsStudent = 'Y'`, `IsEmployee = 'Y'`.

---

### Chapter 4 — Logical Database Design and the Relational Model

#### Key Definitions

| Term | Definition |
|------|-----------|
| **Relation** | A named two-dimensional table of data (= a table) |
| **Tuple** | A row in a relation |
| **Attribute** | A named column of a relation |
| **Schema** | The structure/definition of a relation: name + attributes |
| **Domain** | The set of allowable values for an attribute |

#### Properties of Relations

1. Each relation has a **unique name**
2. Each attribute within a relation has a **unique name**
3. All values in a column are from the **same domain**
4. Each tuple is **distinct** (no duplicate rows)
5. The **order of tuples** is insignificant
6. The **order of attributes** is insignificant
7. Each cell contains a **single atomic value** (no repeating groups)

#### Keys

| Key Type | Definition |
|----------|-----------|
| **Primary Key (PK)** | The candidate key chosen to uniquely identify tuples |
| **Candidate Key** | A minimal set of attributes that uniquely identifies every tuple |
| **Composite Key** | A key made up of two or more attributes |
| **Foreign Key (FK)** | An attribute that references the primary key of another relation |
| **Surrogate Key** | A system-generated key (e.g., auto-increment) with no business meaning |

#### Integrity Constraints

| Constraint | Rule |
|-----------|------|
| **Domain constraint** | Each attribute value must be from its defined domain |
| **Entity integrity** | No primary key attribute may be NULL |
| **Referential integrity** | A foreign key must either match a PK value in the referenced table or be NULL |

**Referential integrity actions** (what happens when a referenced PK is deleted/updated):

| Action | Behavior |
|--------|----------|
| **RESTRICT** | Prevent the delete/update if any FK references exist |
| **CASCADE** | Propagate the delete/update to all referencing rows |
| **SET NULL** | Set the FK to NULL in referencing rows |
| **SET DEFAULT** | Set the FK to its default value |

#### Transforming E-R to Relations

| E-R Construct | Relational Mapping |
|--------------|-------------------|
| **Strong entity** | Create a relation; PK = entity identifier |
| **Weak entity** | Create a relation; PK = owner's PK + weak entity's partial identifier |
| **1:M relationship** | Add FK to the "many" side (child) table |
| **M:N relationship** | Create an associative relation with PKs from both entities as composite PK |
| **1:1 relationship** | Add FK to either side (prefer the mandatory side) |
| **Multivalued attribute** | Create a separate relation with FK back to the owner |
| **Composite attribute** | Include the component simple attributes (not the composite itself) |
| **Supertype/subtype** | Multiple strategies (see below) |

**Supertype/subtype mapping options:**

1. **One relation per supertype** — All attributes (super + sub) in one table; many NULLs
2. **One relation per subtype** — Each subtype gets all inherited + own attributes; redundancy
3. **One relation per type** — Supertype table + separate subtype tables joined by PK (most common)

#### Normalization

**Goal:** Eliminate redundancy and update/insert/delete anomalies.

| Normal Form | Rule |
|-------------|------|
| **1NF** | All attributes are atomic (no repeating groups, no multi-valued cells) |
| **2NF** | 1NF + no partial dependencies (every non-key attribute depends on the FULL primary key) |
| **3NF** | 2NF + no transitive dependencies (non-key attributes don't depend on other non-key attributes) |

**Functional dependency:** A → B means "A determines B" (knowing A uniquely determines B).

!!! example "Normalization Example"
    `OrderLine(OrderID, ProductID, ProductName, Quantity, UnitPrice)`

    - **1NF?** ✅ All atomic
    - **2NF?** ❌ `ProductName` depends only on `ProductID`, not on full PK `(OrderID, ProductID)` — partial dependency
    - **Fix:** Split into `OrderLine(OrderID, ProductID, Quantity)` and `Product(ProductID, ProductName, UnitPrice)`
    - **3NF?** Check: Does any non-key depend on another non-key? If `UnitPrice` depends only on `ProductID` (which is now the PK), we're fine. ✅

#### Self-Test Questions — Chapter 4

??? question "1. What is entity integrity?"
    No attribute participating in the **primary key** of a relation may be NULL. Every row must be uniquely identifiable.

??? question "2. What is the difference between a candidate key and a primary key?"
    A **candidate key** is any minimal set of attributes that uniquely identifies tuples. The **primary key** is the one candidate key you choose for the table. A table may have multiple candidate keys but only one primary key.

??? question "3. A table ORDER(OrderID, CustomerID, CustomerName, OrderDate) has a transitive dependency. Identify it and fix it."
    `CustomerName` depends on `CustomerID`, not on the PK `OrderID`. This is a **transitive dependency** (OrderID → CustomerID → CustomerName). Fix: create `Customer(CustomerID, CustomerName)` and keep `ORDER(OrderID, CustomerID, OrderDate)`.

??? question "4. You have a M:N relationship between Student and Course. How do you map it to relations?"
    Create an **associative relation** (e.g., `Enrollment`) with a composite PK of `(StudentID, CourseID)`, each as a FK. Any relationship attributes (like `Grade`) go in this table.

??? question "5. What referential integrity action would you choose if deleting a Customer should also delete all their Orders?"
    **CASCADE** — the delete propagates to all referencing rows in the Orders table.

---

## Part 2: PL/SQL Programming

### Topic 1: PL/SQL Basics

#### Block Structure

Every PL/SQL program is a block:

```sql
DECLARE    -- (optional) variables, cursors, types
BEGIN      -- (required) executable statements
EXCEPTION  -- (optional) error handlers
END;
/
```

#### Variables and `%TYPE`

```sql
DECLARE
    v_name    Students.sname%TYPE;    -- anchored to column type
    v_count   NUMBER := 0;           -- initialized
    v_active  BOOLEAN := TRUE;       -- PL/SQL only (can't print)
BEGIN
    SELECT sname INTO v_name FROM Students WHERE snum = 101;
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_name);
END;
/
```

!!! warning "Always use `%TYPE`"
    This is a requirement for your project. It makes your code resistant to column type changes.

#### `SELECT INTO`

Fetches **exactly one row** into variables:

```sql
SELECT sname, gpa INTO v_name, v_gpa
FROM Students WHERE snum = 101;
```

- 0 rows → `NO_DATA_FOUND`
- 2+ rows → `TOO_MANY_ROWS`

---

### Topic 2: Conditional Logic

#### IF / ELSIF / ELSE

```sql
IF v_gpa >= 3.7 THEN
    v_honors := 'Summa Cum Laude';
ELSIF v_gpa >= 3.5 THEN
    v_honors := 'Magna Cum Laude';
ELSIF v_gpa >= 3.0 THEN
    v_honors := 'Cum Laude';
ELSE
    v_honors := 'No honors';
END IF;
```

#### CASE Statement

```sql
v_grade_value := CASE v_grade
    WHEN 'A' THEN 4
    WHEN 'B' THEN 3
    WHEN 'C' THEN 2
    WHEN 'D' THEN 1
    ELSE 0
END;
```

---

### Topic 3: Loops

#### Basic LOOP (with EXIT WHEN)

```sql
LOOP
    FETCH c_students INTO v_name;
    EXIT WHEN c_students%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(v_name);
END LOOP;
```

#### FOR Loop

```sql
FOR i IN 1..10 LOOP
    DBMS_OUTPUT.PUT_LINE('Iteration: ' || i);
END LOOP;
```

#### Cursor FOR Loop

```sql
FOR rec IN (SELECT sname, gpa FROM Students) LOOP
    DBMS_OUTPUT.PUT_LINE(rec.sname || ': ' || rec.gpa);
END LOOP;
```

---

### Topic 4: SQL in PL/SQL

You can use DML directly in PL/SQL:

```sql
BEGIN
    INSERT INTO Enrollments (classnum, snum) VALUES (10125, 108);

    UPDATE Students SET gpa = 3.5 WHERE snum = 101;

    DELETE FROM Enrollments WHERE snum = 108 AND classnum = 10125;

    COMMIT;
END;
/
```

#### `COUNT(*)` for Existence Checks

```sql
DECLARE
    v_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_count
    FROM Students WHERE snum = 999;

    IF v_count = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Student not found');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Student exists');
    END IF;
END;
/
```

---

### Topic 5: Procedures and Functions

#### Procedure (does something)

```sql
CREATE OR REPLACE PROCEDURE enroll_student (
    p_snum     Students.snum%TYPE,
    p_classnum SchClasses.classnum%TYPE
) AS
    v_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_count
    FROM Enrollments WHERE snum = p_snum AND classnum = p_classnum;

    IF v_count > 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Already enrolled');
    END IF;

    INSERT INTO Enrollments (classnum, snum) VALUES (p_classnum, p_snum);
    DBMS_OUTPUT.PUT_LINE('Enrolled ' || p_snum || ' into ' || p_classnum);
END;
/
```

#### Function (returns a value)

```sql
CREATE OR REPLACE FUNCTION get_letter_grade (
    p_gpa Students.gpa%TYPE
) RETURN VARCHAR2 AS
BEGIN
    RETURN CASE
        WHEN p_gpa >= 3.7 THEN 'A'
        WHEN p_gpa >= 2.7 THEN 'B'
        WHEN p_gpa >= 1.7 THEN 'C'
        WHEN p_gpa >= 0.7 THEN 'D'
        ELSE 'F'
    END;
END;
/
```

| | Procedure | Function |
|-|-----------|----------|
| **Returns** | Nothing (or via OUT params) | A single value |
| **Called with** | `EXEC proc_name()` or `BEGIN proc_name(); END;` | `SELECT func() FROM dual;` or `v := func();` |
| **Use in SQL** | ❌ Cannot | ✅ Can |
| **Purpose** | Do something (INSERT, UPDATE, print) | Compute and return something |

---

### Topic 6: Cursors

#### Explicit Cursor (OPEN / FETCH / EXIT WHEN / CLOSE)

```sql
DECLARE
    CURSOR c_students IS
        SELECT snum, sname, gpa FROM Students WHERE standing = 4;
    v_snum  Students.snum%TYPE;
    v_sname Students.sname%TYPE;
    v_gpa   Students.gpa%TYPE;
BEGIN
    OPEN c_students;
    LOOP
        FETCH c_students INTO v_snum, v_sname, v_gpa;
        EXIT WHEN c_students%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_sname || ' - GPA: ' || v_gpa);
    END LOOP;
    CLOSE c_students;
END;
/
```

#### Cursor Attributes

| Attribute | Meaning |
|-----------|---------|
| `%FOUND` | Last FETCH returned a row |
| `%NOTFOUND` | Last FETCH returned no row |
| `%ROWCOUNT` | Number of rows fetched so far |
| `%ISOPEN` | Cursor is currently open |

#### Cursor FOR Loop (simplified — auto OPEN/FETCH/CLOSE)

```sql
BEGIN
    FOR rec IN (SELECT sname, gpa FROM Students WHERE standing >= 3) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.sname || ': ' || rec.gpa);
    END LOOP;
END;
/
```

#### Parameterized Cursor

```sql
DECLARE
    CURSOR c_by_standing (p_standing NUMBER) IS
        SELECT sname, gpa FROM Students WHERE standing = p_standing;
BEGIN
    FOR rec IN c_by_standing(4) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.sname);
    END LOOP;
END;
/
```

---

### Topic 7: Exception Handling

#### Predefined Exceptions

| Exception | When It Fires |
|-----------|--------------|
| `NO_DATA_FOUND` | `SELECT INTO` returns 0 rows |
| `TOO_MANY_ROWS` | `SELECT INTO` returns 2+ rows |
| `ZERO_DIVIDE` | Division by zero |
| `VALUE_ERROR` | Type conversion or size error |
| `DUP_VAL_ON_INDEX` | Unique constraint violation |

#### Handling Exceptions

```sql
BEGIN
    SELECT sname INTO v_name FROM Students WHERE snum = 999;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Student not found');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Multiple students found');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
END;
/
```

#### `RAISE_APPLICATION_ERROR`

Raises a custom error with a code and message:

```sql
IF v_count = 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'Student ' || p_snum || ' does not exist.');
END IF;
```

- Error codes must be between **-20000 and -20999**
- Unlike `DBMS_OUTPUT`, this **stops execution** and propagates to the caller
- Use `DBMS_OUTPUT` for informational messages; use `RAISE_APPLICATION_ERROR` for actual errors

---

### Topic 8: Packages

A package groups related procedures, functions, and types together.

#### Specification (public interface)

```sql
CREATE OR REPLACE PACKAGE enrollment_pkg AS
    PROCEDURE enroll_student(p_snum NUMBER, p_classnum NUMBER);
    FUNCTION get_student_gpa(p_snum NUMBER) RETURN NUMBER;
END enrollment_pkg;
/
```

#### Body (implementation)

```sql
CREATE OR REPLACE PACKAGE BODY enrollment_pkg AS

    PROCEDURE enroll_student(p_snum NUMBER, p_classnum NUMBER) AS
    BEGIN
        INSERT INTO Enrollments (classnum, snum) VALUES (p_classnum, p_snum);
    END;

    FUNCTION get_student_gpa(p_snum NUMBER) RETURN NUMBER AS
        v_gpa Students.gpa%TYPE;
    BEGIN
        SELECT gpa INTO v_gpa FROM Students WHERE snum = p_snum;
        RETURN v_gpa;
    END;

END enrollment_pkg;
/
```

#### Calling Package Members

```sql
EXEC enrollment_pkg.enroll_student(108, 10125);

SELECT enrollment_pkg.get_student_gpa(101) FROM dual;
```

---

## Part 3: Practice Exercises

### Exercise 1: E-R Modeling

**Scenario:** A car rental company tracks Customers, Cars, and Rentals. Each rental involves one customer and one car. A customer can have many rentals. A car can be rented many times (but only once at a time). Each rental has a start date, end date, and total cost.

**Tasks:**

1. Identify all entity types
2. List attributes for each entity (mark PKs and FKs)
3. Identify all relationships with cardinalities
4. Is Rental a strong entity or an associative entity? Why?

??? success "Solution"
    **Entities:** Customer, Car, Rental

    **Attributes:**

    - Customer(CustomerID [PK], Name, Phone, LicenseNumber)
    - Car(CarID [PK], Make, Model, Year, DailyRate, Status)
    - Rental(RentalID [PK], CustomerID [FK], CarID [FK], StartDate, EndDate, TotalCost)

    **Relationships:**

    - Customer 1:M Rental (one customer, many rentals)
    - Car 1:M Rental (one car, many rentals over time)

    Rental is an **associative entity** resolving the M:N between Customer and Car, but it has its own PK (RentalID) making it a strong entity too. It has its own attributes (StartDate, EndDate, TotalCost) that belong to the rental event itself.

### Exercise 2: Normalization

**Given:** `INVOICE(InvoiceID, CustomerName, CustomerPhone, ProductID, ProductName, Quantity, UnitPrice)`

Normalize to 3NF.

??? success "Solution"
    **1NF:** Already atomic ✅

    **2NF check (PK = InvoiceID + ProductID as composite):**

    - `CustomerName`, `CustomerPhone` depend only on InvoiceID → **partial dependency** ❌
    - `ProductName`, `UnitPrice` depend only on ProductID → **partial dependency** ❌

    **Fix → 2NF:**

    - `Invoice(InvoiceID, CustomerName, CustomerPhone)`
    - `Product(ProductID, ProductName, UnitPrice)`
    - `InvoiceLine(InvoiceID, ProductID, Quantity)`

    **3NF check:** Does `CustomerPhone` depend on `CustomerName`? If customers are uniquely identified by name, that's a transitive dependency. Better:

    - `Customer(CustomerID, CustomerName, CustomerPhone)`
    - `Invoice(InvoiceID, CustomerID)`
    - `Product(ProductID, ProductName, UnitPrice)`
    - `InvoiceLine(InvoiceID, ProductID, Quantity)`

    Now in **3NF** ✅

### Exercise 3: PL/SQL Procedure

Write a procedure `drop_class` that removes a student from a class. It should:

- Validate the enrollment exists (COUNT(*) check)
- Delete the enrollment if found
- Raise an error if not found
- Print a confirmation message

??? success "Solution"
    ```sql
    CREATE OR REPLACE PROCEDURE drop_class (
        p_snum     Students.snum%TYPE,
        p_classnum SchClasses.classnum%TYPE
    ) AS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*) INTO v_count
        FROM Enrollments
        WHERE snum = p_snum AND classnum = p_classnum;

        IF v_count = 0 THEN
            RAISE_APPLICATION_ERROR(-20001,
                'Student ' || p_snum || ' is not enrolled in class ' || p_classnum);
        END IF;

        DELETE FROM Enrollments
        WHERE snum = p_snum AND classnum = p_classnum;

        DBMS_OUTPUT.PUT_LINE('Successfully dropped student '
            || p_snum || ' from class ' || p_classnum);
        COMMIT;
    END;
    /
    ```

### Exercise 4: Cursor Practice

Write a block that uses an **explicit cursor** to print every student's name and their number of enrolled classes. Use OPEN / FETCH / EXIT WHEN / CLOSE.

??? success "Solution"
    ```sql
    DECLARE
        CURSOR c_students IS
            SELECT s.snum, s.sname, COUNT(e.classnum) AS class_count
            FROM Students s
            LEFT JOIN Enrollments e ON s.snum = e.snum
            GROUP BY s.snum, s.sname
            ORDER BY s.sname;

        v_snum   Students.snum%TYPE;
        v_sname  Students.sname%TYPE;
        v_count  NUMBER;
    BEGIN
        OPEN c_students;
        LOOP
            FETCH c_students INTO v_snum, v_sname, v_count;
            EXIT WHEN c_students%NOTFOUND;
            DBMS_OUTPUT.PUT_LINE(v_sname || ' (' || v_snum || '): '
                || v_count || ' classes');
        END LOOP;
        CLOSE c_students;
    END;
    /
    ```

### Exercise 5: Function with Grading Logic

Write a function `Grading` that takes a student number and returns their letter grade based on GPA. Use the same pattern from class (CASE statement).

| GPA | Grade |
|-----|-------|
| ≥ 3.7 | A |
| ≥ 2.7 | B |
| ≥ 1.7 | C |
| ≥ 0.7 | D |
| < 0.7 | F |
| NULL | N/A |

??? success "Solution"
    ```sql
    CREATE OR REPLACE FUNCTION Grading (
        p_snum Students.snum%TYPE
    ) RETURN VARCHAR2 AS
        v_gpa Students.gpa%TYPE;
    BEGIN
        SELECT gpa INTO v_gpa
        FROM Students
        WHERE snum = p_snum;

        IF v_gpa IS NULL THEN
            RETURN 'N/A';
        END IF;

        RETURN CASE
            WHEN v_gpa >= 3.7 THEN 'A'
            WHEN v_gpa >= 2.7 THEN 'B'
            WHEN v_gpa >= 1.7 THEN 'C'
            WHEN v_gpa >= 0.7 THEN 'D'
            ELSE 'F'
        END;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN 'Student not found';
    END;
    /
    ```

### Exercise 6: Exception Handling

Write a block that attempts to enroll student 999 into class 10125. Handle the following:

- Student doesn't exist → print friendly message
- Class is full (capacity reached) → print friendly message
- Already enrolled → print friendly message

??? success "Solution"
    ```sql
    DECLARE
        v_count     NUMBER;
        v_capacity  SchClasses.capacity%TYPE;
        v_enrolled  NUMBER;
        v_snum      NUMBER := 999;
        v_classnum  NUMBER := 10125;
    BEGIN
        -- Check student exists
        SELECT COUNT(*) INTO v_count
        FROM Students WHERE snum = v_snum;

        IF v_count = 0 THEN
            DBMS_OUTPUT.PUT_LINE('ERROR: Student ' || v_snum || ' does not exist.');
            RETURN;
        END IF;

        -- Check already enrolled
        SELECT COUNT(*) INTO v_count
        FROM Enrollments WHERE snum = v_snum AND classnum = v_classnum;

        IF v_count > 0 THEN
            DBMS_OUTPUT.PUT_LINE('ERROR: Student already enrolled in this class.');
            RETURN;
        END IF;

        -- Check capacity
        SELECT capacity INTO v_capacity
        FROM SchClasses WHERE classnum = v_classnum;

        SELECT COUNT(*) INTO v_enrolled
        FROM Enrollments WHERE classnum = v_classnum;

        IF v_enrolled >= v_capacity THEN
            DBMS_OUTPUT.PUT_LINE('ERROR: Class ' || v_classnum || ' is full ('
                || v_capacity || '/' || v_capacity || ').');
            RETURN;
        END IF;

        -- Enroll
        INSERT INTO Enrollments (classnum, snum) VALUES (v_classnum, v_snum);
        DBMS_OUTPUT.PUT_LINE('Successfully enrolled student ' || v_snum);
        COMMIT;

    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('ERROR: Class ' || v_classnum || ' does not exist.');
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
            ROLLBACK;
    END;
    /
    ```

### Exercise 7: Supertype/Subtype Design

**Scenario:** A bank tracks Accounts. There are three types: CheckingAccount (has OverdraftLimit), SavingsAccount (has InterestRate), and LoanAccount (has LoanTerm, MonthlyPayment). All accounts share AccountID, CustomerID, Balance, and OpenDate. An account is always exactly one type.

1. Draw the EER hierarchy
2. What are the completeness and disjointness constraints?
3. What would the subtype discriminator attribute be?
4. Map this to relational tables (one-relation-per-type approach)

??? success "Solution"
    **1. Hierarchy:** Account (supertype) → Checking, Savings, Loan (subtypes)

    **2. Constraints:**

    - **Total** — every account must be one of the three types
    - **Disjoint** — an account is exactly one type (can't be both checking and loan)

    **3. Discriminator:** `AccountType` with values 'C', 'S', 'L'

    **4. Relational mapping:**

    ```sql
    CREATE TABLE Account (
        AccountID    NUMBER PRIMARY KEY,
        CustomerID   NUMBER NOT NULL,
        Balance      NUMBER(12,2),
        OpenDate     DATE,
        AccountType  CHAR(1) CHECK (AccountType IN ('C','S','L'))
    );

    CREATE TABLE CheckingAccount (
        AccountID      NUMBER PRIMARY KEY REFERENCES Account(AccountID),
        OverdraftLimit NUMBER(10,2)
    );

    CREATE TABLE SavingsAccount (
        AccountID    NUMBER PRIMARY KEY REFERENCES Account(AccountID),
        InterestRate NUMBER(5,4)
    );

    CREATE TABLE LoanAccount (
        AccountID      NUMBER PRIMARY KEY REFERENCES Account(AccountID),
        LoanTerm       NUMBER,
        MonthlyPayment NUMBER(10,2)
    );
    ```

---

## Quick Reference Card

### PL/SQL Syntax Cheat Sheet

```sql
-- Variable with %TYPE
v_name  Students.sname%TYPE;

-- SELECT INTO (one row only)
SELECT col INTO v_var FROM table WHERE condition;

-- COUNT(*) existence check
SELECT COUNT(*) INTO v_count FROM table WHERE condition;
IF v_count = 0 THEN ...

-- Explicit cursor pattern
CURSOR c_name IS SELECT ...;
OPEN c_name;
LOOP
    FETCH c_name INTO v1, v2;
    EXIT WHEN c_name%NOTFOUND;
END LOOP;
CLOSE c_name;

-- Cursor FOR loop
FOR rec IN (SELECT ...) LOOP
    rec.column_name ...
END LOOP;

-- Error: custom
RAISE_APPLICATION_ERROR(-20001, 'message');

-- Error: predefined
EXCEPTION
    WHEN NO_DATA_FOUND THEN ...
    WHEN OTHERS THEN DBMS_OUTPUT.PUT_LINE(SQLERRM);

-- Package structure
CREATE OR REPLACE PACKAGE pkg_name AS
    PROCEDURE proc1(...);
    FUNCTION func1(...) RETURN type;
END pkg_name;
/
CREATE OR REPLACE PACKAGE BODY pkg_name AS
    PROCEDURE proc1(...) AS BEGIN ... END;
    FUNCTION func1(...) RETURN type AS BEGIN ... RETURN val; END;
END pkg_name;
/
```

### Theory Cheat Sheet

```
E-R Model:  Entity → Attributes → Relationships → Cardinality
EER Model:  Supertype/Subtype → Total/Partial → Disjoint/Overlap → Discriminator
Relational: Relation → Tuple → Attribute → Domain → Keys
Integrity:  Domain | Entity (PK ≠ NULL) | Referential (FK → PK)
Normal Forms: 1NF (atomic) → 2NF (no partial deps) → 3NF (no transitive deps)
Mapping:    Strong entity → table | M:N → associative table | 1:M → FK on many side
```

---

*IS480 — Advanced Database Management | CSULB | Spring 2026 | Prof. Jose Pineda*
