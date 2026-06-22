# SAP Architecture: eDocument Cockpit vs. Document and Reporting Compliance (DRC)

**Sources:** SAP Help Portal (Document and Reporting Compliance overview, "Supported Compliance Tasks by Country/Region"); SAP Community posts on Mexico eDocument/Carta Porte configuration; SAP Knowledge Base Article 2523263 (eDocument Mexico Troubleshooting Guide); cross-checked against `ASUG MX DRC.pdf` (16-slide ASUG México deck, scoped entirely to "Contabilidad Electrónica e Informes de IVA").

## The Split, Stated Plainly

SAP Mexico compliance is **not one framework** — it is two architecturally distinct frameworks that happen to both be marketed under "Document and Reporting Compliance" in current SAP terminology, but that are configured, extended, and troubleshot completely differently:

| | **eDocument Cockpit** | **SAP DRC (Mapping Manager / Schema Manager / Document Generator / BRFPlus)** |
|---|---|---|
| **Owns** | CFDI invoice issuance, Complemento de Pago, Complemento Nómina, **Complemento Carta Porte**, Complemento Comercio Exterior — every *transactional, PAC-stamped* CFDI | Electronic Accounting (Contabilidad Electrónica: Catálogo/Balanza/Pólizas/Auxiliares) and VAT reporting (DIOT, Declaración de IVA) — *periodic statutory reports*, not transaction-level documents |
| **Mechanism** | BAdIs (e.g., "Mexico: Filling of Transportation Complement V4.0") + `/EDOMX` namespace value-mapping tables | Mapping Manager → Schema Manager → Document Generator pipeline, driven by BRFPlus rules reading ABAP CDS reporting views |
| **App / Cockpit** | eDocument Cockpit (monitor/resend/cancel individual CFDI documents) | DRC's own compliance-reporting apps (per SAP Note 2812643/2986139; SAP Notes applied in ascending sequence) |
| **Counterparty** | PAC (Proveedor Autorizado de Certificación) — real-time/near-real-time REST stamping | SAT Portal directly — scheduled monthly/on-request batch XML submission |
| **Failure mode** | A single document is rejected by the PAC; resend/correct that one document | An entire periodic report is rejected by SAT; the whole submission for that period must be corrected and resubmitted |

## Why This Matters

It is easy — and this project's own INT/R2R knowledge files have historically blurred this — to describe "SAP DRC" as the single thing that "handles CFDI for Mexico." It does not. If you configure, troubleshoot, or scope an implementation as though Carta Porte and Electronic Accounting share a pipeline, you will:
- Look in the wrong place for a Carta Porte rejection (DRC's Mapping Manager has nothing to do with it — check the eDocument Cockpit and the `/EDOMX` value mappings instead).
- Underestimate the implementation effort, because the two frameworks have separate configuration, separate SAP Notes, and separate go-live testing cycles.
- Miscommunicate scope to a client who read a vendor deck (like the ASUG MX DRC presentation) that — correctly, for its own stated scope — never once mentions CFDI, Carta Porte, or PAC stamping, because that deck is *only* about Electronic Accounting + VAT reporting.

## How to Apply This When Reading the Rest of This Skill

Every reference file in this skill (`cfdi-40-anexo20-core.md`, `complemento-pagos-20.md`, `complemento-nomina-12.md`, `complemento-carta-porte-31.md`, `complemento-comercio-exterior.md`, `cfdi-global-publico-general.md`) describes a document type that lives in the **eDocument Cockpit** world. None of them are configured through DRC. If a knowledge file or agent in this project says "DRC handles CFDI/Carta Porte stamping," that statement is imprecise and should be corrected to reference the eDocument Cockpit instead — DRC's actual Mexico scope is Electronic Accounting and DIOT/VAT only.

## Open Item for This Project

The Vanilla `INT` agent's `int_isa_methodology.md` (SAT Electronic Accounting interface entry) and `R2R`'s `r2r_statutory_compliance_mexico.md` both reference "SAP DRC" — verify each reference is scoped to Electronic Accounting/VAT (correct) and not generalized to cover CFDI stamping broadly (would need correction). This is tracked as part of the CFDI agent rollout's cross-reference weaving step, not a standalone fix.
