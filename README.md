# sap-mx-cfdi

A portable Claude Code skill/plugin: a comprehensive reference for Mexico's **CFDI 4.0** electronic invoicing standard (Anexo 20) and its complements, plus how **SAP S/4HANA** implements them.

It is **generic and industry-agnostic** — it documents the SAT standard and SAP's implementation of it, not any client's business processes. It's designed to be lifted into its own repo or marketplace entry without rework, which is why it lives here on its own.

## What it covers

| Document / Complement | Version | Mandatory Since |
|---|---|---|
| CFDI (Comprobante) | 4.0 | Apr 1, 2023 (available/optional from Jan 1, 2022; mandatory after two SAT extensions) |
| CFDI que ampara Retenciones e Información de Pagos | 2.0 | Apr 1, 2023 (same CFDI 4.0 cutover) |
| Complemento de Pago ("Pagos") | 2.0 | Apr 1, 2023 |
| Complemento Nómina | 1.2, revisión E | Jan 1, 2026 |
| Complemento Carta Porte | 3.1 | Jul 17, 2024 |
| Complemento Comercio Exterior | 2.0 (superseded 1.1, no transition period) | Jan 18, 2024 (1.1 was mandatory Apr 1, 2023 – Jan 17, 2024) |
| CFDI Global (Público en General) | 4.0 | Apr 1, 2023 (same CFDI 4.0 schedule) |

It also disambiguates SAP's **eDocument Cockpit** (transactional CFDI/complement stamping via PAC) from **Document and Reporting Compliance / DRC** (periodic statutory reports: Electronic Accounting, DIOT/VAT) — a distinction that's easy to blur and causes real architecture mistakes when it is.

See the version table and source links in [`skills/sap-mx-cfdi/SKILL.md`](skills/sap-mx-cfdi/SKILL.md) — re-verify against the official SAT URLs there before relying on this for a live engagement, since SAT updates complement catalogs every few months.

## When to use it

- Filling out or validating a CFDI 4.0 document or one of its complements.
- Determining which complement applies to a transaction (sale, payment, payroll, transport, export).
- Checking the current mandatory version/revision before a go-live or year-end cutover.
- Troubleshooting a PAC rejection or field-level validation error.
- Deciding whether a Mexico compliance task belongs to SAP's eDocument Cockpit or DRC.

## Structure

```
sap-mx-cfdi/
├── .claude-plugin/
│   └── plugin.json                              # Plugin manifest
├── commands/
│   └── cfdi-complement-check.md                 # /cfdi-complement-check — determine applicable CFDI type/complement for a described transaction
└── skills/sap-mx-cfdi/
    ├── SKILL.md                                  # Entry point: scope, version table, file index
    ├── references/
    │   ├── cfdi-40-anexo20-core.md               # Base Comprobante schema (Anexo 20 §I)
    │   ├── cfdi-retenciones-pagos-info.md         # CFDI que ampara retenciones e información de pagos (Anexo 20 §II)
    │   ├── complemento-pagos-20.md                # Complemento de Pago 2.0
    │   ├── complemento-nomina-12.md               # Complemento Nómina 1.2 rev. E
    │   ├── complemento-carta-porte-31.md          # Complemento Carta Porte 3.1
    │   ├── complemento-comercio-exterior.md       # Complemento Comercio Exterior
    │   ├── cfdi-global-publico-general.md         # CFDI Global / Público en General
    │   ├── catalogs-and-validation.md             # Common SAT catalogs + PAC rejection causes
    │   └── sap-architecture-edocument-vs-drc.md   # eDocument Cockpit vs. DRC architectural split
    └── templates/
        └── new-complement-onboarding-checklist.md # Checklist for onboarding a new SAT complement version
```

## Related skills

- **sap-btp-integration-suite** — CPI iFlows routing CFDI stamping requests to a PAC, bank H2H files, or OEM endpoints.
- **sap-abap** — BAdI implementations filling custom CFDI/Carta Porte fields in the eDocument Cockpit.
- **sap-abap-cds** — CDS views exposing CFDI/UUID data for embedded analytics or DRC reporting.

## Companion project

The [Vanilla Agents](https://github.com/MexbaliaMX/VanillaAutoMX) framework's `CFDI` agent references this skill for the generic SAT standard rather than re-documenting it, and owns the automotive-specific application on top (Carta Porte trigger analysis for vehicle/parts logistics, Comercio Exterior applicability, multi-entity RFC assignment).

## License

Proprietary — VanillaAutoMX Project.
