# CFDI que Ampara Retenciones e Información de Pagos — Anexo 20 §II

**Source:** SAT, "Anexo 20" §II.
**Version documented:** 2.0.

## What This Is — and What It Is Not

This is a **standalone CFDI document type**, distinct from the regular `Comprobante` schema documented in `cfdi-40-anexo20-core.md` — its root node is `Retenciones`, not `Comprobante`, and it is issued to certify **withholdings** (retenciones) the issuer made on a payment to the receptor, plus informational detail about that payment. It is unrelated to, and must not be confused with, the **Complemento de Pago** (`complemento-pagos-20.md`), which certifies *receipt* of a payment against an existing invoice.

## When It Applies

Issued when a withholding obligation exists outside of payroll (which uses `complemento-nomina-12.md` instead) — e.g.:
- Servicios Profesionales (clave `01`) and Regalías por Derechos de Autor (clave `02`) withholdings.
- Dividend distributions, interest payments to foreign residents, and other withholding scenarios under `c_CveRetenc`.

## Core Structure

| Node / Attribute | Notes |
|---|---|
| `Retenciones` (root) | `Version = "2.0"`, `FolioInt`, `Sello`, `NoCertificado`, `Certificado`, `FechaExp`, `LugarExpRetenc` (postal code). |
| `CveRetenc` | Key from `c_CveRetenc`. Value `"25"` (Otro tipo de retenciones) requires a free-text `DescRetenc`. |
| `CfdiRetenRelacionados` / `TipoRelacion` / `UUID` | Optional — relates this document to a prior retenciones CFDI, e.g. `TipoRelacion = "04"` (Sustitución de los CFDI previos). |
| `Emisor` | `RfcE`, `NomDenRazSocE`, `RegimenFiscalE` (`c_RegimenFiscal`, filtered by person type). |
| `Receptor` / `NacionalidadR` | `"Nacional"` or `"Extranjero"` — branches into a `Nacional` (RFC-based) or `Extranjero` (foreign tax ID) sub-node. |

## Relevance to an Automotive Distributor Group

**Low, but not zero.** A dealer group's day-to-day vehicle/parts/service revenue never triggers this document — that traffic is entirely Ingreso/Egreso/Pago/Traslado. It can surface in edge cases: withholding on a foreign-resident royalty payment (e.g., a brand-licensing or technology-licensing arrangement with an OEM's parent), or withholding on professional-services fees paid to a foreign consultant. Treat this as a **R2R/Treasury exception process**, not a routine L2C/P2P integration point — do not build standing automation for it without a confirmed business case.
