# W4a – Fund workbook probe results (September 2026, 15 workbooks)

Date: 2026-11-03
Source: `samples\mf_statements\<amc>\` (six AMC subfolders, recursive search)
Probes: `probe_w4a_structure.py`, `probe_w4a_values.py`, `probe_w4a_reconcile.py`, `probe_w4a_anomalies.py`
Outputs: `structure_survey.txt`, `value_checks.txt`, `reconcile_checks.txt`, `anomaly_checks.txt`
Privacy: holding names are not recorded here. Instrument and issuer names appear only where they explain an anomaly.

## 1. Structure

| # | AMC | Workbook (sheet) | Header row (0-based) | ISIN column | Weight label | Value label | Width / used cols | Holding rows (ISIN) | Error cells |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Axis | Midcap (AXISMCF) | 3 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | 25 / 9 | 85 | 0 |
| 2 | Axis | Small Cap (AXISSCF) | 3 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | 25 / 9 | 128 | 0 |
| 3 | Canara Robeco | EQ (EQ) | 3 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lacs) | 14 / 13 | 98 | 4 (K5:N5, Excel numbering) |
| 4 | Canara Robeco | IN (IN) | 3 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lacs) | 14 / 13 | 51 | 4 (K5:N5, Excel numbering) |
| 5 | Invesco | CF (CF) | 4 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | 256 / 8 | 71 | 0 |
| 6 | Invesco | GF (GF) | 4 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | 8 / 8 | 44 | 0 |
| 7 | Mirae Asset | EB (MAEBF) | 7 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lacs) | 255 / 8 | 84 | 0 |
| 8 | Mirae Asset | IO (MIIOF) | 7 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lacs) | 255 / 8 | 74 | 0 |
| 9 | Mirae Asset | PF (MAIPF) | 7 | 2 | % to Net Assets | Market/Fair Value (Rs. in Lacs) | 255 / 8 | 151 | 0 |
| 10 | Nippon India | GF (GF) | 3 | 1 (before name) | % to NAV | Market/Fair Value ( Rs. in Lacs) | 8 / 8 | 101 | 0 |
| 11 | Nippon India | SC (SC) | 3 | 1 (before name) | % to NAV | Market/Fair Value ( Rs. in Lacs) | 8 / 8 | 256 | 0 |
| 12 | Nippon India | TS (TS) | 3 | 1 (before name) | % to NAV | Market/Fair Value ( Rs. in Lacs) | 8 / 8 | 62 | 0 |
| 13 | SBI | Contra (SCF) | 5 | 3 | % to AUM | Market value (Rs. in Lakhs) | 256 / 11 | 91 | 0 |
| 14 | SBI | Healthcare Opportunities (SHOF) | 5 | 3 | % to AUM | Market value (Rs. in Lakhs) | 256 / 10 | 36 | 0 |
| 15 | SBI | Small Cap (SSCF) | 5 | 3 | % to AUM | Market value (Rs. in Lakhs) | 256 / 10 | 65 | 0 |

Sheet count: one sheet per workbook in all 15 files.
ISIN format: every holding ISIN is a valid 12-character ISIN (`IN` + 10 characters). Zero format failures. Zero duplicate ISINs within a sheet.

## 2. Weight unit (confirmed against each workbook's own total rows)

| Unit | Workbooks | Evidence |
|---|---|---|
| Fraction (grand total = 1.0) | Axis Midcap, Axis Small Cap, Mirae EB, Mirae IO, Nippon GF, SC, TS | GRAND TOTAL row = 1.0 |
| Fraction (confirmed 2026-11-04 by user and workbook) | Mirae PF | Section totals in rows 97 (equity 0.7614), 174 (debt 0.1983), 185 (money market 0.0121), 191 (TREPS 0.0200), 195 (receivables 0.0082); GRAND TOTAL row 197 = 1. Sections sum to 1.0; value totals sum to 923,895.92 lakhs |
| Percentage (grand total = 100) | Canara EQ, Canara IN, Invesco CF, Invesco GF, SBI Healthcare Opportunities | GRAND TOTAL row = 100 (Canara and Invesco GF), or AUM total = 100 |
| Percentage (confirmed 2026-11-04 for Contra) | SBI Contra | Section totals sum to 100.00 (row 165 AUM = 100); values sum exactly to AUM (4,506,750.91 lakhs); section %s match value ÷ AUM except equity (+0.04) and net receivables (-0.04), see discrepancy S1 |
| Percentage (confirmed 2026-11-04) | SBI Small Cap | AUM grand total row 126 = 100; section values sum exactly to AUM (4,105,916.82 lakhs); all 65 holding rows match value ÷ AUM within 0.006 points. Discrepancy S2 (equity and net receivables) |

Correction to PRD §4.2: the earlier note "only Mirae stores fractions" was wrong. Eight workbooks use fractions.

## 3. Totals and reconciliation

For every workbook the printed section totals were compared with the sum of holding weights.
- The printed "Sub Total" and "Total" rows for each section reconcile to the grand total (100 or 1.0).
- The holding-row sum alone does not reach the total, because the remainder sits in TREPS, cash margin, net receivables, nested fund units without ISIN and derivatives.
- Canara EQ and IN: grand total 100 is reached with TREPS (4.43 and 2.88) and net receivables.
- Invesco CF: sub-total rows for derivatives (1.91, 2.23) are printed separately from equity (96.56).
- Duplicate totals exist for the same section (printed "Sub Total" followed by "Total"). They must not be double counted.

Caveat: this reconciliation is by printed totals only. The value column (Rs. lakhs) was not reconciled against quantity × price. That check belongs in Phase 4 and Phase 5 tests.

## 4. Anomalies and their handling (inputs to W8)

| ID | Anomaly | Where | Impact | Proposed handling |
|---|---|---|---|---|
| A1 | `#VALUE!` in the section-heading row, columns 10–13 (Risk-O-Meter) | Canara EQ, IN | None on holdings; pandas hides them as NaN | Record; openpyxl reads them as errors; never treated as values |
| A2 (corrected 2026-11-04) | Large numbers in the weight column of the lower hedging tables, which use a different meaning ("Margin maintained in Rs. Lakhs"). Not in the holdings table. SBI Contra rows 223–224 (14,846 and 12,936); SBI Small Cap similar (18,477). Mirae PF row 245 (Excel numbering) is not a weight: 43,125 is the Quantity (contracts) of a futures position in the "Other than hedging positions through futures" table (value 359.17 lakhs, 0.0389% of net assets). The probe read the wrong column (see A15) | Would distort any weight sum if the table is read as holdings | Read holdings only from the holdings table (section and ISIN rows); keep hedging tables as raw, with their own column meaning |
| A15 | Probe error, not a data issue: my anomaly probe used the holdings table's column index for every table. Hedging tables have different column layouts, so their values were read from the wrong column (Quantity or Margin) | Mirae PF row 245, SBI Contra rows 223–224, SBI Small Cap row 181 | Produced false "weights" of 43,125, 14,846 and 18,477 | Parsers must locate columns by header label within each table, not by one global index (W4a-1). Probe scripts to be corrected |
| A14 | `#` symbol in the weight column of a holding row (a footnote marker, not a value) | SBI Contra row 99 (a call option row) | Would break numeric parsing | User decision 2026-11-04: the column is a percentage column; `#` is stored as NULL in a numeric field. Raw text kept |
| S2 | SBI Small Cap, same pattern as S1: equity total row 74 printed 85.42, value-derived 85.40 (+0.017); net receivables row 123 printed 0.09, value-derived 0.106 (-0.016); offsetting, about 680 lakhs. Total row 124 printed 0.72, value-derived 0.74 (follows from row 123) | Printed % and value disagree in two sections | Use value-derived %; keep printed % as raw (user decision 2026-11-04) |
| S1 | Printed % to AUM differs from value ÷ AUM | SBI Contra: equity row 96 printed 88.57, value-derived 88.53 (+0.04); net receivables row 162 printed -0.57, value-derived -0.53 (-0.04). Each about 1,700 lakhs; they offset, so the printed total still reads 100 | Printed % and value disagree in two sections | User decision 2026-11-04: use value-derived % as the calculated figure; keep printed % as raw for reconciliation |
| A3 | Two as-on dates in one sheet (current and previous month) | Mirae PF (August comparison row), Axis (August and September plan NAVs) | Wrong date could be taken | Use the primary "as on" statement date only |
| A4 | As-on date is a true date cell | SBI workbooks (after "PORTFOLIO STATEMENT AS ON :") | Text search misses it | Read the date cell by type, not by text |
| A5 | Nested mutual fund units appear as holding rows with INF ISINs | Axis Midcap (one money-market fund), Mirae EB (two), Mirae IO (one) | Look-through would double count if not handled (D9) | One level in the first release (D9); flag INF ISINs |
| A6 | Printed "Sub Total" and "Total" rows for each section | All 15 | Double counting if summed | Use section-level totals only for checks, never as holdings |
| A7 | Non-holding rows with numeric weights (TREPS, triparty repo, cash margin, net receivables, "Net Current Assets", "GRAND TOTAL") | All 15 | They are not instruments | Classify as non-instrument rows; keep raw; exclude from look-through |
| A8 | Trailing empty columns (width 255–256 with 8–11 used) | Mirae (3), Invesco CF, SBI (3) | Wide scan is slow and misleading if read as data | Detect the last used column; ignore empty columns |
| A9 | Header row varies (rows 3, 4, 5, 7 in 0-based) | All AMCs | Hard-coded row fails | Detect the header by the cell containing "ISIN" (as the probes do) |
| A10 | ISIN column varies (column 2 or 1; Nippon has ISIN first) | Nippon | Hard-coded column fails | Detect the ISIN column by its header label |
| A11 | Weight label varies ("% to Net Assets", "% to NAV", "% to AUM") | All AMCs | Hard-coded label fails | Match with a regex on "%" |
| A12 | Value label varies ("Lakhs" or "Lacs") | All AMCs | Unit string differs | Normalise to Rs. lakhs |
| A13 | Rows without ISIN but with numeric quantity or value | All 15 (11–54 rows each) | Could be missed holdings | Classified in A7; SBI Contra and Healthcare have the most (54 and 12) and need review in W4 |

## 5. Items needing a decision or review
1. **Mirae PF grand total not located.** The fraction unit is inferred from the holdings sum and the maximum weight. Confirm in W4.
2. **SBI unit inferred, not confirmed by a grand total.** Contra's "Total" rows sum to 100; Small Cap's AUM grand total is 100. Confirm in W4.
3. **Value checks not done.** Quantity × price against the value column is not checked here.
4. **SBI Contra has 54 non-ISIN rows.** Most are derivative or foreign rows. W4 must catalogue them before any look-through rule is applied.
5. **Derivative rows (A2) are excluded from weight sums, but their existence needs a rule** in W7 and W8 (D1 says equity only).
