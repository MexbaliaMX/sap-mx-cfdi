# Complemento de Pago 2.0 ("Recibo Electrónico de Pago")

**Source:** SAT, "Guía de llenado del comprobante fiscal al que se le incorporará el Complemento para recepción de pagos" (111 pp.).
**Version documented:** 2.0, mandatory since April 1, 2023; compatible only with CFDI 4.0.

## When It Applies vs. When It Doesn't

- **Use it** when an invoice was issued with `MetodoPago = "PPD"` (Pago en parcialidades o diferido) — i.e., payment wasn't received at the moment of issuance — for every payment subsequently received against that invoice.
- **Don't use it** when payment was received at the moment of sale (`MetodoPago = "PUE"`) — in that case `FormaPago` on the original CFDI already reflects it; issuing a redundant Complemento de Pago on top of a PUE invoice is a common, avoidable error.
- The CFDI carrying this complement is issued with `Total = "0"` and **no** `FormaPago`/`MetodoPago` values — those concepts move into the complement's own `Pago` node instead.
- **Issuer rule:** the CFDI with the Complemento de Pago must be issued by **whoever received the payment**, not the original invoice issuer if those differ (e.g., factoring).
- **Payment-application deadline:** absent an explicit agreement otherwise, the payer has **5 días naturales** following the payment date to indicate which invoice(s) it applies to.

## Core Structure

```
Pagos (root, Version="2.0")
  ├── Totales (1,1)         — MXN-denominated tax/amount summary across all Pago nodes
  └── Pago (1..n)            — one per distinct form of payment received
        └── DoctoRelacionado (1..n)   — one per invoice this payment settles
              └── ImpuestosDR (0,1)
                    ├── RetencionesDR / RetencionDR
                    └── TrasladosDR / TrasladoDR
```

### `Totales`
| Field | Notes |
|---|---|
| `TotalRetencionesIVA/ISR/IEPS` | Sum of withheld tax, computed from `DoctoRelacionado/ImpuestosDR`, converted via each `Pago`'s `TipoCambioP`. |
| `TotalTrasladosBaseIVA16/8/0/Exento`, `TotalTrasladosImpuestoIVA16/8/0` | Tax-rate-bucketed transferred-tax summary — **new in 2.0** (this granular rate breakout didn't exist in 1.0). |
| `MontoTotalPagos` | Sum of every `Pago/Monto`, currency-converted. |

### `Pago` node (one per payment/form-of-payment combination)
| Field | Notes |
|---|---|
| `FechaPago` | When the beneficiary actually received the payment (ISO datetime; `12:00:00` if time unknown). |
| `FormaDePagoP` | `c_FormaPago` key, never `"99"` (Por definir). |
| `MonedaP` / `TipoCambioP` | `TipoCambioP = "1"` when `MonedaP = "MXN"`; otherwise required and subject to a SAT-published variance band that can trigger a non-automatic confirmation key. |
| `Monto` | Must be > 0; sum of related `DoctoRelacionado/ImpPagado` (converted to payment currency) must be ≤ `Monto`. |
| `NumOperacion` | Check #, SPEI clave de rastreo, or internal reference, 1-100 chars. |
| `RfcEmisorCtaOrd`, `NomBancoOrdExt`, `CtaOrdenante` | Originating account detail; foreign issuers use generic RFC `XEXX010101000`. |
| `RfcEmisorCtaBen`, `CtaBeneficiario` | Beneficiary (receiving) account detail. |
| `TipoCadPago`, `CertPago`, `CadPago`, `SelloPago` | SPEI-style payment-chain proof — all four are required together or all four absent; never partial. |

### `DoctoRelacionado` (per invoice settled by this payment)
| Field | Notes |
|---|---|
| `IdDocumento` | The invoice's UUID, 16-36 alphanumeric chars. |
| `MonedaDR`, `EquivalenciaDR` | `EquivalenciaDR = "1"` when `MonedaDR` matches `MonedaP`; otherwise required FX rate, up to 10 decimals. |
| `NumParcialidad` | Which installment this payment represents. |
| `ImpSaldoAnt` | Outstanding balance **before** this payment (= full invoice amount if this is installment #1, or the full deferred amount for a single deferred payment). |
| `ImpPagado` | Amount applied to this document by this payment, > 0. |
| `ImpSaldoInsoluto` | `ImpSaldoAnt − ImpPagado`, ≥ 0. |
| `ObjetoImpDR` | `c_ObjetoImp` key; `"02"` requires a child `ImpuestosDR`, `"01"`/`"03"` must omit it. |

## Common PAC Rejection Causes
- Issuing the complement on a `PUE` invoice (logically invalid — most PACs reject or, worse, silently accept a meaningless document).
- `ImpPagado` sum exceeding `Monto` after FX conversion — the standard's rounding-margin formula (documented in the source PDF, §`Monto`) exists specifically to tolerate small FX rounding differences; a mismatch beyond that margin is a real data error, not a rounding artifact.
- Missing the 4-field SPEI proof set (`TipoCadPago`/`CertPago`/`CadPago`/`SelloPago`) when only one or two are populated.
