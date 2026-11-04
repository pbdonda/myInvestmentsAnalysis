# Phase 1 – Discovery and data contracts
## Work item W4a – Fund workbook probing

| Field | Value |
|---|---|
| Phase | 1 – Discovery and data contracts |
| Work item | W4a – Fund workbook probing |
| Status | Probes complete for all 15 workbooks; review items open (see section 5 of `W4a_results.md`) |
| Date | 2026-11-03 |
| PRD | `prompts\phase1-discovery\PRD_Phase1_Discovery.md` §5 (W4a), §4.2 (corrected), §6.1 |
| Size | M (PRD §12) |
| Sequence | After W3 (W3-7, accepted) |

### Objective
Probe all 15 September 2026 fund workbooks, read-only, for structure and values. Produce per-workbook results and a value-check table that feed W4 (catalogue) and W8 (data-quality rules).

### Tasks executed
1. Structure survey of every workbook: sheets, dimensions, header row, column labels, section markers, as-on lines (`probe_w4a_structure.py` → `structure_survey.txt`).
2. Value probe: header detection, column roles (ISIN, value, weight), error cells and their positions, ISIN format, numeric weights, weight unit inference, non-ISIN rows, section hints (`probe_w4a_values.py` → `value_checks.txt`).
3. Reconciliation: printed sub-total and total rows against holding-row sums; non-ISIN numeric rows; duplicate ISINs; as-on text (`probe_w4a_reconcile.py` → `reconcile_checks.txt`).
4. Anomaly checks: weight outliers above 100 in non-ISIN rows; SBI as-on cell layout (`probe_w4a_anomalies.py` → `anomaly_checks.txt`).
5. Check of nested mutual fund unit sections in Axis and Mirae (header rows and the rows beneath them).
6. Results table and anomaly register (`W4a_results.md`).

### Outputs
| Output | Location | Git |
|---|---|---|
| Probe scripts (4) | `...\W4a-fund-workbook-probe\probe_w4a_*.py` | ignored (`scratch/`) |
| Raw outputs (4) | `...\W4a-fund-workbook-probe\*_checks.txt`, `structure_survey.txt` | ignored |
| Results and anomaly register | `...\W4a-fund-workbook-probe\W4a_results.md` | ignored |
| This log | `...\W4a-fund-workbook-probe\W4a_log.md` | ignored |

### Results (summary)
- All 15 workbooks probed. One sheet each. Header rows at 0-based index 3, 4, 5 or 7.
- ISIN column: column 2 in 12 workbooks; column 1 in Nippon (ISIN before name); column 3 in SBI.
- Weight unit: eight workbooks use fractions (Axis ×2, Mirae ×3, Nippon ×3, with Mirae PF inferred). Seven use percentages (Canara ×2, Invesco ×2, SBI ×3, with SBI Contra and Small Cap inferred).
- Error cells: four `#VALUE!` cells in each of the two Canara Robeco workbooks, all in the section-heading row, Risk-O-Meter columns.
- ISIN validity: all holding ISINs have the correct 12-character format. No duplicate ISINs within a sheet.
- Derivative rows carry notional values in the weight column (for example SBI Contra up to 14,846).
- Two as-on dates in one Mirae PF sheet (current and August comparison).
- Nested mutual fund units: one row in Axis Midcap, two in Mirae EB, one in Mirae IO (INF ISINs).

### Acceptance check (PRD §5, W4a)
| Criterion | Result |
|---|---|
| All 15 workbooks probed | Met |
| Each check recorded for each workbook | Met for structure, error cells, ISIN, weight unit and totals. Value-level checks (quantity × price against value) not done (see Issues) |
| Each anomaly listed with its sheet and row reference, no personal values | Met: anomaly register A1–A13 in `W4a_results.md`, with sheet and row references in the raw outputs. Holding names are not recorded |

### Key decisions (proposed; for your review)
| ID | Proposed decision | Rationale |
|---|---|---|
| W4a-1 | Detect the header row, ISIN column, weight column and value column by label, not by position | Header rows, ISIN column and weight label all vary across AMCs (A9–A11) |
| W4a-2 | Read the weight unit per workbook from its grand total row (or inferred total), then normalise to percent | Eight fraction workbooks, seven percent workbooks; the PRD had this wrong |
| W4a-3 | Use openpyxl with error cells kept as errors | pandas hides them as NaN (W3 finding) |
| W4a-4 | Exclude derivative sections from weight sums; keep their rows raw | Notional values in the weight column (A2); D1 scope is equity only |
| W4a-5 | Use the primary as-on date, read by cell type | Two dates in one Mirae sheet (A3); SBI uses a date cell (A4) |
| W4a-6 | Flag INF-ISIN rows as nested fund units for D9 | Axis Midcap and Mirae hold fund units |
| W4a-7 | Do not record holding names or per-holding values in documents | Keeps the PRD and logs free of holdings detail; mirrors W2 rules |

### Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Wrong weight unit is applied to a workbook (fraction read as percent or the reverse) | Medium | High (exposure off by a factor of 100) | Confirm the unit from the grand total row for every workbook (done for 13; Mirae PF and SBI Contra, Small Cap inferred) |
| Derivative notional values are summed as weights | Medium | High | Exclude derivative sections (A2; W4a-4) and test with SBI Contra |
| Value column not reconciled to quantity × price | Medium | Medium | Add to Phase 4 golden tests; not done in W4a |
| Only September 2026 probed | Certain | Medium | D10: trends for January–August cannot be checked |
| Nested fund units double count look-through | Medium | Medium | D9 (one level); flag INF ISINs (W4a-6) |

### Dependencies
- W3 tooling decisions (accepted).
- W2 (CAS) for the scheme-mapping work in W6.
- D10 (January–August fund statements) for trend work.

### Assumptions
- The September 2026 workbooks are representative of the format for later months, to be tested when D10 data arrives.
- The printed "Total" and "Sub Total" rows are authoritative for section totals.
- Fund holdings and instrument names are AMC disclosures, not personal data. They are still kept out of the PRD to keep the documents small and reviewable.

### Issues to bring to your notice
1. **Mirae PF's grand total row was not found.** The fraction unit is inferred. Please confirm if you can see the workbook's total row; otherwise I will check the sheet manually in W4.
2. **SBI Contra and Small Cap units are inferred** from printed totals and the AUM grand total. Both are consistent with percentages, but not confirmed by a "Grand total" row in Contra.
3. **One weight value of 43,125 in Mirae PF** (an index or stock futures position, labelled "Long") needs confirmation that it is a derivative row, not a holding.
4. **PRD correction.** §4.2 said only Mirae uses fractions. Eight workbooks use fractions. Corrected in the PRD.
5. **Interpretation check.** The Nippon, Mirae and Axis fraction workbooks are all in the 0–1 range. The probe does not show whether any holding is recorded as a percentage with a value above 1. The maximum fraction is 0.0789 (Nippon TS), so this is unlikely, but it is not proven.
6. **Recursive folder search.** Discovery must search the six AMC subfolders. Recorded in the PRD (§4.2) and in W3's log.

### Next
- Your review of the anomaly register and the decisions W4a-1 to W4a-7.
- Then W4 (AMC layout catalogue): one row per workbook, built from `W4a_results.md`.
- W8 (data-quality rules) can start from the anomaly register A1–A13.
