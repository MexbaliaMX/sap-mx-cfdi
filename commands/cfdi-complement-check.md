---
description: Determine which CFDI complement(s) apply to a described transaction and surface the currently mandatory version of each
---

Given a description of a Mexican business transaction (a sale, a payment received, a payroll run, a goods transport, an export), determine:

1. **Which CFDI type** applies (`I` Ingreso, `E` Egreso, `T` Traslado, `N` Nómina, `P` Pago) — see `sap-mx-cfdi` skill, `references/cfdi-40-anexo20-core.md`.
2. **Which complement(s)** must be incorporated, cross-checking against `sap-mx-cfdi`'s "Current Mandatory Versions" table in `SKILL.md`:
   - Payment received after invoice issuance (PPD) → `complemento-pagos-20.md`
   - Payroll receipt → `complemento-nomina-12.md`
   - Goods physically moved via a vía general de comunicación (federal highway/rail/air/sea), and not within the C2-truck/30 km exemption → `complemento-carta-porte-31.md`
   - Definitive export with pedimento `A1` → `complemento-comercio-exterior.md`
   - Public-general/cash retail consolidation → `cfdi-global-publico-general.md`
3. **Flag version currency**: state plainly if the version this skill documents may be stale (check `last_verified` in `SKILL.md` frontmatter against today's date) and recommend re-verifying against the SAT source URLs before issuing a live document.
4. **Do not guess catalog values** (e.g., `c_ClaveProdServ`, `c_Pedimento`, `c_RegimenAduanero`) — point to `references/catalogs-and-validation.md` and the official SAT catalog files instead of inventing a code.
