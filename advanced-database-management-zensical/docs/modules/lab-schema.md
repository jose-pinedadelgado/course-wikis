# Lab Schema & Dataset

The lab exercises use a **Student Enrollment** database with four tables.

## Entity-Relationship Overview

```
Students ──┐
            ├── Enrollments ──── SchClasses ──── Courses
Students ──┘
```

- A **Student** can enroll in many **SchClasses** (through Enrollments)
- A **SchClass** is a scheduled section of a **Course**
- **Enrollments** is the associative entity linking Students to SchClasses

## Table Definitions

### Students

| Column | Type | Constraints |
|--------|------|-------------|
| `snum` | NUMBER | **PK** |
| `sname` | VARCHAR2(50) | |
| `standing` | NUMBER | CHECK 1-4 |
| `majorid` | VARCHAR2(15) | |
| `major_gpa` | NUMBER | |
| `gpa` | NUMBER | |

### Courses

| Column | Type | Constraints |
|--------|------|-------------|
| `dept` | VARCHAR2(10) | |
| `cnum` | NUMBER | **PK** |
| `ctitle` | VARCHAR2(100) | |
| `crhr` | NUMBER | |
| `standing` | NUMBER | |

### SchClasses

| Column | Type | Constraints |
|--------|------|-------------|
| `classnum` | NUMBER | **PK** |
| `cnum` | NUMBER | **FK → Courses** |
| `class_year` | NUMBER | |
| `semester` | VARCHAR2(10) | |
| `section` | NUMBER | |
| `class_day` | VARCHAR2(50) | |
| `class_time` | VARCHAR2(50) | |
| `room` | VARCHAR2(50) | |
| `instructor` | VARCHAR2(50) | |
| `capacity` | NUMBER | |

### Enrollments

| Column | Type | Constraints |
|--------|------|-------------|
| `classnum` | NUMBER | **PK, FK → SchClasses** |
| `snum` | NUMBER | **PK, FK → Students** |
| `grade` | VARCHAR2(10) | |

## Schema Creation Script

```sql
-- Drop in dependency order
DROP TABLE Enrollments CASCADE CONSTRAINTS PURGE;
DROP TABLE SchClasses CASCADE CONSTRAINTS PURGE;
DROP TABLE Students CASCADE CONSTRAINTS PURGE;
DROP TABLE Courses CASCADE CONSTRAINTS PURGE;

-- Create tables
CREATE TABLE Students (
    snum       NUMBER        CONSTRAINT pk_students PRIMARY KEY,
    sname      VARCHAR2(50),
    standing   NUMBER        CONSTRAINT ck_students_standing CHECK (standing BETWEEN 1 AND 4),
    majorid    VARCHAR2(15),
    major_gpa  NUMBER,
    gpa        NUMBER
);

CREATE TABLE Courses (
    dept       VARCHAR2(10),
    cnum       NUMBER        CONSTRAINT pk_courses PRIMARY KEY,
    ctitle     VARCHAR2(100),
    crhr       NUMBER,
    standing   NUMBER
);

CREATE TABLE SchClasses (
    classnum    NUMBER        CONSTRAINT pk_schclasses PRIMARY KEY,
    cnum        NUMBER        NOT NULL,
    class_year  NUMBER,
    semester    VARCHAR2(10),
    section     NUMBER,
    class_day   VARCHAR2(50),
    class_time  VARCHAR2(50),
    room        VARCHAR2(50),
    instructor  VARCHAR2(50),
    capacity    NUMBER,
    CONSTRAINT fk_schclasses_courses FOREIGN KEY (cnum) REFERENCES Courses(cnum)
);

CREATE TABLE Enrollments (
    classnum  NUMBER        NOT NULL,
    snum      NUMBER        NOT NULL,
    grade     VARCHAR2(10),
    CONSTRAINT pk_enrollments PRIMARY KEY (classnum, snum),
    CONSTRAINT fk_enrollments_schclasses FOREIGN KEY (classnum) REFERENCES SchClasses(classnum),
    CONSTRAINT fk_enrollments_students FOREIGN KEY (snum) REFERENCES Students(snum)
);
```

## Seed Data

```sql
-- Students
INSERT INTO Students VALUES (101, 'Andy',   4, 'IS',   2.8, 3.2);
INSERT INTO Students VALUES (102, 'Betty',  2, NULL,   3.2, NULL);
INSERT INTO Students VALUES (103, 'Cindy',  3, 'IS',   2.5, 3.5);
INSERT INTO Students VALUES (104, 'Dan',    1, 'ACCT', 3.0, 2.9);
INSERT INTO Students VALUES (105, 'Emily',  3, 'IS',   3.4, 3.6);
INSERT INTO Students VALUES (106, 'Frank',  2, 'MKT',  2.2, 2.5);
INSERT INTO Students VALUES (107, 'Grace',  4, 'IS',   3.8, 3.9);
INSERT INTO Students VALUES (108, 'Henry',  1, NULL,   NULL, 2.1);

-- Courses
INSERT INTO Courses VALUES ('IS',  300, 'Intro to MIS',           3, 2);
INSERT INTO Courses VALUES ('IS',  310, 'Statistics',              3, 3);
INSERT INTO Courses VALUES ('IS',  380, 'Database Administration', 3, 3);
INSERT INTO Courses VALUES ('MKT', 320, 'Intro to Marketing',     4, 2);
INSERT INTO Courses VALUES ('IS',  480, 'Advanced Databases',      3, 4);
INSERT INTO Courses VALUES ('ACCT',301, 'Financial Accounting',    3, 2);

-- SchClasses
INSERT INTO SchClasses VALUES (10110, 300, 2025, 'Sp', 1, 'MW',  '800-930',   '222', 'Smith', 45);
INSERT INTO SchClasses VALUES (10115, 300, 2025, 'Sp', 2, 'MW',  '900-1015',  '235', 'Lee',   35);
INSERT INTO SchClasses VALUES (10112, 380, 2025, 'Fa', 1, 'TTh', '1330-1445', '122', 'Jones', 65);
INSERT INTO SchClasses VALUES (10113, 320, 2025, 'Fa', 1, 'MWF', '1000-1100', '122', 'TBA',   65);
INSERT INTO SchClasses VALUES (10120, 310, 2025, 'Sp', 1, 'TTh', '1100-1215', '305', 'Park',  40);
INSERT INTO SchClasses VALUES (10125, 480, 2026, 'Sp', 1, 'MW',  '1400-1515', '210', 'Pineda',30);
INSERT INTO SchClasses VALUES (10130, 301, 2025, 'Fa', 1, 'MWF', '800-850',   '150', 'Adams', 50);
INSERT INTO SchClasses VALUES (10135, 300, 2025, 'Fa', 1, 'TTh', '1500-1615', '222', 'Smith', 45);

-- Enrollments
INSERT INTO Enrollments VALUES (10110, 101, 'F');
INSERT INTO Enrollments VALUES (10110, 102, 'A');
INSERT INTO Enrollments VALUES (10113, 103, 'A');
INSERT INTO Enrollments VALUES (10120, 101, 'B');
INSERT INTO Enrollments VALUES (10112, 101, 'A');
INSERT INTO Enrollments VALUES (10115, 105, 'B');
INSERT INTO Enrollments VALUES (10120, 105, 'A');
INSERT INTO Enrollments VALUES (10130, 104, 'C');
INSERT INTO Enrollments VALUES (10135, 106, NULL);
INSERT INTO Enrollments VALUES (10125, 107, NULL);

COMMIT;
```

!!! tip "Reset Script"
    If your schema gets corrupted during exercises, use the reset script above or ask your instructor for the `Reset_Class_Lab_CurrentSchema.sql` file.
