# Phase 1 – Discovery and data contracts
## Work item W5 – Canonical domain model

| Field | Value |
|---|---|
| Phase | 1 – Discovery and data contracts |
| Work item | W5 – Canonical domain model |
| Status | Draft complete for review. Depends on W4 (revisit may be needed) |
| Date | 2026-11-04 |
| PRD | `prompts\phase1-discovery\PRD_Phase1_Discovery.md` §5 (W5), §9, §6.2 |
| Size | M (PRD §12) |
| Inputs | W2 data dictionary (draft), W4 catalogue (draft), W4a rules and decisions |

### Objective
Define the entities, keys and relationships for the CAS and fund data, with an ERD sketch. Every W2 and W4 field maps to exactly one attribute, or is marked out of scope.

### Tasks executed
1. Listed entities from W2 (CAS sections) and W4 (fund workbooks), and the snapshot identities needed for month-on-month comparison.
2. Applied the accepted rules: raw as received, derived fields calculated, typed numerics, totals as checks, ISIN by format, idempotent loads by SHA-256, traceability, privacy.
3. Wrote the attribute list, the Mermaid ERD, the snapshot rules, and the out-of-scope list.
4. Mapped every W2 field (sections A–G) and every W4 column to an entity attribute (§7 of the draft).
5. Checked the look-through formula for units: CAS values in rupees, fund values in Rs. lakhs.

### Outputs
| Output | Location | Git |
|---|---|---|
| Domain model draft (entities, ERD, mapping) | `...\W5-domain-model\domain_model_DRAFT.md` | ignored (`scratch/`) |
| This log | `...\W5-domain-model\W5_log.md` | ignored |

### Results (summary)
- 21 entities (tables) and one derived view (`look_through_exposure`).
- Holdings from CAS and fund workbooks share `instrument` by ISIN. Company is linked through `instrument_company` (one company, several ISINs).
- Every W2 field maps to one attribute. Personal fields (name, PAN, address, e-mail, CAS ID) are out of scope; DP ID, client ID and folio number are stored only as salted hashes.
- Every W4 column maps to one attribute, or to `source_table_row` (raw), or is derived.
- Derived fields: `value_calc` (quantity × price), `unrealised_calc`, fund holding percentage (value ÷ grand total), instrument country from the ISIN.

### Acceptance check (PRD §5, W5)
| Criterion | Result |
|---|---|
| Domain model document with an ERD sketch | Met (Mermaid ERD in the draft) |
| Every W2 and W4 field maps to exactly one attribute, or is out of scope | Met for the listed fields. Two items need W2 or W4 follow-up (below) |

### Key decisions (proposed)
| ID | Decision | Basis |
|---|---|---|
| W5-1 | Raw values stored as printed, with derived fields calculated in views | User decision, 2026-11-04 |
| W5-2 | Personal identifiers are not stored; DP ID, client ID and folio number are stored only as salted hashes | PRD §9 privacy |
| W5-3 | Only holdings-table rows create `fund_holding`; hedging, option and swap rows are raw only | W4a-4 |
| W5-4 | Holding type comes from its section, not from its ISIN prefix | Nested fund units (INF) sit in the MF units section |
| W5-5 | Each snapshot is identified by its as-on date and source, and is never updated in place | Month-on-month comparison and traceability |
| W5-6 | Look-through share = my scheme value ÷ fund total, with lakh/rupee conversion | Units must agree (100,000 INR per lakh) |

### Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Salted hashes of DP, client and folio IDs cannot be matched across months if the salt changes | Low | High | Salt stored in the local config (secrets), never in the repo; rotation not planned |
| Scheme matching (W6) is unresolved, so `fund_scheme.mf_scheme_id` is nullable for a time | High | Medium | W6 next; unmatched queue |
| Name and industry columns differ across AMCs | Medium | Low | Keep the printed name per holding; instrument name is a convenience |
| The demat total quantity is unclear (see question 2) | Medium | Medium | Confirm in W2 section B before the parser |

### Dependencies
- W2 (CAS dictionary) and W4 (layout catalogue) must be reviewed by the user.
- W6 (scheme master) and W7 (look-through rules) depend on this model.

### Assumptions
- A single salt is used for hashes of personal references, kept outside the repo.
- Fund statements are month-end; CAS statement dates may differ (alignment in W7).
- The fund total (grand total) is the denominator for fund holding percentages.

### Issues to bring to your notice
1. **W4 revisit needed.** The mapping showed gaps in W4 (see §9 of the draft): name and industry source columns per workbook; the derivative and hedging column maps (A15), not yet in the catalogue; the as-on date read by type. The W4 catalogue should be extended before W5 is signed off.
2. **Demat total quantity (W2 open question).** Which columns add up to the total quantity is not confirmed. Stored only as raw columns for now.
3. **Scheme matching (W6).** Needed before the CAS and fund schemes can link.
4. **Error cells in W8.** The rule codes for error cells, section total mismatches (S1, S2), and unit mismatches are not yet defined.
5. **Scratch folder is git-ignored.** The W5 draft is not committed. If you want it versioned, it must move to `prompts\phase1-discovery\`.

### Next
- User review of the domain model, the ERD and decisions W5-1 to W5-6.
- Revisit W4 (extend the catalogue as in §9 of the draft), then W5 sign-off.
- W6 (scheme master) and W7 (look-through rules) follow.
