# Phase 1 – Discovery and data contracts
## Work item W4 – AMC layout catalogue

| Field | Value |
|---|---|
| Phase | 1 – Discovery and data contracts |
| Work item | W4 – AMC layout catalogue |
| Status | Complete for 15 workbooks (September 2026). Review pending |
| Date | 2026-11-04 |
| PRD | `prompts\phase1-discovery\PRD_Phase1_Discovery.md` §5 (W4), §9 (general rules), §6.2 (W4a decisions) |
| Size | L (PRD §12) |
| Inputs | W4a results, W3 tooling (openpyxl, pandas), accepted rules W4a-1 to W4a-9 |

### Objective
One row per fund workbook: sheet name, header row, column map, weight and value units, as-on location, section markers, footnote block and quirks.

### Tasks executed
1. Built the catalogue with `build_catalogue.py`, so every entry comes from the workbook itself (sheet, header row, column by label, grand total row, ISIN pattern, error cells, as-on cell).
2. Found and fixed two script issues before using the output: a false ISIN match inside a derivative code (`BF…`), and a file-path split that failed on Windows (fixed).
3. Checked the `#VALUE!` addresses against the workbook: K5:N5 in both Canara workbooks (Excel numbering). The earlier note said "row 4, cols 10–13", which used 0-based indexes. Corrected in W4a results and in the catalogue.
4. Wrote `W4_catalogue.md` (table, quirks by AMC, section markers) from `catalogue.json`.

### Outputs
| Output | Location | Git |
|---|---|---|
| Catalogue builder | `...\W4-amc-layout-catalogue\build_catalogue.py` | ignored (`scratch/`) |
| Catalogue (JSON, 15 entries) | `...\W4-amc-layout-catalogue\catalogue.json` | ignored |
| Catalogue (table and quirks) | `...\W4-amc-layout-catalogue\W4_catalogue.md` | ignored |
| Summary output | `...\W4-amc-layout-catalogue\catalogue_summary.txt` | ignored |
| This log | `...\W4-amc-layout-catalogue\W4_log.md` | ignored |

### Results (summary)
- 15 of 15 workbooks catalogued. One sheet each.
- Header rows: 4 (Axis, Canara, Nippon), 5 (Invesco), 6 (SBI), 8 (Mirae), 1-based.
- ISIN column: C in 12 workbooks; B in Nippon (ISIN before name); D in SBI.
- Weight unit from the grand total row: fraction in Axis (2), Mirae (3), Nippon (3); percent in Canara (2), Invesco (2), SBI (3).
- Weight label: "% to Net Assets" (Axis, Canara, Invesco, Mirae), "% to NAV" (Nippon), "% to AUM" (SBI).
- Value unit: Lakhs (Axis, Invesco, SBI); Lacs (Canara, Mirae, Nippon).
- ISIN countries: `IN` in all 15; `US` also in SBI Contra (foreign securities). `INF` fund units are within `IN`.
- As-on location: text cell B2 or B3 or C4 (most); date cell E5 (Mirae).
- Sheet width: 255–256 for Mirae, Invesco CF and SBI (trailing empty columns); 8–25 for the rest.
- Error cells: four `#VALUE!` in each Canara workbook, at K5:N5.
- Section marker rows: 2–7 per workbook (see W4_catalogue.md).

### Acceptance check (PRD §5, W4)
| Criterion | Result |
|---|---|
| All 15 workbooks catalogued | Met |
| Every quirk in PRD §4.2 recorded against its file | Met: per-AMC quirks table in W4_catalogue.md, with file names. The W4a anomaly register (A1–A15) holds the detail |

### Key decisions (proposed; within the accepted rules)
| ID | Decision | Basis |
|---|---|---|
| W4-1 | The catalogue is generated from the workbooks, not typed by hand | Repeatable; avoids the errors found in this session |
| W4-2 | Each catalogue row records the exact cell address behind every value (header, unit, as-on, error cells) | Traceability (PRD §9) |

### Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| A new AMC or a new month changes the layout | High | High | Header and unit detection by label and grand total (W4a-1, W4a-2); the catalogue is rebuilt for each new month |
| Catalogue covers September only | Certain | Medium | D10 (January–August statements) |
| The `#VALUE!` cells in Canara are in the heading row, so they are safe now but would break a parser that reads that row as data | Low | Medium | Error cells recorded; never read as values (W4a-3) |

### Dependencies
- W4a (complete). W3 (complete). W5 (domain model) depends on this catalogue.

### Assumptions
- The September 2026 workbooks represent the layouts for the other months. This is unverified until D10 data arrives.
- Section markers are taken from text rows that are not holdings. Some markers vary in spelling between AMCs; the catalogue keeps the spelling as printed.

### Issues to bring to your notice
1. **Mirae PF and Axis Midcap nested fund units.** These are INF-ISIN rows in the holdings table. D9 (one level) applies. The look-through scope decision is still open.
2. **Catalogue is built, not reviewed.** Please check `W4_catalogue.md` against the workbooks before W5 is signed off.
3. **Scratch folder is git-ignored.** The catalogue is not committed. If you want it versioned, it must move to `prompts\phase1-discovery\`.

### Next
- User review of the catalogue and decisions W4-1 and W4-2.
- W5 (canonical domain model), using the catalogue fields and the CAS dictionary from W2.
