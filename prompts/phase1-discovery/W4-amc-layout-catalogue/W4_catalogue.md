# W4 – AMC layout catalogue (15 fund workbooks, September 2026)

Built by `build_catalogue.py` from each workbook (1-based Excel rows and columns). Source JSON: `catalogue.json`.

| # | AMC | Workbook | Sheet | Header row | ISIN col | Name col | Quantity col | Value col | Weight col | Weight label | Value label | Value unit | Weight unit (grand total) | Grand total row | Holdings rows (ISIN) | ISIN countries | As-on location | Used / width cols | Error cells |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Axis | Monthly_Portfolio_Axis_Midcap_Fund_30_Septemb | AXISMCF | 4 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | Lakhs | fraction | 106 | 85 | IN | B3 text | I/25 | 0 |
| 2 | Axis | Monthly_Portfolio_Axis_Small_Cap_Fund_30_Sept | AXISSCF | 4 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | Lakhs | fraction | 149 | 128 | IN | B3 text | I/25 | 0 |
| 3 | Canara Robeco | EQ - Sep 2026.xlsx | EQ | 4 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lacs) | Lacs | percent | 112 | 98 | IN | B3 text | N/14 | 4 |
| 4 | Canara Robeco | IN - Sep 2026.xlsx | IN | 4 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lacs) | Lacs | percent | 65 | 51 | IN | B3 text | N/14 | 4 |
| 5 | Invesco | CF-Sep 2026.xlsx | CF | 5 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | Lakhs | percent | 99 | 71 | IN | B2 text | H/256 | 0 |
| 6 | Invesco | GF-Sep 2026.xlsx | GF | 5 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lakhs) | Lakhs | percent | 62 | 44 | IN | B2 text | H/8 | 0 |
| 7 | Mirae Asset | EB - Sep 2026.xlsx | MAEBF | 8 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lacs) | Lacs | fraction | 123 | 84 | IN | E5 date cell | I/255 | 0 |
| 8 | Mirae Asset | IO - Sep 2026.xlsx | MIIOF | 8 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lacs) | Lacs | fraction | 113 | 74 | IN | E5 date cell | I/255 | 0 |
| 9 | Mirae Asset | PF - Sep 2026.xlsx | MAIPF | 8 | C | B | E | F | G | % to Net Assets | Market/Fair Value (Rs. in Lacs) | Lacs | fraction | 197 | 151 | IN | E5 date cell | I/255 | 0 |
| 10 | Nippon India | GF - Sep 2026.xlsx | GF | 4 | B | C | E | F | G | % to NAV | Market/Fair Value ( Rs. in Lacs) | Lacs | fraction | 121 | 101 | IN | B2 text | H/8 | 0 |
| 11 | Nippon India | SC - Sep 2026.xlsx | SC | 4 | B | C | E | F | G | % to NAV | Market/Fair Value ( Rs. in Lacs) | Lacs | fraction | 275 | 256 | IN | B2 text | H/8 | 0 |
| 12 | Nippon India | TS - Sep 2026.xlsx | TS | 4 | B | C | E | F | G | % to NAV | Market/Fair Value ( Rs. in Lacs) | Lacs | fraction | 90 | 62 | IN | B2 text | H/8 | 0 |
| 13 | Sbi | SBI-CONTRA-FUND-MONTHLY-PORTFOLIO---SEPTEMBER | SCF | 6 | D | C | F | G | H | % to AUM | Market value (Rs. in Lakhs) | Lakhs | percent | 165 | 93 | IN, US | C4 text | L/256 | 0 |
| 14 | Sbi | SBI-HEALTHCARE-OPPORTUNITIES-FUND-MONTHLY-POR | SHOF | 6 | D | C | F | G | H | % to AUM | Market value (Rs. in Lakhs) | Lakhs | percent | 97 | 36 | IN | C4 text | K/256 | 0 |
| 15 | Sbi | SBI-SMALL-CAP-FUND-MONTHLY-PORTFOLIO---SEPTEM | SSCF | 6 | D | C | F | G | H | % to AUM | Market value (Rs. in Lakhs) | Lakhs | percent | 126 | 65 | IN | C4 text | K/256 | 0 |

## Quirks by AMC (per workbook; see W4a results for the anomaly register A1–A15)

- **Axis** (Monthly_Portfolio_Ax, Monthly_Portfolio_Ax): Nested MF units (INF) in holdings table after the equity section (Midcap 1 row; Small Cap none listed); derivative and option tables below holdings; plan NAV table on as-on comparison.
- **Canara Robeco** (EQ - Sep 2026.xlsx, IN - Sep 2026.xlsx): Four #VALUE! cells at K5:N5 (Excel numbering), in the Risk-O-Meter heading row; Risk-O-Meter columns to the right of the value/weight columns.
- **Invesco** (CF-Sep 2026.xlsx, GF-Sep 2026.xlsx): Sheet width 256 (CF) or 8 (GF) with trailing empty columns; derivative table with margin figures; sub-total rows for derivatives.
- **Mirae Asset** (EB - Sep 2026.xlsx, IO - Sep 2026.xlsx, PF - Sep 2026.xlsx): Header row 8; sheet width 255 with 8 used columns; as-on date is a date cell (E5); derivative tables with Quantity and Market value columns (not weights); nested MF units (INF) in section (a) Mutual Fund Units; two as-on dates in PF (current and August).
- **Nippon India** (GF - Sep 2026.xlsx, SC - Sep 2026.xlsx, TS - Sep 2026.xlsx): ISIN is column B (before name); weight label '% to NAV'; section 'Triparty Repo' used instead of TREPS; CCIL and margin rows labelled as instruments.
- **Sbi** (SBI-CONTRA-FUND-MONT, SBI-HEALTHCARE-OPPOR, SBI-SMALL-CAP-FUND-M): Header row 6; ISIN column D; weight label '% to AUM'; as-on is a date cell after 'PORTFOLIO STATEMENT AS ON :'; sheet width 256 with 10-11 used columns; 'Stock Options' rows with '#' footnote marker in the weight column (Contra row 99); foreign US ISINs in Contra (foreign securities section); derivative tables use Margin and Quantity columns; Contra printed % vs value mismatch (S1); Small Cap S2.

## Section markers found in the holdings table (text rows, first workbook per AMC)

- Axis (Monthly_Portfolio_Axis_M): r5: Equity & Equity related; r95: Others; r96: Mutual Fund Units; r100: Reverse Repo / TREPS
- Canara Robeco (EQ - Sep 2026.xlsx): r5: Equity & Equity related #VALUE! #VALUE! ; r107: TREPS
- Invesco (CF-Sep 2026.xlsx): r6: Equity & Equity related; r83: Derivatives; r93: Money Market Instruments; r94: TREPS / Reverse Repo
- Mirae Asset (EB - Sep 2026.xlsx): r9: EQUITY & EQUITY RELATED; r97: DEBT INSTRUMENTS; r106: MONEY MARKET INSTRUMENTS; r109: OTHERS; r119: Other Current Assets / (Liabilities); r149: Derivatives disclosure Table
- Nippon India (GF - Sep 2026.xlsx): r5: Equity & Equity related; r112: Money Market Instruments; r113: Triparty Repo/ Reverse Repo Instrument; r116: OTHERS; r149: Derivatives disclosure Table
- Sbi (SBI-CONTRA-FUND-MONTHLY-): r8: EQUITY & EQUITY RELATED; r10: Equity Shares; r98: Stock Options; r105: Real Estate Investment Trust; r109: Foreign Securities and /or overseas ETF; r116: DEBT INSTRUMENTS; r122: Securitised Debt Instruments NIL NIL; r130: MONEY MARKET INSTRUMENTS
