# Normalization: from the raw CSV to 3NF

This document shows how the wide file `data/raw/insurance_claims.csv` (1,000 records, 40 columns) becomes related tables in Third Normal Form. All example rows are the **first three real records** of the file (policy numbers 521585, 342868, 687698). Nothing is cleaned here; cleaning is done in Phase IV.

**Design decision about the key.** In the file every `policy_number` is unique, so each row holds one policy with one incident. In the real business a policy can have **several incidents on different dates**, so the key used for normalization is the pair **(policy_number, incident_date)**. This is what makes partial dependencies visible in 2NF.

## Step 1: Entities identified (Task 3)

| Entity | Why it is separate |
|---|---|
| INSURED | The person insured; one person can hold many policies |
| OCCUPATION | 14 repeated job values |
| HOBBY | 20 repeated hobby values |
| AUTO_MAKE | 14 repeated car brands |
| AUTO_MODEL | 39 car models; each belongs to exactly one make |
| VEHICLE | The insured car |
| POLICY | The insurance contract |
| INCIDENT | The accident; a policy can have many incidents |
| CLAIM | The money claimed; an incident can lead to more than one claim |
| CLAIM_PAYMENT | Many payments can be made on one claim (created by PL/SQL, not in the CSV) |
| AUDIT_LOG | Record of every change (written by a trigger, not in the CSV) |

There is no many-to-many relationship in this data, so no junction table is needed. The design has **11 related tables**.

## Step 2: UNF (unnormalised form)

The raw file is one wide table. Only some columns are shown here.

| policy_number | policy_csl | age | insured_occupation | auto_make | auto_model | incident_type | injury_claim | property_claim | vehicle_claim | total_claim_amount | fraud_reported | _c39 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 521585 | 250/500 | 48 | craft-repair | Saab | 92x | Single Vehicle Collision | 6510 | 13020 | 52080 | 71610 | Y | (empty) |
| 342868 | 250/500 | 42 | machine-op-inspct | Mercedes | E400 | Vehicle Theft | 780 | 780 | 3510 | 5070 | Y | (empty) |
| 687698 | 100/300 | 29 | sales | Dodge | RAM | Multi-vehicle Collision | 7700 | 3850 | 23100 | 34650 | N | (empty) |

**Problems:** `policy_csl` holds two numbers in one cell (not atomic), `_c39` is an empty extra column, and policy, insured, vehicle, incident and claim facts are all mixed in one row.

## Step 3: 1NF

| policy_number | incident_date | csl_per_person | csl_per_accident | age | insured_occupation | auto_make | auto_model | incident_type | injury_claim | property_claim | vehicle_claim | total_claim_amount | fraud_reported |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 521585 | 2015-01-25 | 250 | 500 | 48 | craft-repair | Saab | 92x | Single Vehicle Collision | 6510 | 13020 | 52080 | 71610 | Y |
| 342868 | 2015-01-21 | 250 | 500 | 42 | machine-op-inspct | Mercedes | E400 | Vehicle Theft | 780 | 780 | 3510 | 5070 | Y |
| 687698 | 2015-02-22 | 100 | 300 | 29 | sales | Dodge | RAM | Multi-vehicle Collision | 7700 | 3850 | 23100 | 34650 | N |

- **Repeating/non-atomic value removed:** `policy_csl` (for example `250/500`) is split into two single-value columns, `csl_per_person` and `csl_per_accident`.
- The empty column `_c39` is dropped because it holds no data.
- Every column now holds one value. **Key: (policy_number, incident_date).**

## Step 4: 2NF

**Partial dependency removed:** the policy, insured and vehicle facts depend only on `policy_number` (one part of the key), so they move to a table of their own. The incident and claim facts depend on the whole key `(policy_number, incident_date)` and stay together.

**POLICY_DETAILS** (key: policy_number)

| policy_number | csl_per_person | csl_per_accident | age | insured_occupation | auto_make | auto_model | auto_year |
|---|---|---|---|---|---|---|---|
| 521585 | 250 | 500 | 48 | craft-repair | Saab | 92x | 2004 |
| 342868 | 250 | 500 | 42 | machine-op-inspct | Mercedes | E400 | 2007 |
| 687698 | 100 | 300 | 29 | sales | Dodge | RAM | 2007 |

**INCIDENT_CLAIM** (key: policy_number + incident_date)

| policy_number | incident_date | incident_type | injury_claim | property_claim | vehicle_claim | total_claim_amount | fraud_reported |
|---|---|---|---|---|---|---|---|
| 521585 | 2015-01-25 | Single Vehicle Collision | 6510 | 13020 | 52080 | 71610 | Y |
| 342868 | 2015-01-21 | Vehicle Theft | 780 | 780 | 3510 | 5070 | Y |
| 687698 | 2015-02-22 | Multi-vehicle Collision | 7700 | 3850 | 23100 | 34650 | N |

## Step 5: 3NF

**Transitive dependencies removed (one sentence each):**

1. `auto_model` determines `auto_make` (each of the 39 models belongs to exactly one make), so models go to AUTO_MODEL and makes to AUTO_MAKE.
2. Policy facts about the car (`auto_year`, model) depend on the vehicle and not directly on the policy, so they move to VEHICLE.
3. Facts about the person (age, sex, education, zip, relationship, months as customer, capital gains and loss) depend on the insured and not directly on the policy, so they move to INSURED.
4. Repeated text values `insured_occupation` and `insured_hobbies` are stored once in OCCUPATION and HOBBY (repeated values, so no spelling can differ).
5. `total_claim_amount` depends on `injury_claim + property_claim + vehicle_claim` (non-key depends on non-key), so it is removed and calculated when needed.
6. Claim amounts and the fraud flag describe the claim and not the incident, so they move to CLAIM, which has its own key.

**Resulting tables** (surrogate keys `*_id` are new, because the file has no IDs for insured people, vehicles, incidents or claims):

**OCCUPATION**

| occupation_id | occupation_name |
|---|---|
| 3 | craft-repair |
| 7 | machine-op-inspct |
| 12 | sales |

**HOBBY**

| hobby_id | hobby_name |
|---|---|
| 18 | sleeping |
| 16 | reading |
| 3 | board-games |

**AUTO_MAKE**

| make_id | make_name |
|---|---|
| 11 | Saab |
| 9 | Mercedes |
| 5 | Dodge |

**AUTO_MODEL** (make_id is a foreign key)

| model_id | model_name | make_id |
|---|---|---|
| 29 | 92x | 11 |
| 24 | E400 | 9 |
| 14 | RAM | 5 |

**INSURED** (occupation_id and hobby_id are foreign keys)

| insured_id | age | insured_sex | insured_education_level | insured_relationship | insured_zip | months_as_customer | capital-gains | capital-loss | occupation_id | hobby_id |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 48 | MALE | MD | husband | 466132 | 328 | 53300 | 0 | 3 | 18 |
| 2 | 42 | MALE | MD | other-relative | 468176 | 228 | 0 | 0 | 7 | 16 |
| 3 | 29 | FEMALE | PhD | own-child | 430632 | 134 | 35100 | 0 | 12 | 3 |

**VEHICLE** (model_id is a foreign key)

| vehicle_id | auto_year | model_id |
|---|---|---|
| 1 | 2004 | 29 |
| 2 | 2007 | 24 |
| 3 | 2007 | 14 |

**POLICY** (insured_id and vehicle_id are foreign keys)

| policy_number | policy_bind_date | policy_state | csl_per_person | csl_per_accident | policy_deductable | policy_annual_premium | umbrella_limit | insured_id | vehicle_id |
|---|---|---|---|---|---|---|---|---|---|
| 521585 | 2014-10-17 | OH | 250 | 500 | 1000 | 1406.91 | 0 | 1 | 1 |
| 342868 | 2006-06-27 | IN | 250 | 500 | 2000 | 1197.22 | 5000000 | 2 | 2 |
| 687698 | 2000-09-06 | OH | 100 | 300 | 2000 | 1413.14 | 5000000 | 3 | 3 |

**INCIDENT** (policy_number is a foreign key; unique on policy_number + incident_date)

| incident_id | policy_number | incident_date | incident_type | collision_type | incident_severity | authorities_contacted | incident_state | incident_city | incident_location | incident_hour_of_the_day | number_of_vehicles_involved | property_damage | bodily_injuries | witnesses | police_report_available |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 521585 | 2015-01-25 | Single Vehicle Collision | Side Collision | Major Damage | Police | SC | Columbus | 9935 4th Drive | 5 | 1 | YES | 1 | 2 | YES |
| 2 | 342868 | 2015-01-21 | Vehicle Theft | ? | Minor Damage | Police | VA | Riverwood | 6608 MLK Hwy | 8 | 1 | ? | 0 | 0 | ? |
| 3 | 687698 | 2015-02-22 | Multi-vehicle Collision | Rear Collision | Minor Damage | Police | NY | Columbus | 7121 Francis Lane | 7 | 3 | NO | 2 | 3 | NO |

**CLAIM** (incident_id is a foreign key)

| claim_id | incident_id | injury_claim | property_claim | vehicle_claim | fraud_reported |
|---|---|---|---|---|---|
| 1 | 1 | 6510 | 13020 | 52080 | Y |
| 2 | 2 | 780 | 780 | 3510 | Y |
| 3 | 3 | 7700 | 3850 | 23100 | N |

**CLAIM_PAYMENT** and **AUDIT_LOG** are not in the CSV. They are filled by the PL/SQL procedures and triggers of Phases V and VI.

## Step 6: Field mapping (Task 5)

| Raw field | Target table.column | Conversion or note (cleaning checklist for Phase IV) |
|---|---|---|
| months_as_customer | INSURED.months_as_customer | text to NUMBER |
| age | INSURED.age | text to NUMBER |
| policy_number | POLICY.policy_number | text to NUMBER (identifier, primary key) |
| policy_bind_date | POLICY.bind_date | text YYYY-MM-DD to DATE; 1 policy is bound after its incident |
| policy_state | POLICY.policy_state | none |
| policy_csl | POLICY.csl_per_person and POLICY.csl_per_accident | split text such as 250/500 into two NUMBER columns |
| policy_deductable | POLICY.deductible | rename (spelling); text to NUMBER |
| policy_annual_premium | POLICY.annual_premium | text to NUMBER with 2 decimals |
| umbrella_limit | POLICY.umbrella_limit | text to NUMBER; 1 negative value to review |
| insured_zip | INSURED.zip_code | keep as text (identifier, not a quantity) |
| insured_sex | INSURED.sex | none |
| insured_education_level | INSURED.education_level | none |
| insured_occupation | OCCUPATION.occupation_name (via INSURED.occupation_id) | lookup table |
| insured_hobbies | HOBBY.hobby_name (via INSURED.hobby_id) | lookup table |
| insured_relationship | INSURED.relationship | none |
| capital-gains | INSURED.capital_gains | rename (hyphen); text to NUMBER |
| capital-loss | INSURED.capital_loss | rename (hyphen); text to NUMBER (525 negative values) |
| incident_date | INCIDENT.incident_date | text YYYY-MM-DD to DATE |
| incident_type | INCIDENT.incident_type | none |
| collision_type | INCIDENT.collision_type | replace ? with NULL (178 rows) |
| incident_severity | INCIDENT.severity | rename |
| authorities_contacted | INCIDENT.authorities_contacted | keep the word None as a value (91 rows); do not load it as NULL |
| incident_state | INCIDENT.incident_state | none |
| incident_city | INCIDENT.incident_city | none |
| incident_location | INCIDENT.incident_location | none |
| incident_hour_of_the_day | INCIDENT.incident_hour | rename; text to NUMBER (0 to 23) |
| number_of_vehicles_involved | INCIDENT.vehicles_involved | rename; text to NUMBER |
| property_damage | INCIDENT.property_damage | replace ? with NULL (360 rows) |
| bodily_injuries | INCIDENT.bodily_injuries | text to NUMBER |
| witnesses | INCIDENT.witnesses | text to NUMBER |
| police_report_available | INCIDENT.police_report_available | replace ? with NULL (343 rows) |
| total_claim_amount | DROPPED | calculated: it equals injury + property + vehicle claim in all 1,000 rows |
| injury_claim | CLAIM.injury_claim | text to NUMBER |
| property_claim | CLAIM.property_claim | text to NUMBER |
| vehicle_claim | CLAIM.vehicle_claim | text to NUMBER |
| auto_make | AUTO_MAKE.make_name | correct spellings: Accura to Acura (68 rows), Suburu to Subaru (80 rows) |
| auto_model | AUTO_MODEL.model_name | none |
| auto_year | VEHICLE.manufacture_year | rename; text to NUMBER |
| fraud_reported | CLAIM.fraud_flag | rename; keep Y/N |
| _c39 | DROPPED | empty column in all 1,000 rows |

All 40 raw fields are accounted for: 38 are mapped to a table and 2 are dropped with a reason.

## Class demo: where record 521585 ends up

| Raw value | Ends up in |
|---|---|
| policy_number = 521585 | POLICY.policy_number |
| policy_bind_date = 2014-10-17 | POLICY.bind_date |
| policy_csl = 250/500 | POLICY.csl_per_person and csl_per_accident |
| age = 48 | INSURED.age |
| insured_occupation = craft-repair | OCCUPATION.occupation_name |
| insured_hobbies = sleeping | HOBBY.hobby_name |
| auto_make = Saab | AUTO_MAKE.make_name |
| auto_model = 92x | AUTO_MODEL.model_name |
| auto_year = 2004 | VEHICLE.manufacture_year |
| incident_date = 2015-01-25 | INCIDENT.incident_date |
| incident_type = Single Vehicle Collision | INCIDENT.incident_type |
| injury_claim = 6510 | CLAIM.injury_claim |
| property_claim = 13020 | CLAIM.property_claim |
| vehicle_claim = 52080 | CLAIM.vehicle_claim |
| fraud_reported = Y | CLAIM.fraud_flag |
| total_claim_amount = 71610 | not stored: calculated from the three claim amounts |
