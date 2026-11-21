# Data Dictionary: ClaimGuard

This dictionary describes the 11 tables of the logical design. Oracle data types and sizes are left out on purpose; they are added in Phase IV.

**How to read it**
- **Key:** PK = primary key, FK = foreign key.
- **Required:** Yes means the column can never be empty.
- **Unique:** Yes means no two rows can have the same value.
- **Source:** the raw CSV field, or *Added* when the column is not in the CSV and is filled by the system.

## 1. OCCUPATION
Job types of insured people (lookup table).

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| occupation_id | PK | Identifier of the occupation | Yes | Yes | Added |
| occupation_name | | Name of the occupation, for example sales | Yes | Yes | insured_occupation |

## 2. HOBBY
Hobbies of insured people (lookup table).

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| hobby_id | PK | Identifier of the hobby | Yes | Yes | Added |
| hobby_name | | Name of the hobby, for example chess | Yes | Yes | insured_hobbies |

## 3. AUTO_MAKE
Car brands.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| make_id | PK | Identifier of the car brand | Yes | Yes | Added |
| make_name | | Brand name, for example Saab | Yes | Yes | auto_make |

## 4. AUTO_MODEL
Car models; each model belongs to one make.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| model_id | PK | Identifier of the car model | Yes | Yes | Added |
| model_name | | Model name, for example 92x | Yes | Unique together with make_id | auto_model |
| make_id | FK to AUTO_MAKE | The brand this model belongs to | Yes | No | auto_make |

## 5. VEHICLE
A car that is insured.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| vehicle_id | PK | Identifier of the vehicle | Yes | Yes | Added |
| manufacture_year | | Year the car was made | Yes | No | auto_year |
| model_id | FK to AUTO_MODEL | The model of the car | Yes | No | auto_model |

## 6. INSURED
The person who holds the insurance.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| insured_id | PK | Identifier of the insured person | Yes | Yes | Added |
| age | | Age in years | Yes | No | age |
| sex | | Sex of the insured person | Yes | No | insured_sex |
| education_level | | Highest education level | Yes | No | insured_education_level |
| relationship | | Family relationship status | Yes | No | insured_relationship |
| zip_code | | Zip code of the insured person (an identifier, not a quantity) | Yes | No | insured_zip |
| months_as_customer | | How many months the person has been a customer | Yes | No | months_as_customer |
| capital_gains | | Capital gains of the insured person | Yes | No | capital-gains |
| capital_loss | | Capital losses (stored as negative numbers) | Yes | No | capital-loss |
| occupation_id | FK to OCCUPATION | The job of the insured person | Yes | No | insured_occupation |
| hobby_id | FK to HOBBY | The hobby of the insured person | Yes | No | insured_hobbies |

## 7. POLICY
The insurance contract.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| policy_number | PK | Number of the policy | Yes | Yes | policy_number |
| bind_date | | Date the policy started | Yes | No | policy_bind_date |
| expiry_date | | Date the policy ends; empty when unknown (not in the CSV) | No | No | Added |
| policy_state | | State where the policy was issued | Yes | No | policy_state |
| csl_per_person | | Combined single limit: maximum paid per person | Yes | No | policy_csl (first number) |
| csl_per_accident | | Combined single limit: maximum paid per accident | Yes | No | policy_csl (second number) |
| deductible | | Amount the insured pays first | Yes | No | policy_deductable |
| annual_premium | | Premium paid each year | Yes | No | policy_annual_premium |
| umbrella_limit | | Extra liability cover | Yes | No | umbrella_limit |
| insured_id | FK to INSURED | The person who holds the policy | Yes | No | Added |
| vehicle_id | FK to VEHICLE | The car covered by the policy | Yes | No | Added |

## 8. INCIDENT
An accident that happened on a policy.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| incident_id | PK | Identifier of the incident | Yes | Yes | Added |
| policy_number | FK to POLICY | The policy the accident is reported on | Yes | Unique together with incident_date | policy_number |
| incident_date | | Date of the accident | Yes | See policy_number | incident_date |
| incident_type | | Kind of incident, for example Vehicle Theft | Yes | No | incident_type |
| collision_type | | Kind of collision; empty for theft or parked car | No | No | collision_type |
| severity | | How serious the damage is | Yes | No | incident_severity |
| authorities_contacted | | Which authority was contacted; the value None is allowed | Yes | No | authorities_contacted |
| incident_state | | State where it happened | Yes | No | incident_state |
| incident_city | | City where it happened | Yes | No | incident_city |
| incident_location | | Street location of the accident | Yes | No | incident_location |
| incident_hour | | Hour of the day, 0 to 23 | Yes | No | incident_hour_of_the_day |
| vehicles_involved | | Number of vehicles in the accident | Yes | No | number_of_vehicles_involved |
| property_damage | | Whether property was damaged; empty when unknown | No | No | property_damage |
| bodily_injuries | | Number of bodily injuries | Yes | No | bodily_injuries |
| witnesses | | Number of witnesses | Yes | No | witnesses |
| police_report_available | | Whether a police report exists; empty when unknown | No | No | police_report_available |

## 9. CLAIM
Money claimed after an incident.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| claim_id | PK | Identifier of the claim | Yes | Yes | Added |
| incident_id | FK to INCIDENT | The incident this claim belongs to | Yes | No | Added |
| injury_claim | | Amount claimed for injuries | Yes | No | injury_claim |
| property_claim | | Amount claimed for property damage | Yes | No | property_claim |
| vehicle_claim | | Amount claimed for vehicle damage | Yes | No | vehicle_claim |
| fraud_flag | | Y if the claim is marked as fraud, N if not | Yes | No | fraud_reported |
| claim_status | | Stage of the claim: Registered, Assessed, Approved, Rejected, Paid or Closed | Yes | No | Added |
| assessed_amount | | Amount set by the assessor | No | No | Added |
| decision_date | | Date the manager decided | No | No | Added |
| decision_reason | | Reason for approval or rejection | No | No | Added |

The total of a claim is not stored. It is calculated as injury_claim + property_claim + vehicle_claim.

## 10. CLAIM_PAYMENT
Payments made on an approved claim (filled by PL/SQL).

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| payment_id | PK | Identifier of the payment | Yes | Yes | Added |
| claim_id | FK to CLAIM | The claim that is paid | Yes | No | Added |
| payment_date | | Date of the payment | Yes | No | Added |
| amount | | Amount paid | Yes | No | Added |
| payment_method | | How it was paid, for example bank transfer | Yes | No | Added |

## 11. AUDIT_LOG
Record of every change (filled by a trigger). It has no foreign key on purpose, so the history is kept even if a row is later deleted.

| Column | Key | Meaning | Required | Unique | Source |
|---|---|---|---|---|---|
| audit_id | PK | Identifier of the log entry | Yes | Yes | Added |
| table_name | | Table that was changed | Yes | No | Added |
| record_id | | Identifier of the changed row | Yes | No | Added |
| action | | INSERT, UPDATE or DELETE | Yes | No | Added |
| old_value | | Value before the change | No | No | Added |
| new_value | | Value after the change | No | No | Added |
| changed_by | | User who made the change | Yes | No | Added |
| changed_at | | Date and time of the change | Yes | No | Added |

## Relationships (cardinality and optionality)

| Parent (one side) | Child (many side) | Meaning |
|---|---|---|
| OCCUPATION | INSURED | One occupation is held by zero or many insured people; each insured has exactly one occupation |
| HOBBY | INSURED | One hobby is shared by zero or many insured people; each insured has exactly one hobby |
| AUTO_MAKE | AUTO_MODEL | One make has zero or many models; each model has exactly one make |
| AUTO_MODEL | VEHICLE | One model is used by zero or many vehicles; each vehicle has exactly one model |
| INSURED | POLICY | One insured holds zero or many policies; each policy has exactly one insured |
| VEHICLE | POLICY | One vehicle is covered by zero or many policies over time; each policy covers exactly one vehicle |
| POLICY | INCIDENT | One policy has zero or many incidents; each incident belongs to exactly one policy |
| INCIDENT | CLAIM | One incident has zero or many claims; each claim belongs to exactly one incident |
| CLAIM | CLAIM_PAYMENT | One claim has zero or many payments; each payment belongs to exactly one claim |

AUDIT_LOG has no relationship line in the ERD.
