---
name: sap-mx-cfdi
description: "Comprehensive reference for Mexico's CFDI 4.0 electronic invoicing standard (Anexo 20) and its complements: Complemento de Pago 2.0, Complemento Nómina 1.2 (rev. E), Complemento Carta Porte 3.1, Complemento Comercio Exterior, CFDI Global (Público en General), and the standalone CFDI that amparas retenciones e información de pagos. Use when filling out, validating, or troubleshooting any CFDI document or complement, when checking which node/attribute/catalog applies, when determining which SAT version is currently mandatory, or when distinguishing SAP's eDocument Cockpit (CFDI/Carta Porte stamping) from Document and Reporting Compliance / DRC (Electronic Accounting + VAT/DIOT reporting)."
license: Proprietary
metadata:
  maintainer: "VanillaAutoMX Project"
  version: "1.0.0"
  last_verified: "2026-06-22"
  sources:
    - "https://www.sat.gob.mx/ (Anexo 20, Guía de llenado CFDI, Guía de llenado CFDI global)"
    - "http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/Guia_llenado_pagos.pdf"
    - "http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/Guia_llenado_Nomina.pdf"
    - "http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/Guia_complemento_Comercio_Exterior.pdf"
    - "http://omawww.sat.gob.mx/tramitesyservicios/Paginas/complemento_carta_porte.htm"
    - "https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE (Document and Reporting Compliance, eDocument Cockpit Mexico)"
  keywords: [CFDI, CFDI 4.0, Anexo 20, SAT, Mexico, Complemento de Pago, Complemento Pagos, Complemento Nomina, CFDI de Nomina, Complemento Carta Porte, Complemento Comercio Exterior, CFDI Global, Publico en General, eDocument Cockpit, SAP DRC, Document and Reporting Compliance, PAC, UUID, c_FormaPago, c_UsoCFDI, c_ClaveProdServ]
---

# SAP MX CFDI — Mexico Electronic Invoicing Reference

## Related Skills

- **sap-btp-integration-suite**: Use for the CPI iFlows that route CFDI stamping requests to a PAC, bank H2H files, or OEM endpoints.
- **sap-abap**: Use for BAdI implementations that fill custom CFDI/Carta Porte fields in the eDocument Cockpit.
- **sap-abap-cds**: Use when exposing CFDI/UUID data through CDS views for embedded analytics or DRC reporting views.

## When to Use This Skill

Use this skill when:
- Filling out or validating a CFDI 4.0 document (Comprobante node) or one of its complements.
- Determining which complement applies to a given transaction (sale, payment, payroll, transport, export).
- Checking the current mandatory version/revision of a complement before a go-live or year-end cutover.
- Distinguishing which SAP framework is responsible for a given Mexico compliance task: **eDocument Cockpit** (transactional CFDI stamping) vs. **Document and Reporting Compliance / DRC** (periodic statutory reports) — see `references/sap-architecture-edocument-vs-drc.md`.
- Troubleshooting a PAC rejection or a field-level validation error.

This skill is **generic and industry-agnostic** — it documents the SAT standard and SAP's implementation of it, not any specific client's business processes. For automotive-distributor-specific application of this standard (vehicle logistics Carta Porte, multi-brand commission CFDI, etc.), see the `CFDI` agent in the Vanilla Agents framework, which references this skill rather than duplicating it.

## Current Mandatory Versions (verify before relying on this — SAT updates catalogs every few months)

| Document / Complement | Current Version | Mandatory Since | Reference File |
|---|---|---|---|
| CFDI (Comprobante) | 4.0 | Jan 1, 2022 | `references/cfdi-40-anexo20-core.md` |
| CFDI que ampara Retenciones e Información de Pagos | 2.0 | — | `references/cfdi-retenciones-pagos-info.md` |
| Complemento de Pago ("Pagos") | 2.0 | Apr 1, 2023 | `references/complemento-pagos-20.md` |
| Complemento Nómina | 1.2, revisión E | Jan 1, 2026 | `references/complemento-nomina-12.md` |
| Complemento Carta Porte | 3.1 | Jul 17, 2024 (catalogs updated Aug 7 2025 and Jan 13 2026) | `references/complemento-carta-porte-31.md` |
| Complemento Comercio Exterior | Version attribute = "2.0" per current SAT guide (commonly still referred to in industry literature as "1.1") | Jan 1, 2023 | `references/complemento-comercio-exterior.md` |
| CFDI Global (Público en General) | 4.0 | Jan 1, 2022 | `references/cfdi-global-publico-general.md` |

## File Index

| File | Covers |
|---|---|
| `references/cfdi-40-anexo20-core.md` | Base `Comprobante` node/attributes, Conceptos, Impuestos, Apéndice catalogs (Anexo 20 §I) |
| `references/cfdi-retenciones-pagos-info.md` | The standalone "CFDI que ampara retenciones e información de pagos" (Anexo 20 §II) — **not** the Complemento de Pago; low applicability to a vehicle dealer's day-to-day operations |
| `references/complemento-pagos-20.md` | Complemento de Pago 2.0 — PPD payment receipts |
| `references/complemento-nomina-12.md` | Complemento Nómina 1.2 rev. E — payroll receipts |
| `references/complemento-carta-porte-31.md` | Complemento Carta Porte 3.1 — goods-in-transit, incl. the 30 km / C2-truck exemption and the Jan-2026 `RegimenAduanero` catalog update |
| `references/complemento-comercio-exterior.md` | Complemento Comercio Exterior — export invoices with pedimento `A1` |
| `references/cfdi-global-publico-general.md` | CFDI Global — public-general / cash retail consolidation |
| `references/catalogs-and-validation.md` | Common SAT catalogs (`c_FormaPago`, `c_UsoCFDI`, `c_ClaveProdServ`, `c_Moneda`...) and common PAC rejection causes |
| `references/sap-architecture-edocument-vs-drc.md` | The eDocument Cockpit vs. DRC architectural split — which framework owns which document type |

## Version Compatibility

CFDI 4.0 is the only comprobante version SAT accepts as of this writing (3.3 was decommissioned). All complements referenced here are compatible with CFDI 4.0; none of them work with CFDI 3.3. Re-verify the "Current Mandatory Versions" table above against the SAT URLs in `sources:` before using this skill on a live engagement — SAT typically updates complement catalogs (not necessarily the XSD schema itself) every 2-6 months, and a stale catalog causes outright PAC rejection, not a soft warning.
