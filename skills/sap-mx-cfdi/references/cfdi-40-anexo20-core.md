# CFDI 4.0 Core — Anexo 20 §I (Comprobante)

**Source:** SAT, "Anexo 20 — Guía de llenado de los comprobantes fiscales digitales por Internet" (120 pp.), §I.
**Version documented:** CFDI 4.0 (available/optional from Jan 1, 2022, coexisting with 3.3; mandatory since Apr 1, 2023 after two SAT extensions — 3.3 decommissioned as of that date).

## 1. Comprobante Node — Required/Conditional Attributes

| Attribute | Rule |
|---|---|
| `Version` | Fixed value `"4.0"`. |
| `Serie` / `Folio` | Internal control numbers, alphanumeric, 1-25 / 1-40 chars. Not SAT-validated for uniqueness across PACs. |
| `Fecha` | Issuance timestamp, `AAAA-MM-DDThh:mm:ss`, local time of issuance. |
| `Sello` | Digital seal generated with the issuer's CSD (Certificado de Sello Digital). |
| `FormaPago` | Required **only if payment was received at the moment of issuance** — `c_FormaPago` key. If paid later/in parts, omit and use the Complemento de Pago instead; do not emit both for the same operation. If more than one payment form was used in one transaction, register the key for whichever form settled the largest amount. |
| `CondicionesDePago` | Optional free text (e.g., "30 días"). |
| `SubTotal` / `Descuento` / `Total` | Numeric, must reconcile with the sum of `Conceptos`. |
| `Moneda` | `c_Moneda` key; `TipoCambio` required whenever `Moneda` ≠ `MXN`. |
| `TipoDeComprobante` | `I` Ingreso, `E` Egreso, `T` Traslado, `N` Nómina, `P` Pago — drives which complement, if any, is mandatory. |
| `Exportacion` | **New in 4.0.** `01` No aplica, `02` Definitiva (pedimento A1), `03` Temporal, `04` Definitiva (otra clave de pedimento o sin enajenación). Drives whether `ComercioExterior` complement is required. |
| `MetodoPago` | `PUE` (Pago en una sola exhibición) or `PPD` (Pago en parcialidades o diferido) — `PPD` is the trigger for the Complemento de Pago downstream. |
| `LugarExpedicion` | Postal code of issuance, must be a valid `c_CodigoPostal` key. |
| `Confirmacion` | Conditional — only when an amount exceeds the SAT-published threshold for its currency conversion (requires a confirmation key issued non-automatically by SAT, not yet broadly enforced as of this writing — re-verify). |

## 2. Emisor / Receptor (new in 4.0)

CFDI 4.0 added **mandatory `RegimenFiscalReceptor`** and **mandatory `DomicilioFiscalReceptor`** (postal code only) on the `Receptor` node — both must match SAT's RFC registry exactly, or the PAC rejects the stamping request. This is the single most common 4.0 migration defect: a `Receptor` whose registered tax regime or fiscal postal code in the issuer's CRM/customer master doesn't match SAT's current record.

| Node | Attribute | Rule |
|---|---|---|
| `Emisor` | `Rfc`, `Nombre`, `RegimenFiscal` | Must match the issuer's own SAT registration exactly. |
| `Receptor` | `Rfc`, `Nombre` | For the generic public-general receptor, use `XAXX010101000` (national) or `XEXX010101000` (foreign). |
| `Receptor` | `DomicilioFiscalReceptor` | **Required, 4.0-only.** Postal code of the receptor's registered fiscal address. |
| `Receptor` | `RegimenFiscalReceptor` | **Required, 4.0-only.** Must be a regime valid for the receptor's person type (`c_RegimenFiscal`, filtered by Física/Moral column). |
| `Receptor` | `UsoCFDI` | Must be compatible with both the receptor's `RegimenFiscalReceptor` and the `TipoDeComprobante` (`c_UsoCFDI` cross-validates against both). |

## 3. Conceptos / Impuestos

- Each `Concepto` requires `ClaveProdServ` (`c_ClaveProdServ`, SAT's product/service catalog — tens of thousands of entries; vehicle sales typically use the automotive-specific branch of this catalog), `ClaveUnidad`, `Cantidad`, `ValorUnitario`, `Importe`, `ObjetoImp`.
- `ObjetoImp = "02"` (sí objeto de impuesto) requires a child `Impuestos` node on the `Concepto`; `"01"`/`"03"` must **not** carry one.
- `Impuestos/Traslados` and `Impuestos/Retenciones` follow the same `TipoFactor` (Tasa/Cuota/Exento) + `TasaOCuota` + `Importe` pattern used throughout every complement described in this skill (Pagos, Nómina) — learning this pattern once here means recognizing it instantly in the others.

## 4. Apéndice 5 — CFDI Egreso

Egreso (`TipoDeComprobante = E`) is used for credit notes, discounts, and returns. It must reference the original Ingreso via `CfdiRelacionados`/`CfdiRelacionado` with `TipoRelacion` from `c_TipoRelacion` (commonly `01` Nota de crédito de los documentos relacionados).

## 5. Apéndice 6 — Anticipos (Advance Payments)

Two valid mechanisms, **never mixed within the same operation**:
- **(a) CFDI por el anticipo, then CFDI por el remanente**, relating the second to the first via `CfdiRelacionados` with `TipoRelacion = "07"` (CFDI por aplicación de anticipo).
- **(b) Issue the full-value CFDI at zero `FormaPago`/`MetodoPago` and settle via the Complemento de Pago** for each amount received — used when the final price isn't known at the time the advance is received.

## 6. Apéndice 4 — Comprobante Catalogs (selected, see `catalogs-and-validation.md` for the full cross-reference)

`c_TipoDeComprobante`, `c_FormaPago`, `c_MetodoPago`, `c_UsoCFDI`, `c_ClaveProdServ`, `c_ClaveUnidad`, `c_Moneda`, `c_CodigoPostal`, `c_RegimenFiscal`, `c_Exportacion`, `c_ObjetoImp`, `c_TipoRelacion`.

## 7. CFDI Traslado (`TipoDeComprobante = "T"`)

Used for goods movement without a sale/enajenación (no `Total`, conceptually a delivery note made fiscal) — e.g., consignment, demo-unit loans, samples, or single-RFC intra-entity logistics. **Not** the right type when an agreed transfer price changes hands between two different legal entities (RFCs): that price is enajenación, so the issuing entity invoices with CFDI de **Ingreso** instead, carrying the Carta Porte complement when the route requires it — see `complemento-carta-porte-31.md` §1 for the transportista/owner trigger paths, and the Vanilla `CFDI` agent's `cfdi_carta_porte_vehicle_logistics.md` §1.4 for the automotive multi-entity application.
