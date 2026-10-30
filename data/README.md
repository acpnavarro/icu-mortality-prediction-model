# Data (not distributed)

This folder is intentionally **empty**. No patient data is shared with this repository.

## What goes here

Place the source spreadsheet (`Excel01GlobalAdmission.xlsx`) in this folder and point `FILE_PATH` in
notebook `01_cohort_construction.ipynb` to it. The file must come from an **authorized copy** of the
hospital database, used under the same ethical approvals as the original study.

## Expected input schema

The source is a single Excel workbook with seven sheets (`Folha2`–`Folha8`), each with a section-title
row followed by the real header (read with `header=1`):

| Sheet | Content | Key columns |
|---|---|---|
| `Folha2` | Admission index: demographics, admission context, severity scores, discharge fields | `EpimedCode`, `MedicalRecord` |
| `Folha3` | Comorbidity flags (~53 boolean indicators) | `EpimedCode`, `MedicalRecord` |
| `Folha4` | Acute admission reasons | `EpimedCode`, `MedicalRecord` |
| `Folha5` | First-hour / first-day complications | `EpimedCode`, `MedicalRecord` |
| `Folha6` | 6-hour physiology and laboratory | `EpimedCode`, `MedicalRecord` |
| `Folha7` | Resource use over the entire stay (**excluded** — leakage) | `EpimedCode`, `MedicalRecord` |
| `Folha8` | Secondary diagnoses (12 slots, with "present at ICU admission" flags) | `EpimedCode`, `MedicalRecord` |

The sheets link to each other **only** through the pair (`EpimedCode`, `MedicalRecord`) — there is no
admission-level identifier. The pipeline handles duplicated keys via positional alignment with key
verification (see Phase 1, Section 6).

Outcome fields: `HospitalDischargeName` (Alta/Óbito — discharge/death) and `HospitalDestinationName`
(used to detect transfers to other hospitals and define the adjusted outcome).

## Why no data is shared

The database contains identifiable patient information from electronic health records. Sharing it — even
partially or in derived form — is restricted by privacy regulations and the study's ethical approval.
This repository publishes the **methods and code** only.
