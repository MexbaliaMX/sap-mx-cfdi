# CFDI Global (Público en General) — Version 4.0

**Source:** SAT, "Guía de llenado del CFDI global versión 4.0" (44 pp.).

## What It's For

Consolidates retail/cash sales to customers who don't request an individual CFDI into a single periodic comprobante issued to the generic receptor RFC `XAXX010101000`. Relevant to a dealer group mainly for **parts counter retail sales** and **incidental cash service transactions** — not vehicle sales, which always require an individually identified `Receptor` (financing, registration, and warranty all depend on it).

## Key Rules

| Rule | Detail |
|---|---|
| **Issuance deadline** | Within 24 hours of closing the operations it consolidates. |
| **Periodicity** | Diaria, semanal, mensual, or bimestral — chosen by the issuer, consistent for the type of operation. |
| **$100 MXN threshold** | If the customer doesn't request a receipt and the amount is under $100, the issuer is not obligated to issue an individual CFDI — but **must still include the sale inside the corresponding period's CFDI Global**. Never simply omit it. |
| **Tax breakout** | IVA and IEPS amounts must be desglosados (broken out) explicitly and separately within the Global CFDI, not folded into a single tax line. |
| **`FormaPago`** | Use the form of payment that settled the **largest single amount** among the consolidated receipts (same tie-break rule as the base CFDI — see `cfdi-40-anexo20-core.md` §1). |

## Structural Note

The Global CFDI uses the **same base `Comprobante` schema as a normal CFDI 4.0** (it is not a separate complement/root node like Carta Porte or Retenciones) — the distinguishing factors are the generic `Receptor` RFC and a dedicated `InformacionGlobal` node (`Periodicidad`/`Meses`/`Año` attributes per the SAT guide), itself a base-schema child of `Comprobante`, not a `Complemento`, that breaks down the consolidated operations by period. Treat this file as a thin overlay on `cfdi-40-anexo20-core.md`, not a parallel standard.

## Automotive Application

If parts-counter POS terminals in a multibrand branch process walk-in cash sales below the $100 threshold without capturing an RFC, the **L2C billing process must still roll those tickets into the branch's periodic CFDI Global** — confirm the POS-to-CFDI integration captures every ticket, not just the ones a customer explicitly asked to be billed individually, or revenue silently leaks out of the CFDI trail (a SAT audit-exposure issue, not just a reporting gap).
