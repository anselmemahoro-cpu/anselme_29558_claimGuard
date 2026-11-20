# Dataset Card: insurance\_claims

## Description

|Item|Value|
|-|-|
|**Name**|insurance\_claims (file `insurance\_claims.csv`)|
|**Source**|Aqqad, A. (2023). *insurance\_claims* (Version 2) \[Data set]. Mendeley Data. https://doi.org/10.17632/992mh7dk9y.2|
|**Link**|https://data.mendeley.com/datasets/992mh7dk9y/2|
|**Licence**|CC BY 4.0 (educational use allowed, author must be credited)|
|**Format**|CSV, comma-separated text, one header row, line endings LF|
|**Size**|1,000 records, 40 columns (39 data columns and 1 empty trailing column), 266,964 bytes (about 261 KB)|
|**Date collected**|12 November 2026|
|**Location of raw file**|`data/raw/insurance\_claims.csv` (stored exactly as downloaded, never edited)|

**What one record represents:** one motor insurance policy (identified by a unique `policy\_number`) together with the insured person's details, one accident incident, the vehicle involved, the claim amounts and a fraud flag.

**Personal data:** the file has no names, phone numbers or ID numbers. Policy numbers are anonymised identifiers. The `insured\_zip` and `incident\_location` fields describe places only.

## Data quality results (counts from the real file; nothing has been fixed)

|#|Check|Result|
|-|-|-|
|1|Duplicate rows|**0** (also 0 duplicate `policy\_number` values and 0 duplicate `incident\_location` values)|
|2|Empty trailing column `\_c39`|**1** column, empty in **1,000 of 1,000** rows|
|3|Missing values written as `?`|`collision\_type` **178**, `property\_damage` **360**, `police\_report\_available` **343**. Total **881** cells; **660** rows have at least one `?`; **22** rows have all three|
|4|Meaning of `?` in `collision\_type`|All **178** rows are Vehicle Theft (94) or Parked Car (84) incidents, so there the `?` means "not applicable"|
|5|The word `None` in `authorities\_contacted`|**91** rows. This is a real category (no authority contacted) and must not be loaded as NULL|
|6|Dates stored as text (`YYYY-MM-DD`)|`policy\_bind\_date` and `incident\_date`: **2,000** values, **0** invalid. Ranges: bind date 1990-01-08 to 2015-02-22; incident date 2015-01-01 to 2015-03-01|
|7|Policy bound after the incident|**1** row (policy 794731: bound 2015-02-22, incident 2015-02-02)|
|8|Negative `umbrella\_limit`|**1** row (policy 526039: -1,000,000)|
|9|`capital-loss` stored as negative numbers|**525** rows have a negative value|
|10|`policy\_csl` holds two numbers in one text value (for example `250/500`)|**1,000** rows, **3** distinct values|
|11|Misspelled car brands|`Accura` (should be Acura) **68** rows; `Suburu` (should be Subaru) **80** rows|
|12|Column names not usable in Oracle or misspelled|`capital-gains` and `capital-loss` (hyphens), `policy\_deductable` (misspelled), `\_c39` (empty)|
|13|`insured\_zip` is an identifier but looks like a number|**5** zip codes are shared by **10** rows|
|14|Redundant column|`total\_claim\_amount` equals `injury\_claim + property\_claim + vehicle\_claim` in **1,000 of 1,000** rows|
|15|Numbers stored as text that fail to convert|**0** across the 17 numeric columns|
|16|Leading or trailing spaces|**0** cells|
|17|Different spellings of the same city or state|**0**|

**Other facts used in the design:** `fraud\_reported` is `Y` in 247 rows and `N` in 753 rows. Each of the 39 `auto\_model` values belongs to exactly one `auto\_make`. The 7 incident cities each appear with more than one incident state, so the city does not determine the state.

These issues will be fixed in Phase IV, not now.

