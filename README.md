# MyInvestmentsAnalysis

A personal, local-only tool that consolidates my monthly investment statements into one view of how much I am exposed to each company, directly and through mutual funds, and how that exposure changes month to month.

## Why this project exists

Each month I receive two kinds of statement by email:

1. A **CDSL Consolidated Account Statement (CAS)** PDF. It lists my directly held shares and my mutual fund folios.
2. **Monthly portfolio disclosures** from mutual fund companies (AMCs), as Excel workbooks. Each one lists the stocks and other instruments a fund holds.

Neither statement answers the question I care about: *how much of my money is ultimately invested in company X?* A stock I hold directly and the same stock held inside three of my funds are spread across separate documents. This project brings them together.

## What it will do

- Store the holdings from both statement types in a **local PostgreSQL database**, so my financial data never leaves my machine.
- Calculate a **look-through exposure** per company: direct holdings plus each fund's holding in that company, scaled by how much of the fund I own.
- Track exposure **month on month**, so I can see trends and concentration.
- Produce **reports and charts** with Python (seaborn, scikit-learn) or Power BI.
- Later, fetch the statements **automatically from Gmail and Rediffmail**, so the monthly run needs no manual download.

## Privacy

- Everything runs locally. There is no cloud service and no third-party API that receives my holdings.
- Statement PDFs and workbooks are personal data. They stay in a git-ignored `samples\` folder and are never committed.
- Passwords for the CAS PDFs are kept in Windows Credential Manager through `keyring`. They are never written to files, logs or documentation.
- Documentation and logs contain no PAN numbers, account numbers, e-mail addresses or holding values.

## Current status

The project is in **Phase 1 – Discovery and data contracts**. No application code exists yet. Phase 1 is about understanding the real structure of the statements and agreeing the data model before any parser or database is built.

Completed so far:

- Access to all nine CAS statements for 2026 (January–September), confirmed with a keyring-stored password.
- Field inventory of the CAS statements, including the layout differences between months.
- PDF and Excel tooling decisions, based on trials.
- Structural and value checks on the 15 fund workbooks for September 2026.

## Roadmap

The phased plan lives in [`prompts/PLAN.md`](prompts/PLAN.md). The Phase 1 requirements are in [`prompts/phase1-discovery/PRD_Phase1_Discovery.md`](prompts/phase1-discovery/PRD_Phase1_Discovery.md).

| Phase | Focus |
|---|---|
| 0 | Project skeleton, quality gates, dependency management |
| 1 | Discovery and data contracts (current) |
| 2 | PostgreSQL schema and repositories |
| 3 | CAS ingestion |
| 4 | Fund portfolio ingestion (XLSX) |
| 5 | Look-through analytics |
| 6 | Reports and visualisation |
| 7 | Email automation (Gmail, Rediffmail) |
| 8 | Hardening, scheduling and operations |

## Technology

- Python 3.13
- PostgreSQL (local; Docker is the planned runtime)
- PDF: PyMuPDF (text), pdfplumber (tables)
- Excel: openpyxl, pandas
- Secrets: keyring (Windows Credential Manager)

## Repository layout

```
prompts/   Requirements, roadmap and per-phase PRDs
src/       Application code (empty until Phase 0)
test/      Tests (empty until Phase 0)
samples/   Statements used for discovery (git-ignored, personal data)
scratch/   Throwaway probes and their outputs (git-ignored)
```

## Licence and use

This is a private project for my own use. It uses PyMuPDF, which is licensed under AGPL; that is acceptable for private, local use only. The project is not intended for distribution.
