# New CFDI Complement Onboarding Checklist

Use this when a project needs to support a CFDI complement not yet implemented (e.g., adding Carta Porte to an entity that previously only issued plain CFDI Ingreso/Egreso).

## 1. Scope confirmation
- [ ] Confirm the actual trigger condition applies (e.g., for Carta Porte: route exceeds the 30 km/C2 exemption; for Comercio Exterior: the entity genuinely exports under pedimento A1) — don't build for a complement "just in case."
- [ ] Identify every legal entity in scope and confirm each has its own correct master data for the complement's entity-specific fields (RFC, Registro Patronal, address) — never assume group-wide values.

## 2. Version/catalog currency
- [ ] Check `SKILL.md`'s "Current Mandatory Versions" table `last_verified` date against today.
- [ ] Re-fetch the live SAT guide/catalog from the URL in `sources:` if `last_verified` is more than ~90 days old.
- [ ] Confirm the PAC/integration partner's catalog files match the same SAT publication date — a stale catalog on the PAC side rejects identically to a stale catalog on the ERP side.

## 3. SAP-side framework identification
- [ ] Confirm this complement is an **eDocument Cockpit** concern (almost certainly yes — see `sap-architecture-edocument-vs-drc.md`), not a DRC concern.
- [ ] Identify the relevant BAdI(s) and `/EDOMX` value-mapping tables; do not assume a brand-new custom development is required before checking SAP's standard BAdI catalog.

## 4. Field mapping
- [ ] Walk every required/conditional attribute in the relevant `references/*.md` file and map it to a source field in the ERP (master data or transaction).
- [ ] Flag every field whose source is "not currently captured anywhere" — these are real gaps, not implementation details to defer.

## 5. Testing
- [ ] Test against the PAC's sandbox/test-stamping environment before production cutover.
- [ ] Test at least one rejection scenario deliberately (e.g., a deliberately stale catalog code) to confirm the error-handling/resend flow in the eDocument Cockpit actually surfaces it to a human.

## 6. Cross-reference this project's other agents
- [ ] If this is the Vanilla Agents project: confirm the relevant L2C/P2P/R2R/H2R/INT knowledge files reference this skill's file by name (`filename.md §N`) rather than re-describing the SAT standard inline.
