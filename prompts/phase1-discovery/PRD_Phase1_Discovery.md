# PRD: Phase 1 – Discovery and Data Contracts

Parent roadmap: `prompts\PLAN.md` (Phase 1)
Source requirements: `prompts\PRD`
Status: Draft for review

## 1. Overview and goal
Before any parser, database or report is built, the project must know the real structure of its inputs and agree on the data model they feed. Phase 1 produces that knowledge as documents and decisions. It writes no ingestion, database or reporting code.

## 2. Objectives
- Inventory every field in the CDSL CAS statements for the full year (demat holdings, MF folios, transactions, totals).
- Catalogue the layout of every AMC fund statement (6 AMCs, 15 sample workbooks).
- Choose the PDF and Excel tooling based on evidence from trials.
- Fix the canonical domain model, the scheme mapping approach and the look-through rules.

## 3. Scope
**In scope**
- Probing the sample files (read-only) with throwaway scripts.
- Writing the data dictionary, AMC layout catalogue, tooling decision record, domain model, look-through rules, data-quality rules and decisions log.

**Out of scope**
- PostgreSQL schema, migrations or database access.
- Production parsers, ingestion pipeline or email automation.
- Reports, charts or Power BI.

**Scratch work:** probe scripts live in `scratch\`, which is git-ignored. Probe output is redacted before it is written into any document.

## 4. Inputs and known findings

### 4.1 CDSL CAS (`samples\cdsl_ecas\`)
Nine monthly statements covering January to September 2026. All nine are password-protected. The earlier unprotected copies (produced with "Microsoft: Print To PDF") have been removed from the samples, so they are not an input.

| File | Password | Opens with keyring password | Pages | Text characters | Embedded images |
|---|---|---|---|---|---|
| 202601_CDSL_CAS.pdf | Yes | Yes | 34 | 88,468 | 38 |
| 202602_CDSL_CAS.pdf | Yes | Yes | 38 | 95,584 | 42 |
| 202603_CDSL_CAS.pdf | Yes | Yes | 52 | 133,297 | 56 |
| 202604_CDSL_CAS.pdf | Yes | Yes | 39 | 100,630 | 43 |
| 202605_CDSL_CAS.pdf | Yes | Yes | 40 | 101,430 | 44 |
| 202606_CDSL_CAS.pdf | Yes | Yes | 41 | 104,186 | 45 |
| 202607_CDSL_CAS.pdf | Yes | Yes | 40 | 102,711 | 44 |
| 202608_CDSL_CAS.pdf | Yes | Yes | 40 | 100,237 | 44 |
| 202609_CDSL_CAS.pdf | Yes | Yes | 39 | 107,534 | 43 |

Result of the keyring probe (2026-11-03): one password opens all nine months, and every month has a text layer (no OCR needed). Counts come from `scratch\cas_probe_keyring.py`; no statement content is recorded here.

W2 drift check (2026-11-03, draft): the demat holdings columns are identical in all nine months. Differences are confined to March and September (ARN code column in the MF folio table, a commission and TER section, and a ₹ rather than backtick rupee sign), plus the page positions of some tables. August has no demat transactions ("No Transaction during the period"), which is a genuine empty month. Full register: `scratch\phase1-discovery\W2-cas-field-inventory\data_dictionary_cas_DRAFT.md`.

### 4.2 Fund statements (`samples\mf_statements\`), all as on 30 Sep 2026
| AMC | Workbooks | Schemes (codes seen) |
|---|---|---|
| Invesco | 2 | CF, GF |
| SBI | 3 | SBI Contra, SBI Healthcare Opportunities, SBI Small Cap |
| Canara Robeco | 2 | EQ, IN |
| Mirae Asset | 3 | PF (MI006), EB, IO |
| Nippon India | 3 | TS (RLMF046), SC, GF |
| Axis | 2 | Midcap, Small Cap |

Only September 2026 is available for fund statements. Trend analysis for January to August needs matching fund statements for those months.

Observed variations:
- Workbooks sit in six AMC subfolders under `samples\mf_statements\` (axis, canara robeco, invesco, mirae asset, nippon india, sbi), not in one flat folder. Discovery must search recursively.
- Header row differs (rows 4–8); title blocks differ.
- Column order differs (Nippon has ISIN before the name; others have name first).
- Weight column is named "% to Net Assets", "% to AUM", or "% to NAV".
- Weight units (corrected by W4a, 2026-11-03; confirmed by each workbook's GRAND TOTAL row):
  - **Fractions** (grand total 1.0): Axis Midcap, Axis Small Cap, Mirae EB, IO, PF (PF inferred; its grand total row was not located), Nippon GF, SC, TS.
  - **Percentages** (grand total 100): Canara Robeco EQ, IN; Invesco CF, GF; SBI Contra, Healthcare Opportunities, Small Cap.
  - The earlier statement that only Mirae uses fractions was wrong.
- Value unit is Rs. lakhs (named "Lakhs" or "Lacs"; Canara and Nippon use "Lacs").
- Sheets with trailing empty columns: Mirae (255 wide, 8 used), Invesco CF (256 wide, 8 used), SBI (256 wide, 10–11 used).
- Derivative rows (index futures, stock futures, option contracts) carry notional values in the weight column, not percentages. Examples: SBI Contra up to 14,846 and SBI Small Cap 18,477. Look-through scope (D1) excludes them, but they must not be summed as weights.
- Canara Robeco EQ and IN contain four `#VALUE!` cells each, all in the section-heading row (row 4 of the sheet, columns 10–13: Risk-O-Meter columns). They are outside the holding columns.
- SBI workbooks store the as-on date in a true date cell (after the label "PORTFOLIO STATEMENT AS ON"). Mirae PF also shows the August comparison date in the same sheet; the primary as-on date must be used.
- Nested mutual fund units appear as holding rows with INF ISINs: Axis Midcap (one money-market fund), Mirae EB and IO (money-market and liquid funds).
- Each holding section is followed by printed "Sub Total" and "Total" rows. These must not be double counted.
- Sections other than equity appear: debt, money market, TREPS/repo, derivatives (futures and options), REITs, foreign securities, unlisted, and mutual fund units (Axis Midcap).
- Every holding row seen carries a 12-character ISIN.

## 5. Work items

### W1 – CAS access
- **Description:** define how the password-protected CAS files are opened. The password is stored in Windows Credential Manager through `keyring` (service `MyInvestmentsAnalysis`, username `cas_pdf`). It is read only inside the function that opens a file, and is never printed, logged, written to the repo or recorded in any document.
- **Recommendation:** keyring (Windows Credential Manager). Confirm that the same password works for all nine months, or record a per-month password source. For probe work, the user supplies the password for the session; it is used, then discarded, and not written to any file, log or memory.
- **Deliverable:** decision recorded in the decisions log (D2), and the keyring service and username agreed.
- **Acceptance:** all nine months open, or each failure is recorded with its cause and the chosen path.

### W2 – CAS field inventory
- **Description:** list every section and field per statement: demat holdings (ISIN, company, quantity, price, value), MF folios (scheme, folio, units, NAV, value), transactions, portfolio totals, and statement period. Record the data type and unit for each field. Check each month for layout drift.
- **Deliverable:** data dictionary (CAS section), with a note of any month whose layout differs.
- **Acceptance:** all nine months (202601–202609) are covered, or each gap is explicitly listed.

### W3 – PDF and Excel tooling decision
- **Description:** compare candidates on real text-based pages once the CAS files open. Candidates: pdfplumber, PyMuPDF, pypdf, PyPDF2 (installed), camelot, pikepdf. For Excel: openpyxl and pandas.
- **Result (2026-11-03, proposed for review):** PyMuPDF opens and extracts text (all nine CAS files; fastest). pdfplumber extracts tables (8–12 s per file). Camelot, pypdf and PyPDF2 are not used. pikepdf is not needed because PyMuPDF decrypts the files. openpyxl reads and checks the workbooks, because pandas hides error cells as NaN. Full evidence: `scratch\phase1-discovery\W3-tooling\tooling_decision_record.md`.
- **Caveat:** table accuracy is measured by coverage only. Accuracy against printed totals is a Phase 3 reconciliation task.
- **Deliverable:** tooling decision record with trial results (table extraction accuracy, speed, dependency footprint).
- **Acceptance:** one library per job is chosen, with the evidence written down.

### W4a – Fund workbook probing
- **Status:** proposed; the probe runs only after the user approves this item.
- **Description:** run read-only probes across all 15 fund workbooks (`samples\mf_statements\`), using the same masking rules as W2. Output must not contain holding names, values, scheme-specific personal data, or e-mail addresses. Probe two areas:
  - *Structure:* sheet names, header row index, column labels, trailing empty columns (Mirae sheets are about 255 columns wide), section markers, footnote block, and the as-on date location.
  - *Values:* weight column total per section (near 100% in its unit), weight unit (fraction or percentage), error cells (`#VALUE!`) with their location, value unit label, and ISIN presence and format on holding rows.
- **Deliverable:** per-workbook probe results and a value-check table, in `scratch\phase1-discovery\W4a-fund-workbook-probe\`, with a W4a log in the same format as W1 and W2. The results feed W4 (catalogue) and W8 (data-quality rules).
- **Acceptance:** all 15 workbooks probed; each check recorded for each workbook; each anomaly listed with its sheet and row reference, with no personal values.
- **Limit:** only September 2026 statements exist (D10). The probe confirms layouts for that month only.

### W4 – AMC layout catalogue
- **Description:** one row per workbook: sheet name, header row index, column map (name, ISIN, industry, quantity, value, weight), weight unit, value unit, location of the as-on date, section markers, footnote block, and quirks.
- **Deliverable:** AMC layout catalogue (table, one row per workbook).
- **Acceptance:** all 15 workbooks catalogued; every quirk in §4.2 is recorded against its file.

### W5 – Canonical domain model
- **Description:** define entities and keys: instrument (ISIN as the join key), company (one company may have several ISINs), holding (direct), scheme, fund_portfolio_snapshot (fund holdings), source_file (SHA-256, as-on date from content), and load_run. Define relationships and how month-on-month snapshots are identified.
- **Deliverable:** domain model document with an ERD sketch (text or Mermaid).
- **Acceptance:** every field in W2 and W4 maps to exactly one entity attribute, or is marked as out of scope.

### W6 – Scheme master mapping
- **Description:** define how a fund scheme in an AMC workbook (code such as CF, MI006, RLMF046, or a scheme name) is matched to the MF folio scheme name in the CAS. Define matching rules (exact code, normalised name, ISIN of the scheme if present) and the unmatched queue.
- **Deliverable:** scheme master specification with matching rules.
- **Acceptance:** each of the 15 workbooks has a proposed scheme key, and the mapping method is written down.

### W7 – Look-through rules
- **Description:** define the formula (units held × NAV × weight), the look-through scope (see decision D1), the treatment of nested mutual fund units (Axis Midcap), and the date alignment rule between month-end fund statements and the CAS statement date.
- **Deliverable:** look-through rules document.
- **Acceptance:** the formula, scope and alignment rule are each stated with a worked example using September 2026 data.

### W8 – Data-quality rules catalogue
- **Description:** list the checks that later phases must apply: `#VALUE!` and other error cells, footnote and total rows that must be excluded, duplicate ISINs within a scheme, missing ISIN, weight totals not summing near 100%, unit mismatches, and empty trailing columns.
- **Deliverable:** data-quality rules table (rule, detection, action, severity).
- **Acceptance:** every quirk in §4.2 has a rule.

### W9 – Decisions log
- **Description:** record each open decision with options, recommendation, status and owner. See §6.
- **Deliverable:** decisions log section in this document, updated as decisions close.
- **Acceptance:** every decision has a status of Decided, Deferred, or Open with a date.

## 6. Decisions

| ID | Decision | Options | Recommendation | Status |
|---|---|---|---|---|
| D1 | Look-through scope | Equity only / equity + derivatives / all holdings | Equity only; store other sections raw | Open |
| D2 | CAS password access | Keyring (Windows Credential Manager) / env-var password / per-month password source | Keyring entry `MyInvestmentsAnalysis` / `cas_pdf`; one password if it opens all nine months | Decided 2026-11-03 (keyring); single password confirmed for all nine months |
| D3 | Reporting first | Python / Power BI / both | Python first; Power BI views added later | Open |
| D4 | PostgreSQL runtime | Docker / native install | Docker with a persistent volume | Open (Phase 2) |
| D5 | Python version | 3.13 (installed) / other | 3.13 | Open |
| D6 | Samples in git | Git-ignore / commit | Git-ignore `samples\` (contains personal data) | Open |
| D7 | Initial commit | Required before GitHub integration | Make an initial commit on the `production` branch after Phase 0 | Open |
| D8 | Truncated PRD line 34 ("Follow design patterns and al...") | Ask user / assume | Assume standard GoF patterns; confirm with user | Open |
| D9 | Nested mutual fund units (Axis Midcap) | Ignore / one level / recursive | One level in the first release; record recursion as a later item | Open |
| D10 | Fund statements for January–August 2026 | Provide them / trend from September only | Provide them, so trends span the full year | Open |

### 6.1 Tooling decisions (W3, accepted 2026-11-03)
Evidence: `scratch\phase1-discovery\W3-tooling\tooling_decision_record.md`. The user accepted all seven decisions and noted that the PyMuPDF AGPL licence is acceptable for private use.

| ID | Decision | Status |
|---|---|---|
| W3-1 | PyMuPDF opens and extracts text from CAS PDFs | Decided |
| W3-2 | pdfplumber extracts tables and rows | Decided |
| W3-3 | Camelot not used | Decided |
| W3-4 | pypdf and PyPDF2 not used | Decided |
| W3-5 | openpyxl reads and checks workbooks; pandas only after validation | Decided |
| W3-6 | pikepdf not needed; PyMuPDF decrypts | Decided |
| W3-7 | Sequence W3 → W4a → W4 → W5 | Decided |
| L1 | PyMuPDF AGPL licence (private use only) | Accepted by user |

## 7. Deliverables of Phase 1
1. Data dictionary (CAS sections and fields, nine months) – W2
2. AMC layout catalogue (15 workbooks) – W4
3. Tooling decision record – W3
4. Canonical domain model and ERD sketch – W5
5. Scheme master specification – W6
6. Look-through rules document – W7
7. Data-quality rules table – W8
8. Decisions log – W9
9. Outline of the Phase 2–6 PRDs (one paragraph per phase, identifying the inputs each phase needs from Phase 1)

## 8. Acceptance / exit criteria
> 🟢 **Exit criteria:**
> - CAS field inventory covers all nine months (202601–202609), or the gaps are listed with their blocker.
> - All 15 fund workbooks are catalogued in the AMC layout catalogue.
> - PDF and Excel tooling choices are backed by a recorded trial.
> - Decisions D1 (scope), D2 (CAS password access) and D10 (fund statements for January–August) are closed.
> - The canonical domain model and look-through rules are reviewed and signed off by the user.

## 9. Engineering principles for this phase
- **SOLID-driven model design:** entities are defined by responsibility; parsers and the scheme mapper are specified as interchangeable strategies.
- **Traceability:** every catalogue row and rule cites the sample file it came from.
- **Repeatability:** probe scripts are kept in `scratch\` and can be rerun.
- **Security and privacy:** no passwords, PAN numbers, email addresses, or account numbers are written into any document. Samples are redacted before quoting. Passwords live only in keyring; probe work receives the password from the user for the session and does not persist it.

## 10. Risks specific to Phase 1
| Risk | Mitigation |
|---|---|
| CAS files are password-protected; no layout can be checked until the password is available | Password in Windows Credential Manager via keyring (D2); read only at use, never logged |
| One or more CAS months may be image-only (no text layer) | Checked: all nine months have a text layer, so OCR is not needed for now. Re-check any future month that fails to produce text |
| Layout changes between months | Catalogue per AMC with header detection; check all nine months for drift in W2 |
| Only September fund statements exist, so trends for January–August cannot be checked against fund data | Decision D10: obtain the matching fund statements |
| Scheme names do not match between AMC files and the CAS | Scheme master with code and normalised-name rules; unmatched queue |
| Fund statements (30 Sep) and CAS dates differ | Alignment rule defined in W7 |
| Personal data in samples and documents | Redaction before quoting; `samples\` git-ignored (D6) |

## 11. Dependencies and sequencing
- Depends on: sample files (present), the CAS password (D2, stored via keyring), Phase 0 for `scratch\` and `.gitignore` (a root `.gitignore` already exists and covers `scratch\`, `samples\` and `secrets\`).
- Sequence: W1 → W2 → W3 → W4a → W4 → W5 (W4a uses the libraries chosen in W3; W5 needs the W2 and W4 outputs).
- Blocks: Phases 2–6. Phase 2 cannot start until W5 is signed off.

## 12. Effort estimate (relative)
| Work item | Size |
|---|---|
| W1 CAS access | S (depends on the password being provided) |
| W2 CAS field inventory | L (nine months, drift check per month) |
| W3 Tooling decision | M |
| W4a Fund workbook probing | M (15 workbooks, read-only checks) |
| W4 AMC layout catalogue | L (15 workbooks, 6 layouts) |
| W5 Domain model | M |
| W6 Scheme master | M |
| W7 Look-through rules | S |
| W8 Data-quality rules | S |
| W9 Decisions log | S |

## 13. Verification
- This document exists in `prompts\phase1-discovery\` and follows the same format as `PLAN.md` (objectives, task bullets, 🟢 exit criteria).
- Every Phase 1 task and risk in `PLAN.md` appears in a work item, a decision or a risk.
- The file contains no passwords, PAN numbers, email addresses or account numbers.
- The user reviews and amends the decisions in §6 before Phase 2 begins.
