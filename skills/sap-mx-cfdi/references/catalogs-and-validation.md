# Common SAT Catalogs & Validation Patterns

**Source:** Anexo 20 Apéndice 3/4 and the per-complement Apéndice "Catálogos del comprobante" sections referenced throughout this skill.

## 1. Catalogs Every CFDI Touches

| Catalog | Used By | Notes |
|---|---|---|
| `c_TipoDeComprobante` | Base CFDI | `I`/`E`/`T`/`N`/`P`. |
| `c_FormaPago` | Base CFDI, Complemento de Pago (`FormaDePagoP`) | Never use `"99"` (Por definir) on a Complemento de Pago. |
| `c_MetodoPago` | Base CFDI | `PUE`/`PPD` — drives whether a Complemento de Pago is needed later. |
| `c_UsoCFDI` | Base CFDI `Receptor` | Cross-validated against both `RegimenFiscalReceptor` and `TipoDeComprobante` — a value valid for one receptor regime can be invalid for another even on an identical transaction. |
| `c_ClaveProdServ` | `Concepto` | SAT's product/service catalog; automotive-specific branches exist for new/used vehicles, parts, and labor — confirm the dealer's ERP product master maps to the *current* catalog version, since SAT periodically deprecates and replaces codes. |
| `c_ClaveUnidad` | `Concepto` | Unit of measure (e.g., "H87" = piece). |
| `c_Moneda` | Base CFDI, Pagos, Comercio Exterior | Publishes decimal precision and FX variance bands per currency; never use `"XXX"` (no-currency code) on a real transaction. |
| `c_CodigoPostal` / `c_Estado` / `c_Municipio` / `c_Localidad` / `c_Colonia` | `LugarExpedicion`, `DomicilioFiscalReceptor`, Carta Porte `Ubicacion`, Comercio Exterior `Domicilio` | Hierarchically cross-validated — postal code must belong to the stated municipality, which must belong to the stated state. |
| `c_RegimenFiscal` | `Emisor`/`Receptor` | Filtered by person type (Física/Moral) — a regime valid for one is invalid for the other; this is a common onboarding-time data error when migrating customer masters. |
| `c_ObjetoImp` | `Concepto`, Pagos `DoctoRelacionado` | `"02"` (sí objeto) mandates a child taxes node; `"01"`/`"03"` forbid one. |
| `c_Pedimento` / `c_ClavePedimento` | Comercio Exterior, Carta Porte `RegimenAduaneroCCP` | Customs declaration references — these are the catalogs SAT updates most frequently (multiple times a year); a stale local copy is the single most common Carta Porte/Comercio Exterior PAC rejection. |
| `c_Pais` | Comercio Exterior, Carta Porte | ISO 3166-1 based, with SAT-specific tax-ID-format and online-verification metadata per country. |

## 2. The `TipoFactor` / `TasaOCuota` / `Importe` Pattern

Every tax-bearing node in every complement described in this skill — base CFDI `Concepto/Impuestos`, Complemento de Pago `ImpuestosDR`/`ImpuestosP`, etc. — follows the same three-field pattern:

```
TipoFactor:   "Tasa" | "Cuota" | "Exento"
TasaOCuota:   numeric rate/amount; required when TipoFactor ∈ {Tasa, Cuota}; forbidden when "Exento"
Importe:      computed tax amount; required when TipoFactor ∈ {Tasa, Cuota}
```
`TasaOCuota`, when fixed (e.g., IVA 16% = `0.160000`), must match a `c_TasaOCuota` catalog entry for that `Impuesto`+`TipoFactor` pair exactly — do not round or reformat it.

## 3. Common PAC Rejection Causes (cross-complement)

| Symptom | Root Cause | Where Documented |
|---|---|---|
| `Receptor` rejected: RFC/regime/postal-code mismatch | Customer master's fiscal data out of sync with SAT's current registry | `cfdi-40-anexo20-core.md` §2 |
| Carta Porte / Comercio Exterior pedimento-catalog rejection | Local catalog file older than SAT's latest publication | `complemento-carta-porte-31.md` §3, `complemento-comercio-exterior.md` |
| Complemento de Pago issued on a `PUE` invoice | Confusing `FormaPago`-at-sale with deferred-payment settlement | `complemento-pagos-20.md` |
| Nómina rejected under the 2026 error matrix | Wage-type mapping not updated for revisión E (zero-value prohibition, code 038, new codes 054/055/108-111) | `complemento-nomina-12.md` |
| `UsoCFDI` rejected for a valid-looking value | `UsoCFDI` valid for the `TipoDeComprobante` but not for the receptor's specific `RegimenFiscalReceptor` (or vice versa) | `cfdi-40-anexo20-core.md` §2 |

## 4. Where to Get the Authoritative, Current Catalog Files

Always pull catalogs directly from SAT's published XSD/XLSX files referenced in each complement's official guide (see `SKILL.md` `sources:`) rather than hand-maintaining a copy — every catalog above is republished by SAT independently of the XSD schema version, on its own cadence.
