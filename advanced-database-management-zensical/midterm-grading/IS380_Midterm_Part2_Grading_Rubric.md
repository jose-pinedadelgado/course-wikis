# IS 380 Midterm Part 2 — Modeling and Normalizing
## Detailed Grading Rubric & Answer Keys

**Total: 60 points** (20 per problem)

---

## PROBLEM 1: EERD for Happy Tails Animal Shelter (20 points)

### Business Rules Summary
- **PERSON** (PersonID, Email, Phone) — supertype
- **VOLUNTEER** (TrainingLevel) — subtype of Person
- **ADOPTER** (HomeType) — subtype of Person
- Person can be both, neither, or one → **overlapping, partial specialization**
- **ANIMAL** (AnimalID, AnimalName, Species, ArrivalDate)
- **HOUSING_AREA** (HousingAreaID, AreaName)
- Animal → Housing Area: each animal assigned to exactly one (mandatory), housing area may have 0 or many animals → **1:M (mandatory on animal side, optional on housing side)**
- Volunteer ↔ Animal: volunteer cares for 1..M animals, animal cared for by 1..M volunteers → **M:N (mandatory both sides)** → requires associative entity (e.g., CARE_ASSIGNMENT)

### Answer Key

**Entities (5-6):**
1. PERSON (supertype)
2. VOLUNTEER (subtype)
3. ADOPTER (subtype)
4. ANIMAL
5. HOUSING_AREA
6. CARE_ASSIGNMENT (associative entity for Volunteer-Animal M:N) — OR the M:N can be shown directly with crow's foot notation without a separate entity, depending on convention taught

**Attributes:**
- PERSON: PersonID (PK), Email, Phone
- VOLUNTEER: TrainingLevel (+ inherits PersonID as PK/FK)
- ADOPTER: HomeType (+ inherits PersonID as PK/FK)
- ANIMAL: AnimalID (PK), AnimalName, Species, ArrivalDate
- HOUSING_AREA: HousingAreaID (PK), AreaName

**Relationships:**
- PERSON ← supertype/subtype → VOLUNTEER, ADOPTER
- ANIMAL → HOUSING_AREA (M:1, mandatory on animal side)
- VOLUNTEER ↔ ANIMAL (M:N, mandatory both sides) — via associative entity or direct M:N line

**Supertype/Subtype:**
- Discriminator attribute on PERSON (e.g., PersonType or a "d"/"o" circle)
- **Overlapping** constraint (person can be both volunteer and adopter)
- **Partial** specialization (person can be neither) — shown with single line (not double/total)

### Rubric (20 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Entities** | 5 | All entities identified: PERSON, VOLUNTEER, ADOPTER, ANIMAL, HOUSING_AREA, plus associative entity if M:N requires one. No unnecessary entities. | 4: Missing associative entity but all others correct. 3: Missing 1–2 entities. 2: Only 2–3 entities identified. 1: Attempted but mostly wrong. | 0: No entities or completely wrong. |
| **B. Attributes** | 4 | All specified attributes placed on correct entities. PKs identified. No missing attributes, no invented attributes. | 3: Most attributes correct, 1–2 misplaced or missing. 2: Half correct. 1: Few attributes, many errors. | 0: No attributes. |
| **C. Relationships & Cardinality** | 4 | All relationships shown with correct cardinality/optionality: Animal-Housing (M:1, mandatory), Volunteer-Animal (M:N, mandatory both sides). Crow's foot or Chen notation used correctly. | 3: Relationships present but 1 cardinality wrong. 2: Relationships present but multiple cardinality errors. 1: Some lines drawn but cardinalities mostly wrong/missing. | 0: No relationships. |
| **D. Supertype/Subtype** | 4 | Correct supertype (PERSON) with subtypes (VOLUNTEER, ADOPTER). Shows: overlapping constraint (o), partial specialization (single line), subtype discriminator. | 3: Structure correct but wrong constraint label (disjoint instead of overlapping, or total instead of partial). 2: Supertype/subtype present but significant errors. 1: Attempted but fundamentally wrong. | 0: No supertype/subtype. |
| **E. Clean Structure & Naming** | 3 | Clean diagram, readable, proper naming conventions (singular entity names, meaningful attribute names). No unnecessary entities. Professional presentation. | 2: Minor naming issues or slightly messy but readable. 1: Messy, hard to read, inconsistent naming. | 0: Illegible or no diagram. |

### Common Errors to Watch For
- Making VOLUNTEER and ADOPTER separate entities unconnected to PERSON (not a supertype/subtype)
- Using **disjoint** instead of **overlapping** (the prompt says "may be both")
- Using **total** specialization instead of **partial** (the prompt says "may also be neither")
- Missing the M:N between Volunteer and Animal (or showing it as 1:M)
- Making Housing Area an attribute of Animal instead of a separate entity
- Not showing mandatory participation of Animal in Housing Area relationship
- Creating an ADOPTION entity (not required by the prompt — no adoption event described)

---

## PROBLEM 2: Transform EERD to Logical Model — Consulting Firm (20 points)

### Source Conceptual Model (from drawio template)

The ERD shows a consulting firm with:
- **EMPLOYEE** (EmployeeID PK, EmployeeName, Email, EmployeeType?) — supertype
- **CONSULTANT** (Specialty) — subtype of Employee
- **MANAGER** (BonusRate) — subtype of Employee
- Subtype discriminator: "d" (disjoint), appears to be total specialization
- **CLIENT** (ClientID PK, ClientName)
- **PROJECT** (ProjectID PK, ProjectName, StartDate)
- CONSULTANT ↔ PROJECT: M:N relationship via **CONSULTANT_PROJECT** (HoursAssigned)
- CLIENT → PROJECT: "C" and "M" labels suggest 1:M (one client has many projects)
- MANAGER → EMPLOYEE: possible unary (manages relationship) — need to verify from diagram

### Answer Key — Logical Relations

**1. EMPLOYEE** (EmployeeID PK, EmployeeName, Email, EmployeeType)

**2. CONSULTANT** (EmployeeID PK/FK → EMPLOYEE, Specialty)

**3. MANAGER** (EmployeeID PK/FK → EMPLOYEE, BonusRate)

**4. CLIENT** (ClientID PK, ClientName)

**5. PROJECT** (ProjectID PK, ProjectName, StartDate, ClientID FK → CLIENT)
- ClientID placed here because CLIENT:PROJECT is 1:M (FK goes on the "many" side)

**6. CONSULTANT_PROJECT** (EmployeeID FK → CONSULTANT, ProjectID FK → PROJECT, HoursAssigned)
- Composite PK: (EmployeeID, ProjectID)
- This is the associative entity transformation

### Rubric (20 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **A. Relations & Attributes** | 4 | All necessary relations created (5–6 tables). All attributes from ERD included in correct relations. No missing attributes. | 3: Most relations correct, 1–2 attributes misplaced. 2: Half the relations correct. 1: Attempted but mostly wrong. | 0: No relations created. |
| **B. Primary Keys** | 4 | PK correctly identified for every relation. Composite PK for CONSULTANT_PROJECT (EmployeeID, ProjectID). Subtypes use supertype PK as their PK. | 3: Most PKs correct, 1 composite key error. 2: Some PKs identified but errors. 1: Few PKs, mostly wrong. | 0: No PKs. |
| **C. Relationship Transformation (FK placement)** | 4 | FKs correctly placed: ClientID in PROJECT (1:M), EmployeeID in subtypes (supertype reference). FK labels or references shown. | 3: Most FKs correct, 1 wrong placement. 2: Some FKs but direction errors. 1: FKs attempted but mostly wrong. | 0: No FKs. |
| **D. Supertype/Subtype Transformation** | 4 | Subtypes (CONSULTANT, MANAGER) correctly reference supertype (EMPLOYEE) via EmployeeID as PK/FK. EmployeeType discriminator in EMPLOYEE table. All subtype-specific attributes in correct subtype tables. | 3: Structure mostly correct, minor error (e.g., missing discriminator). 2: Attempted but significant errors (e.g., merging everything into one table). 1: Wrong approach. | 0: No transformation attempted. |
| **E. Special Structures (M:N / Associative)** | 4 | CONSULTANT_PROJECT correctly transformed into a relation with composite PK and two FKs. HoursAssigned attribute included. Any other special structures (multivalued attributes, unary relationships) handled correctly. | 3: Associative entity transformed but minor error (e.g., missing HoursAssigned or wrong PK). 2: Attempted but composite key wrong. 1: Barely attempted. | 0: Not transformed. |

### Common Errors to Watch For
- Putting ClientID in CLIENT table as FK to PROJECT (backwards — FK goes on the "many" side)
- Not creating CONSULTANT_PROJECT as a separate table (just showing M:N line)
- Missing composite PK on CONSULTANT_PROJECT
- Merging EMPLOYEE + CONSULTANT + MANAGER into one table (wrong transformation approach)
- Not including EmployeeType discriminator in EMPLOYEE
- Subtypes using their own separate PK instead of inheriting EmployeeID

---

## PROBLEM 3: Normalization — PacoDogShop Training Recommendations (20 points)

### Source Table (UNF/1NF)

| PetTypeID | PetTypeName | TrainingTypeID | TrainingTypeName | AccessoryID | AccessoryName | TrainerID | TrainerName | RecommendedQty |
|-----------|-------------|----------------|------------------|-------------|---------------|-----------|-------------|----------------|

**Composite PK:** (PetTypeID, TrainingTypeID, AccessoryID)

### Functional Dependencies (from assumptions)

- PetTypeID → PetTypeName
- TrainingTypeID → TrainingTypeName, TrainerID
- TrainerID → TrainerName
- AccessoryID → AccessoryName
- (PetTypeID, TrainingTypeID, AccessoryID) → RecommendedQty

### Answer Key

**Q1: Current Normal Form (4 pts)**
The table is in **1NF** (assuming no repeating groups/multi-valued cells). It is NOT in 2NF because there are partial dependencies (non-key attributes depend on only part of the composite key).

**Q2: Full & Partial Dependencies (4 pts)**

Full dependency:
- (PetTypeID, TrainingTypeID, AccessoryID) → RecommendedQty

Partial dependencies:
- PetTypeID → PetTypeName (depends on only part of the PK)
- TrainingTypeID → TrainingTypeName, TrainerID (depends on only part of the PK)
- AccessoryID → AccessoryName (depends on only part of the PK)

**Q3: 2NF Model (4 pts)**

Remove partial dependencies by creating separate tables:

**PET_TYPE** (PetTypeID PK, PetTypeName)

**TRAINING_TYPE** (TrainingTypeID PK, TrainingTypeName, TrainerID, TrainerName)

**ACCESSORY** (AccessoryID PK, AccessoryName)

**TRAINING_RECOMMENDATION** (PetTypeID FK, TrainingTypeID FK, AccessoryID FK, RecommendedQty)
- Composite PK: (PetTypeID, TrainingTypeID, AccessoryID)

**Q4: Transitive Dependencies (4 pts)**

Yes, there is a transitive dependency in TRAINING_TYPE:
- TrainingTypeID → TrainerID → TrainerName
- TrainerName depends on TrainerID, not directly on the PK TrainingTypeID

**Q5: 3NF Model (4 pts)**

Remove transitive dependency:

**PET_TYPE** (PetTypeID PK, PetTypeName)

**TRAINING_TYPE** (TrainingTypeID PK, TrainingTypeName, TrainerID FK)

**TRAINER** (TrainerID PK, TrainerName)

**ACCESSORY** (AccessoryID PK, AccessoryName)

**TRAINING_RECOMMENDATION** (PetTypeID FK, TrainingTypeID FK, AccessoryID FK, RecommendedQty)
- Composite PK: (PetTypeID, TrainingTypeID, AccessoryID)

### Rubric (20 points)

| Criterion | Points | Full Credit | Partial Credit | No Credit |
|-----------|--------|-------------|----------------|-----------|
| **Q1. Current Normal Form** | 4 | Correctly identifies as 1NF (or UNF if repeating groups are assumed). Explains why it's not 2NF (partial dependencies exist). | 3: Correct form identified but weak/no explanation. 2: Says "not normalized" or "0NF" without precision. 1: Wrong form but shows some understanding. | 0: No answer or completely wrong. |
| **Q2. Dependencies** | 4 | Lists all partial dependencies correctly: PetTypeID→PetTypeName, TrainingTypeID→TrainingTypeName+TrainerID, AccessoryID→AccessoryName. Shows the full dependency for RecommendedQty. | 3: Most dependencies correct, 1 missing. 2: Some dependencies identified but incomplete or with errors. 1: Attempted but mostly wrong. | 0: No dependencies listed. |
| **Q3. 2NF Model** | 4 | Correctly decomposes into 4 tables removing all partial dependencies. PKs and FKs correct. TRAINING_RECOMMENDATION retains composite PK. TrainerID and TrainerName stay together in TRAINING_TYPE (still has transitive dep — that's OK for 2NF). | 3: Mostly correct, 1 table wrong or missing attribute. 2: Attempted decomposition but significant errors. 1: Wrong approach. | 0: No 2NF model. |
| **Q4. Transitive Dependencies** | 4 | Correctly identifies TrainingTypeID → TrainerID → TrainerName as transitive. Explains that TrainerName depends on TrainerID, not on the PK. | 3: Identifies the transitive dependency but explanation is weak. 2: Partially correct identification. 1: Attempted but wrong. | 0: No answer. |
| **Q5. 3NF Model** | 4 | Correctly creates TRAINER table (TrainerID PK, TrainerName). TRAINING_TYPE now has TrainerID as FK. All other tables remain. 5 tables total. | 3: Correct idea but minor error (e.g., missing FK notation). 2: Attempted but wrong decomposition. 1: Barely attempted. | 0: No 3NF model. |

### Common Errors to Watch For
- Saying the table is in "0NF" or "UNF" — acceptable if they explain repeating groups, but if the data is already flat (no multi-valued cells), it's 1NF
- Not identifying PetTypeID → PetTypeName as a partial dependency
- Removing TrainerID from TRAINING_TYPE in 2NF (premature — that's a 3NF step)
- Not creating TRAINER table in 3NF
- Losing RecommendedQty or not knowing where it belongs
- Making RecommendedQty part of a PK (it's a non-key attribute)
- Not maintaining the composite PK in TRAINING_RECOMMENDATION

---

## GRADING GUIDELINES

### Diagram vs. Text Submissions
- Some students submit `.drawio` diagrams, others submit `.docx`/`.pdf` with text or screenshots
- Grade based on content, not format — a correct text-based relational schema is as valid as a drawio diagram
- For Problem 3, the docx answers (Q1, Q4) should be in text; the models (Q3, Q5) may be in drawio or text

### Notation Flexibility
- Accept both **crow's foot** and **Chen** notation for ERDs
- Accept both **underline** and **PK label** notation for primary keys
- Accept **FK** labels, arrows, or written references for foreign keys

### Partial Credit Philosophy
- Correct structure with wrong labels → give most credit
- Wrong structure with correct labels → give less credit
- A student who shows understanding of the concept but makes execution errors deserves more credit than one who copies without understanding
