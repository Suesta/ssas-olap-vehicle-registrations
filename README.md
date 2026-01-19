# ssas-olap-vehicle-registrations

End-to-end OLAP (SSAS) project built on a **star schema** centered on `FACT_vehicle_registration` and its related `DIM_*` tables. The repository documents the full workflow in **Visual Studio (SSDT / Analysis Services Multidimensional)**: data source configuration, Data Source View (DSV), cube creation, dimension hierarchies, processing/deployment, and exploratory OLAP analysis (MDX via the cube browser).

> Context: UOC — Analytical Databases (PR4).  
> Focus: reproducible, evidence-driven OLAP modeling and multidimensional analysis.

---

## Contents

- [Project overview](#project-overview)
- [Data model](#data-model)
- [OLAP artifacts](#olap-artifacts)
- [Repository structure](#repository-structure)
- [How to reproduce (lab environment)](#how-to-reproduce-lab-environment)
- [Cube exploration (MDX-style analysis)](#cube-exploration-mdx-style-analysis)
- [Evidence and report](#evidence-and-report)
- [Notes on limitations](#notes-on-limitations)
- [License](#license)

---

## Project overview

This project creates and validates an OLAP cube to analyze **vehicle registrations** from a data warehouse. The work follows a classic analytical pipeline:

1. Ensure the relational dataset exists and is loaded in SQL Server.
2. Configure an Analysis Services project and its deployment target.
3. Build a **Data Source View (DSV)** that represents the star schema.
4. Create the **cube** with a single measure (`Quantity`) and the required dimensions.
5. Configure **hierarchies, attributes, and attribute relationships** for performant slicing/drilling.
6. Process and deploy the cube.
7. Run exploratory analysis in the cube browser (and optionally in Excel as a client).

---

## Data model

The cube is based on a star schema with:

**Fact table**
- `FACT_vehicle_registration`  
  - Measure: `Quantity` (count of registrations)

**Dimensions**
- `DIM_Date` (time)
- `DIM_dgt_type` (vehicle classification from DGT)
- `DIM_vehicle` (vehicle descriptors: brand, model, fuel/propulsion, eco label, seats, etc.)
- `DIM_municipality` (geography: province, municipality, population)

The DSV links the fact table to each dimension through foreign keys, enabling clean multidimensional navigation.

---

## OLAP artifacts

### Cube: `Cube_vehicle_registrations`
- **Measure group**: registrations
- **Measure**: `Quantity`
- **Dimensions**:
  - Date
  - DGT Type
  - Vehicle
  - Municipality

### Dimension configuration highlights

**DIM_Date**
- Hierarchy: `Fecha` (Year → Month → Day)
- Attribute relationships: Day → Month → Year (and Date at the leaf level)
- Ordering: `OrderBy = Key` to keep chronological order
- Visibility: base attributes hidden when only the hierarchy is required

**DIM_dgt_type**
- Hierarchy: `Tipo` → `Subtipo`
- Attribute relationships aligned to granularity
- `KeyColumns`, `NameColumn`, and ordering configured to avoid processing errors

**DIM_vehicle**
- Attributes (renamed for end-user friendliness):  
  `Marca`, `Modelo`, `Categoría Eléctrica`, `Alimentación`, `Propulsión`, `Etiqueta ECO`, `Plazas`
- Hierarchy: `Marca` → `Modelo`
- `Id Vehicle` kept as a technical key (not exposed as a user hierarchy)

**DIM_municipality**
- Attributes (renamed): `Provincia`, `Municipio`, `Habitantes`
- Hierarchy: `Provincia` → `Municipio`
- Technical identifiers not exposed as end-user hierarchies

---

## Repository structure

Current contents:

- `README.md` — this documentation.
- `LICENSE` — MIT license.
- `BDA_PR4_Suesta_Arribas_Victor.pdf` — final deliverable/report (includes the step-by-step process and evidences).
- `M2.888_PR4_Enunciado.pdf` — official statement/specification for the assignment.

> Note: The Visual Studio project files (`.sln`, `.dwproj`, `.dim`, `.dsv`, `.cube`, etc.) can be added later if you want the repository to be fully reproducible outside the report-driven approach. In the academic context, the PDF deliverable is the primary audited artifact.

---

## How to reproduce (lab environment)

This summarizes the operational steps used in the lab environment (SQL Server + SSAS).

### 1) Load relational data in SQL Server
- Execute: `PR4_DDL+Data_SQLS.sql` (provided with the assignment) to create and populate tables.

### 2) Create SSAS Multidimensional project (Visual Studio / SSDT)
- Create an **Analysis Services Multidimensional and Data Mining** project (or import from server if required by the lab).
- Configure deployment properties:
  - Target SSAS server provided by the lab
  - Database name: `DEST_<loginuoc>` (as specified)

### 3) Configure Data Source
- Edit the `.ds` connection:
  - SQL Server: lab server
  - Authentication: `STUDENT_<loginuoc>` + password (lab credentials)
- `Impersonation Information`: **Inherit**
- Test connection (must succeed)

### 4) Create the Data Source View (DSV)
- Include all star schema tables:
  - `DIM_Date`, `DIM_dgt_type`, `DIM_municipality`, `DIM_vehicle`, `FACT_vehicle_registration`
- Name: `View_Vehicle_registrations`
- Verify relationships: fact table connected to each dimension

### 5) Create the cube
- Method: **Use existing tables**
- Measure group: `FACT_vehicle_registration`
- Measures: select only `Quantity`
- Dimensions: all `DIM_*` tables
- Name: `Cube_vehicle_registrations`

### 6) Configure dimensions + hierarchies
- Implement the required hierarchies and attribute properties described in the statement.
- Process each dimension after configuration changes.

### 7) Process & deploy cube
- Process the cube and confirm:
  - No processing errors
  - Deployment completes successfully

---

## Cube exploration (MDX-style analysis)

The cube was validated through interactive browsing (equivalent to MDX query execution):

Typical analyses include:
- Total registrations by year
- Registrations by DGT type (e.g., filtering `AUTOBUSES`)
- ECO label analyses by brand/model with date constraints
- Municipality-level breakdowns for specific vehicle categories
- Seat-count constrained analyses over a given year/month split

The report shows the exact selections, filters, and results with screenshots so the analysis is auditable.

---

## Evidence and report

The full set of evidences (screenshots) and the narrative explanation are included in:

- `BDA_PR4_Suesta_Arribas_Victor.pdf`

This document covers:
- Project setup and deployment configuration
- Data Source + DSV creation
- Cube creation and processing
- Dimension modeling (attributes, hierarchies, relationships)
- OLAP exploration results (filters, drilldowns, outputs)

---

## Notes on limitations

- The repository currently prioritizes **auditable documentation** (PDF deliverable) over uploading the full SSDT project directory.
- If the SSAS project files are later added, it is recommended to:
  - Include a `src/ssas_project/` folder with the Visual Studio solution
  - Add an `evidence/` folder with the screenshots referenced in the report
  - Include a short `repro.md` with environment prerequisites and deployment notes

---

## License

This project is released under the **MIT License** (see `LICENSE`).
