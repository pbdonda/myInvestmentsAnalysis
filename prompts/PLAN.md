# Phase-wise Roadmap: MyInvestmentsAnalysis

Source: `prompts\PRD`

## Context
A local, private system that ingests monthly CDSL CAS PDFs (direct stocks plus mutual funds) and monthly mutual fund portfolio disclosures. It stores everything in local PostgreSQL and produces a consolidated, look-through view of exposure per company, with month-on-month trends. Later, the inputs are fetched automatically from Gmail and Rediffmail.

### Samples available (as of this revision)
- **CDSL CAS (PDF, password-protected):** 4 months: `202606`, `202607`, `202608`, `202609`.
- **Mutual fund statements (XLSX):** 15 workbooks from 6 AMCs (Invesco 2, SBI 3, Canara Robeco 2, Mirae Asset 3, Nippon India 3, Axis 2), all as on 30 Sep 2026.
- **Month overlap:** September 2026 has both a CAS and fund statements, so the first end-to-end match is possible. Jun-Aug have CAS only; fund statements for those months are needed for trend analysis across all months.

Note: the layouts below come from a survey of the first sheet of each workbook and the first rows of each file. Full parsing is Phase 3-4 work.

## 1. Task inventory
1. Discover the formats: CDSL CAS layout (demat holdings, MF folios, transactions) from PDFs, and per-AMC fund statement layouts from XLSX files.
2. Secure ingestion: PDF decryption with passwords from env/keyring, never stored in the repo.
3. Parsing: extract holdings and valuations per month from the CAS; extract fund portfolio holdings (company, ISIN, weight, quantity, value) from 6 AMC layouts.
4. Reference data: security master (ISIN, company, sector), instrument-to-company mapping (one company may have several ISINs), and a scheme master mapping fund scheme codes and names to the MF folios in the CAS.
5. Database: PostgreSQL schema, migrations, idempotent loads.
6. Look-through engine: direct holding plus (units held x fund NAV x % allocation) per company, per month.
7. Analytics: exposure ranking, concentration (HHI), fund overlap, sector exposure, MoM change and trend.
8. Reporting: seaborn/matplotlib/scikit-learn visuals, exported HTML/PDF; optional Power BI over Postgres views.
9. Operations: CLI, logging, audit trail, tests, CI, packaging, docs.
10. Email automation: fetch CAS and fund statements from Gmail (Gmail API/OAuth2) and Rediffmail (IMAP), filter by sender/subject, download attachments, deduplicate by hash, hand off to the same ingestion pipeline, then schedule unattended monthly runs.

## 2. Phases (in sequence)

### Phase 0 – Foundations

**Objectives**
- Create a clean, testable Python project skeleton.
- Set up quality gates so every later phase builds on the same standards.

**Tasks**
- Repo structure using a `src/` layout.
- Choose the Python version (Python 3.13 is installed locally).
- Dependency management with uv or poetry.
- Lint, format and type-check with ruff and mypy.
- pytest setup and pre-commit hooks.
- `.gitignore` excluding `samples\`, passwords, `.env` and data folders.
- Config handling with pydantic-settings; logging setup.

> 🟢 **Exit criteria:** empty skeleton passes CI.

### Phase 1 – Discovery and data contracts

**Objectives**
- Understand the real structure of the CAS and all six AMC statement layouts before writing parsers.
- Agree on the canonical data model and the scope of look-through, which all later phases depend on.

**Tasks**
- Analyse the four CDSL CAS PDFs and write the field inventory (holdings, MF folios, transactions, totals).
- Test parsing libraries on the CAS: pdfplumber for text/tables, pikepdf for decryption, camelot if tables are not extracted cleanly.
- Use openpyxl (or pandas on top of it) for the XLSX fund statements.
- Document each AMC layout: header row position, column order (Nippon puts ISIN before name), weight column name ("% to Net Assets", "% to AUM", "% to NAV"), weight units (Mirae stores fractions, others percentages), sheet name, and footnote blocks.
- **Decide look-through scope:** equity only, or all holdings (debt issuers, derivatives, REITs, foreign securities, unlisted, and mutual fund units held by Axis Midcap). Record the decision.
- Define the canonical domain model (instrument, company, holding, fund, scheme, fund portfolio).
- Confirm the value unit (Rs. lakhs in all sampled files) and store it explicitly.

> 🟢 **Exit criteria:** data dictionary, AMC layout catalogue, look-through scope decision, and agreed PRDs for Phases 2-6.

### Phase 2 – Data model and storage

**Objectives**
- Build a local PostgreSQL store that keeps every load traceable and repeatable.
- Make re-running an import safe (no duplicates).

**Tasks**
- Design the ERD: instrument, company, holding_snapshot, fund, scheme, fund_portfolio_snapshot, source_file (with SHA-256 and the "as on" date read from file content), load_run.
- Write Alembic migrations.
- Build the repository layer (Repository pattern).
- Implement idempotency using the file hash.

> 🟢 **Exit criteria:** schema deployed locally; repositories pass integration tests (testcontainers or local DB).

### Phase 3 – Ingestion pipeline (CAS)

**Objectives**
- Load direct stock and direct mutual fund holdings from all four CAS months (Jun-Sep 2026).
- Prove each load is correct by reconciling it against the PDF's own totals.

**Tasks**
- Decrypt password-protected PDFs.
- Extract and parse holdings and valuations for each month.
- Read the statement date from the document content, not the filename (`YYYYMM_CDSL_CAS*.pdf` is the convention, but content is the source of truth).
- Validate parsed data (Chain of Responsibility validators).
- Load into the database through the pipeline (Template Method).
- Select the per-format parser with a Factory (Strategy pattern).
- Reconcile totals against the PDF's stated portfolio value and write a report.

> 🟢 **Exit criteria:** all four months (202606-202609) load, with reconciliation reports.

### Phase 4 – Ingestion (fund portfolios, XLSX)

**Objectives**
- Load the stocks and other holdings each mutual fund scheme holds, from the 15 sample workbooks (Sep 2026) and later months.
- Map every fund holding to the security master so it can be combined with direct holdings.

**Tasks**
- One adapter per AMC (Strategy pattern) for the six layouts; detect the header row rather than hard-coding it.
- Read the statement date from the "as on" line in each workbook.
- Build the scheme master: map each file's scheme code and name (for example CF, GF, MI006, RLMF046) to the MF folio names in the CAS.
- Keep only the equity rows for the chosen look-through scope; keep other sections as raw data so the decision can change later.
- Normalise weights (Mirae fractions to percentages) and store the source unit.
- Handle Excel error values (for example `#VALUE!`), stray trailing columns (Mirae sheets are 255 columns wide), and footnote blocks.
- Normalise ISINs and map them to the security master; route unmatched instruments to a review queue.
- Golden-file test each AMC adapter using the sample workbooks.

> 🟢 **Exit criteria:** all 15 September 2026 workbooks load per scheme, with validation reports and an empty or reviewed unmatched queue.

### Phase 5 – Look-through analytics

**Objectives**
- Compute your true exposure to each company, direct and indirect through funds.
- Track how that exposure changes month on month.

**Tasks**
- Compute exposure as SQL views or materialized views, or as a pandas service.
- Calculate look-through exposure: units held x fund NAV x % allocation.
- Compute MoM deltas, concentration (HHI), fund overlap and sector roll-up.
- Handle nested fund holdings (Axis Midcap's mutual fund units) if the scope includes them.
- Test every calculation against hand-calculated fixtures.
- Use September 2026 (CAS plus fund statements) as the first end-to-end test month.

> 🟢 **Exit criteria:** September 2026 consolidated exposure table is correct against hand-calculated fixtures.

### Phase 6 – Reporting and visualization

**Objectives**
- Turn the analytics into clear monthly reports and trend views.
- Make reports reproducible with one command.

**Tasks**
- Seaborn charts: top exposures, trend lines, fund-overlap heatmap.
- Optional scikit-learn analysis: holding clustering, anomaly detection on exposure shifts.
- Monthly report generator (HTML/PDF).
- Power BI-ready view layer over Postgres.
- Trend views only become meaningful once fund statements exist for Jun-Aug 2026 (or the trend is shown from September onward).

> 🟢 **Exit criteria:** reproducible monthly report from a single command.

### Phase 7 – Email automation

**Objectives**
- Fetch the CAS and fund statements from Gmail and Rediffmail with no manual download.
- Keep the manual drop-in path working as a fallback.

**Tasks**
- `MailSource` abstraction (Strategy/Adapter pattern).
- `GmailSource`: Gmail API, OAuth2, read-only scope.
- `RediffSource`: IMAP with app password.
- Config-driven sender and subject rules; one rule set per AMC, since fund statements arrive from six different senders.
- Expect XLSX attachments from AMCs; confirm whether any are password-protected.
- Select attachments by rule and content (not by filename), because the sample filenames are inconsistent.
- Save attachments to an inbox folder and register them in `source_file` (hash dedup).
- Track processed messages; send failures to a quarantine folder with a notification.
- Phases 3-6 stay unchanged because the pipeline already takes a file path.

> 🟢 **Exit criteria:** a new email is fetched, ingested and reported with no manual step.

### Phase 8 – Hardening, scheduling and operations

**Objectives**
- Run the full monthly flow unattended and recover safely from failures.
- Make the system easy to operate, back up and maintain.

**Tasks**
- CLI with `typer`.
- Scheduler (Windows Task Scheduler or cron) for the end-to-end monthly run.
- Secrets management for mail credentials and PDF passwords.
- Backup and restore; audit log.
- Performance tuning: indexes, batch inserts.
- Documentation and release.

> 🟢 **Exit criteria:** monthly run completes unattended, and backup/restore is tested.

## 3. Cross-cutting engineering concerns
- **SOLID:** parsers, repositories, and analytics depend on abstractions (Protocols); dependency injection at a composition root.
- **Auditability and traceability:** every row links to `source_file` and `load_run`; raw parsed data kept; immutable snapshots.
- **Repeatability:** deterministic, idempotent reruns; pinned dependencies; seeded ML.
- **Performance and scalability:** batch loads, indexed (month, ISIN) keys, materialized views.
- **Security and privacy:** local only; passwords via keyring/env; no PII in logs; data directories git-ignored.
- **Testing:** unit tests on parsers with redacted fixtures, golden-file tests per AMC adapter, integration tests on the DB, golden-file tests on reports.

## 4. Risks and open questions
- PDF table parsing is brittle; CAS layout can change month to month.
- Six AMC layouts for fund statements, with differing columns, units and headers; each adapter needs its own tests, and new AMCs will need new adapters.
- Fund statements are as on month-end (30 Sep 2026), while CAS dates are statement dates; the alignment rule must be defined.
- Trend analysis needs fund statements for Jun-Aug 2026; only the CAS is available for those months so far.
- Look-through scope (equity only vs all holdings, nested mutual fund units, derivatives) is undecided.
- Confirm that `samples\` (which contains PII) stays out of git.
- Decide: Power BI vs Python-only reports; Python version.
- Email access: OAuth token storage and refresh for Gmail; whether Rediffmail still offers IMAP; keeping mailbox credentials local with minimal scopes; PDF password handling per sender; whether AMC attachments are password-protected.
- PRD line 34 is cut off ("Follow design patterns and al..."); GoF patterns assumed.
- An initial commit is needed on the GitHub repo before any GitHub integration (the app install failed on an empty repo).

## 5. Verification
- This file covers every PRD bullet: direct stock and fund storage, look-through exposure, monthly trend, local PostgreSQL, Python/Power BI reporting, email automation, and coding standards.
- Phases are ordered so each phase's exit criteria enable the next.
- Sample coverage is checked: CAS Jun-Sep 2026, fund statements Sep 2026.
- Next step: review and amend this file, then request detailed per-phase PRDs.
