# Resume point – Phase 1 (Discovery and data contracts)

Last updated: 2026-11-04. Branch: `phase1-discovery` (not merged to `production`).

## Where we stopped
| Work item | Status | Output |
|---|---|---|
| W1 CAS access | Complete (keyring, D2 decided) | `scratch\phase1-discovery\W1-cas-access\W1_log.md` (scratch, not committed) |
| W2 CAS field inventory | Draft complete; value checks pending | `scratch\phase1-discovery\W2-cas-field-inventory\` (scratch, not committed) |
| W3 Tooling decision | Accepted (W3-1 to W3-7, PyMuPDF AGPL accepted) | PRD §6.1; `scratch\...\W3-tooling\` |
| W4a Fund workbook probe | Complete; decisions W4a-1 to W4a-9 accepted | `prompts\phase1-discovery\W4a-fund-workbook-probe\` |
| W4 AMC layout catalogue | Built for 15 workbooks; **needs extension** (see below) | `prompts\phase1-discovery\W4-amc-layout-catalogue\` |
| W5 Domain model | Draft; **depends on W4 extension**; decisions W5-1 to W5-6 awaiting review | `prompts\phase1-discovery\W5-domain-model\` |
| W6 Scheme master | Not started | – |
| W7 Look-through rules | Not started | – |
| W8 Data-quality rules | Not started (inputs: W4a anomalies A1–A15, S1, S2) | – |
| W9 Decisions log | Open (see PRD §6) | PRD §6, §6.1, §6.2 |

## Restart here (in order)
1. **Extend W4** (the catalogue must be revisited before W5 is signed off):
   - Record the name and industry source columns for each workbook.
   - Add a column map for each derivative and hedging table (for example Mirae PF header row 244; SBI Contra headers at rows 204, 222, 243, 254, 263). Use header labels within each table (rule in PRD §9).
   - Read the as-on date by cell type (text or date cell, W4a-5).
   - Close Excel first: the Mirae PF workbook has a lock file (`~$PF - Sep 2026.xlsx`) and is skipped until it is closed.
2. **Review and decide the open W5 items:**
   - Decisions W5-1 to W5-6 (PRD and `W5_log.md`).
   - Question 1: does each holding keep its printed name, or only link to the instrument? (Proposed: keep the printed name.)
   - Question 2: which demat columns add up to the total quantity? Check W2 section B against a real month.
3. **Sign off W4 and W5** (user review of `W4_catalogue.md` and `domain_model_DRAFT.md`).
4. **Start W6 (scheme master):** match CAS scheme codes and names (for example CF, MI006, RLMF046) to fund workbook schemes. Output: matching rules and an unmatched queue.
5. **Start W7 (look-through rules):** formula, scope (D1), nested funds (D9), date alignment, with a worked example from September 2026.
6. **W8 (data-quality rules)** and **W9 (decisions log)** to close Phase 1.

## Decisions still open (PRD §6)
- D1 look-through scope (equity only is recommended) – needed for W7.
- D3 reporting first (Python), D4 PostgreSQL via Docker, D5 Python 3.13, D6 `samples\` git-ignored (in effect), D7 initial commit (repo has commits now on `phase1-discovery`), D8 truncated PRD line (GoF assumed), D9 nested units (one level).
- D10 fund statements for January to August 2026 – needed for trend analysis; only September is available.

## Things to remember
- `scratch\` is git-ignored. Working logs for W1, W2, W3 and the probe scripts are not in git. If they need to be versioned, copy them into `prompts\phase1-discovery\` as was done for W4a, W4 and W5.
- Commit signing is turned off for this repo only (`commit.gpgsign=false` in the local config).
- The GitHub repo `pbdonda/myInvestmentsAnalysis` is public. The commit author email is `prasad.donda@outlook.com`. Decide whether to change it to the GitHub no-reply address before further pushes.
- Passwords are in keyring (`MyInvestmentsAnalysis` / `cas_pdf`). Never write them into files or logs.

## Git state
- Commits on `phase1-discovery`: `4291959` (planning files), `4f6871b` (README), `7cd78ce`, `f89303d`, `ea05bdc`, `06c416d`, `03634eb`, and the commit that adds this file.
- Not merged to `production`. `production` has no commits yet.
