# Complemento Comercio Exterior

**Source:** SAT, "Guía de llenado del comprobante fiscal al que se le deberá incorporar el complemento para Comercio Exterior" (79 pp.).
**Version note:** "1.1" and "2.0" are two distinct, sequential SAT versions, not an informal-name-vs-official-name naming-drift issue. Version 1.1 was incorporated with CFDI 4.0 and was mandatory from **April 1, 2023**. SAT published version 2.0 on Dec 19, 2023, with a hard, no-transition-period cutover: version 1.1 stopped being valid and version 2.0 became mandatory on **January 18, 2024**. The current official guide's `ComercioExterior` root attribute requires the literal value `"2.0"` — verify against the live SAT guide URL in `SKILL.md`'s `sources:` before any go-live, since a system still stamping with the retired 1.1 schema will get rejected.
**Mandatory since:** January 18, 2024 (version 2.0; version 1.1 was mandatory April 1, 2023 – January 17, 2024).

## When It Applies

Mandatory for **definitive export of merchandise under pedimento key `A1`, when the export constitutes a sale (enajenación)** — i.e., `Exportacion = "02"` on the base CFDI. Optional (but then must be accompanied by an "acuse de valor" transmission) for non-sale movements like sample shipments or third-party-owned goods in transit.

Coexists with: Timbre Fiscal Digital, Otros derechos e impuestos, Leyendas fiscales, CFDI registro fiscal, and **Carta Porte** — an exported vehicle physically leaving Mexico by truck plausibly carries both complements simultaneously.

## Core Structure

```
ComercioExterior (root, Version="2.0")
  ├── Emisor (Curp, Domicilio)
  ├── Propietario (0..n)         — only when MotivoTraslado="05" (goods owned by a third party)
  ├── Receptor (NumRegIdTrib, Domicilio)
  ├── Destinatario (0,1)          — only when different from Receptor, or a branch address
  └── Mercancias
        └── Mercancia (1..n)
```

### Root attributes
| Field | Notes |
|---|---|
| `MotivoTraslado` | Only for `TipoDeComprobante = "T"`. `"01"` (previously-invoiced goods now shipped) requires a `CfdiRelacionados` entry with `TipoRelacion = "05"`. `"05"` (third-party-owned goods) requires ≥1 `Propietario` node. |
| `ClaveDePedimento` | Must be `"A1"` for this complement's primary (sale-triggered) use case. |
| `CertificadoOrigen` | `"0"` No funge / `"1"` Sí funge as a certificate of origin under an applicable Mexico FTA. `NumCertificadoOrigen` is required iff this is `"1"`; forbidden iff `"0"`. |
| `Incoterm` | Required only when `Exportacion = "02"` (definitive, pedimento A1); omit entirely when `Exportacion = "04"`. |
| `TipoCambioUSD` / `TotalUSD` | USD equivalent per CFF Art. 20; `TotalUSD` = sum of all `Mercancia/ValorDolares`. |

### `Receptor` / `Destinatario`
- `Receptor.NumRegIdTrib` is required **only** when the base CFDI's `Receptor.Rfc = "XEXX010101000"` (generic foreign receptor) — otherwise it must be absent.
- `Destinatario` is a separate node from `Receptor`, used when the merchandise physically goes to a different party or branch address than the invoiced receptor. For `TipoDeComprobante = "T"`, only one `Destinatario` may be registered.
- Both nodes carry a full `Domicilio` (Calle, Colonia/Localidad/Municipio/Estado per SAT catalogs when `Pais = "MEX"`, free text otherwise, `CodigoPostal`).

### `Mercancia`
- `NoIdentificacion` must match the corresponding base-CFDI `Concepto`'s identifier — for a vehicle export this is typically the VIN.
- For `TipoDeComprobante = "I"`/`"E"`, the sum of `Importe` across concepts sharing a `NoIdentificacion`, converted to the comprobante's currency, must reconcile against the sum of that merchandise's `ValorDolares`.
- `FraccionArancelaria` (HS/tariff classification) and `CantidadAduana`/`UnidadAduana`/`ValorUnitarioAduana` are customs-specific fields distinct from the commercial quantity/unit/price already on the base `Concepto`.

## Automotive Relevance

Applies only if a legal entity in the group **exports** vehicles or parts directly (e.g., a Mexican plant/distributor entity re-exporting to another Latin American market) rather than purely **importing** through a Mexican NSC/importer-of-record relationship. If the group's entities are pure importers/retailers with no export activity, this complement is out of scope — confirm this assumption explicitly with the client rather than building for it speculatively (see the Vanilla `CFDI` agent's `cfdi_comercio_exterior_vehicle_export.md` for the applicability checklist).
