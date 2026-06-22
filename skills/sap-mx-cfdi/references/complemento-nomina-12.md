# Complemento Nómina 1.2 (revisión E)

**Sources:** SAT, "Guía de llenado del comprobante del recibo de pago de nómina y su complemento" (133 pp.); KPMG México flash note on the 2026 catalog/validation changes (Dec 2025).
**Version documented:** 1.2, **revisión E mandatory since January 1, 2026** — this is the current revision; older 1.2 revisions (A-D) are no longer valid for 2026-dated payroll.

## Core Structure

```
Nomina (root, Version="1.2")
  ├── Emisor (RegistroPatronal, Curp...)
  ├── Receptor (TipoContrato, RiesgoPuesto, PeriodicidadPago...)
  ├── Percepciones
  │     ├── Percepcion (1..n)
  │     ├── JubilacionPensionRetiro (0,1)
  │     └── SeparacionIndemnizacion (0,1)
  ├── Deducciones
  │     └── Deduccion (1..n)
  ├── OtrosPagos
  │     └── OtroPago (1..n)
  └── Incapacidades (0,1)
```

### Root attributes
| Field | Notes |
|---|---|
| `Version` | Fixed `"1.2"`. |
| `TipoNomina` | `O` Ordinaria (periodic: diaria/semanal/quincenal/mensual/etc.) or `E` Extraordinaria (non-periodic: separación, aguinaldo, PTU, laudo). |
| `FechaPago` | Actual date the employer disbursed pay — varies by `FormaDePago`: efectivo = date of cash handover; cheque = check issuance date; transferencia = date the employer *ordered* the bank transfer (not when the employee's bank posted it). |
| `FechaInicialPago` / `FechaFinalPago` | Period covered; may be the same date for extraordinary payrolls (e.g., a same-day severance payment). |
| `NumDiasPagados` | Days/fraction paid, up to 3 decimals; use `"1"` when no day-level detail is determinable for that payment. |
| `TotalPercepciones` | Must equal `TotalSueldos + TotalSeparacionIndemnizacion + TotalJubilacionPensionRetiro`. Omit entirely if the receipt contains only `OtrosPagos`. |
| `TotalDeducciones` | Must equal `TotalOtrasDeducciones + TotalImpuestosRetenidos`. Omit if there are no deductions. |
| `TotalOtrosPagos` | Sum of the `OtrosPagos` section (e.g., Subsidio al Empleo). |

**Invariant:** at least one of `TotalPercepciones` or `TotalOtrosPagos` must be present, and the base CFDI's `Total` field may never be negative.

## 2026 Revision E — What Changed (mandatory Jan 1, 2026)

| Area | Change |
|---|---|
| **Zero-value prohibition** | A Percepción's `ImporteExento` and `ImporteGravado` can no longer both be reported as zero for the same concept — at least one must carry the actual amount. |
| **Code 038 (Other salary income)** | Must now be reported entirely as taxable; marking any portion exempt now triggers rejection under the new error matrix. |
| **Subsidio al Empleo (causado)** | Increased from MXN 475.00 to MXN 628.00. |
| **New Deducciones codes 108-111** | Adjustments for worked rest days (standard and mandatory, taxable and tax-exempt variants). |
| **New Percepciones codes 054 / 055** | Worked rest day / worked mandatory rest day. |

These changes affect every payroll posted with a `FechaPago` in 2026 and the corresponding annual ISR filings — re-validate any pre-built wage-type-to-`c_TipoPercepcion`/`c_TipoDeduccion` mapping table against this list before the first 2026 payroll run, not just at year-end.

## Automotive Relevance

This is the technical layer underneath the Vanilla `H2R` agent's `h2r_payroll_compensation_compliance.md` §4 (CFDI de Nómina 4.0). In particular, the new codes 054/055 (worked rest days) and the 108-111 deduction adjustments are directly relevant to **workshop technicians on flat-rate schedules who work a scheduled rest day** — H2R's wage-type mapping should be checked against this table specifically for that population.
