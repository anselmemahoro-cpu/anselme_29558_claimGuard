# ClaimGuard: Motor Accident Claims and Fraud-Flag System

**Student:** NIYIMBONERA Mahoro Anselme
**Student ID:** 29558
**Course:** INSY 8311 – Database Development with PL/SQL
**Instructor:** Eric Maniraguha

## Problem Statement
Motor insurance companies often handle accident claims on paper or in spreadsheets. Because of this, claims are settled late, claims on expired policies slip through, and suspicious claims go unnoticed.

## Objectives
**Main objective:** Build an Oracle PL/SQL database that records and controls motor accident claims.

**Specific objectives:**
1. Store policyholders, vehicles, policies, claims and claim payments in related tables.
2. Block any claim filed on an expired policy.
3. Flag claims with unusually high amounts for review.
4. Let claims officers register claims and track their status through an Oracle APEX app.

## Data Source
- Dataset: insurance_claims.csv (1,000 anonymized motor insurance claims)
- Author: Aqqad, A. (2023), Mendeley Data
- Link: https://data.mendeley.com/datasets/992mh7dk9y/2
- Licence: CC BY 4.0

## Innovation
**Option:** User interface
**Tool:** Oracle APEX. A claims officer enters a claim in the app, and the app calls a PL/SQL procedure. The business rules stay in the database.

## Planned Tables
POLICYHOLDER, VEHICLE, POLICY, CLAIM, CLAIM_PAYMENT, AUDIT_LOG
(These may change after the normalization in Phase II.)

## Planned PL/SQL Features
- Procedure: register a new claim
- Function: check if a policy is active on the incident date
- Cursor: list all flagged claims
- Trigger: block a claim on an expired policy
- Trigger: flag claims above a set amount
- Package: group the claim procedures and functions together
- Exceptions: handle invalid policy numbers and missing data

## References
1. Aqqad, A. (2023). insurance_claims (Version 2) [Data set]. Mendeley Data. https://doi.org/10.17632/992mh7dk9y.2
2. Dondapati, H. (2025, November 29). Fraud detection in auto insurance claims using advanced machine learning models. The National High School Journal of Science. https://nhsjs.com/2025/fraud-detection-in-auto-insurance-claims-using-advanced-machine-learning-models/
3. Prabhakaran, S. (2025). The role of automation in insurance: Improving policy and claim management systems. Journal of Computer Science and Technology Studies, 7(6), 264-270. https://al-kindipublisher.com/index.php/jcsts/article/download/9962/8668/27561
4. Mwiza, S. (2026, March 13). Rwanda: National Insurance Strategy expected to expand coverage, build trust. The New Times. https://allafrica.com/stories/202603130547.html
5. Guidewire Software. (n.d.). ClaimCenter claims management software. https://guidewire.com/products/claimcenter-claims-management-software
6. Oracle. (n.d.). Oracle Database PL/SQL language reference, 19c: PL/SQL triggers. https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/plsql-triggers.html
