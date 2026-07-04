# Complemento Carta Porte 3.1

**Sources:** SAT, "Carta Porte 3.1" technical standard (132 pp.) and "Instructivo Complemento Carta Porte — Autotransporte" (107 pp.); SAT catalog updates of Aug 7, 2025 and Jan 13, 2026; SAT FAQ on the 30 km / C2 exemption.
**Version documented:** 3.1, mandatory since July 17, 2024. **Catalog currency: re-verify** — SAT updates the underlying catalogs (not the XSD schema) every few months; the most recent update as of this writing (Jan 13, 2026) added ~3,912 new pedimento relationships and is required for any 2026-dated transaction.

## 1. When Carta Porte Is Required

Two distinct trigger paths:
1. **Transportista (carrier) providing a transport service** → CFDI Ingreso/Traslado with Carta Porte, for the leg(s) traveled on a *vía general de comunicación* (federal highway, railway, navigable waterway, or air).
2. **Owner moving its own goods** with its own vehicles, no third-party carrier → CFDI Traslado with Carta Porte.

### Exemption (do not skip Carta Porte by mistake, but also don't over-apply it)
If the vehicle does **not exceed a C2-class truck** (per NOM-012-SCT-2-2017) **and** the federal-highway portion of the route (origin to final destination, including intermediate stops) **does not exceed 30 km**, the CFDI can be issued **without** the Carta Porte complement. This 30 km/C2 exemption also covers towing, salvage, and vehicle-deposit services, and **transporting a vehicle under its own power** (i.e., driving a vehicle rather than hauling it) within the same 30 km radius.

**Automotive relevance:** a short intra-city dealer-to-dealer parts run by a courier van is likely exempt; an OEM-plant-to-distributor vehicle delivery or any inter-entity STO transfer crossing real distance on a federal highway is not. Always check the actual route, not just "did we cross a state line."

## 2. Core Structure

```
CartaPorte (root, Version="3.1", IdCCP, TranspInternac, ...)
  ├── RegimenesAduaneros (0,1)
  │     └── RegimenAduaneroCCP (1..10)   — part of the 3.1 schema since its July 2024 introduction; up to 10 customs regimes per transfer
  ├── Ubicaciones (1,1)
  │     └── Ubicacion (2..unbounded)     — at least one Origen + one Destino
  ├── Mercancias (1,1)
  └── FiguraTransporte (0,1)
```

### `CartaPorte` root attributes
| Attribute | Notes |
|---|---|
| `Version` | Fixed `"3.1"`. |
| `IdCCP` | 36-char RFC-4122-style UUID identifying this Carta Porte instance, prefixed `CCC` per the documented pattern. |
| `TranspInternac` | `"Sí"` / `"No"` — international transport. |
| `EntradaSalidaMerc`, `PaisOrigenDestino`, `ViaEntradaSalida` | Conditional, required when `TranspInternac = "Sí"`. |
| `TotalDistRec` | Sum of all `DistanciaRecorrida` values, km, 0.01-99999. This is the field to check against the 30 km exemption threshold. |
| `RegistroISTMO`, `UbicacionPoloOrigen/Destino` | Only for transport through the Istmo de Tehuantepec development corridor — generally N/A for an automotive distributor unless a branch sits in that corridor. |

### `RegimenAduaneroCCP` (customs regime per merchandise)
Each `RegimenAduaneroCCP` node carries one `RegimenAduanero` key from `catCartaPorte:c_RegimenAduanero`. Up to **10** regimes are registrable per transfer — a schema capability of Carta Porte 3.1 itself since its July 2024 introduction, not a change from the Jan-2026 catalog update (that update was catalog-data-only: ~3,912 new `c_NumPedimentoAduana` pedimento-relationship entries for fiscal year 2026 — see the file header). Registering multiple regimes is relevant when a single shipment mixes, e.g., a definitive-import vehicle and a temporary-import demo unit.

### `Ubicacion` (Origen / Destino, ≥2 required)
| Attribute | Notes |
|---|---|
| `TipoUbicacion` | `"Origen"` or `"Destino"`. |
| `IDUbicacion` | Pattern `(OR\|DE)[0-9]{6}` — issuer-assigned. |
| `RFCRemitenteDestinatario` | **Required.** RFC of whoever ships from / receives at this location. For an inter-entity transfer, this is the *issuing legal entity's RFC at Origen* and the *receiving legal entity's RFC at Destino* — never a shared group RFC. |
| `FechaHoraSalidaLlegada` | Required, ISO datetime. |
| `DistanciaRecorrida` | Conditional, km between this location and the next. |
| `NumRegIdTrib`, `ResidenciaFiscal` | Only for foreign remitente/destinatario. |

### `Mercancias` / `Mercancia`
Each declared good ties back to a `Concepto` in the base CFDI via matching `NoIdentificacion`. For vehicles, this is typically the VIN or the dealer's internal stock number; weight/volume/unit fields follow `c_ClaveUnidad`/`c_Pedimento` rules consistent with the base CFDI.

### `FigurasTransporte` / `FiguraTransporte`
Driver/operator and vehicle (placas, permiso SCT) detail — required whenever `ViaTransporte` (autotransporte) applies; the Autotransporte-specific instructivo (107 pp.) covers tractor/trailer/semi-trailer configurations, axle counts, and insurance policy fields not summarized here in depth.

## 3. Common PAC Rejection Causes
- Stale `c_RegimenAduanero`/pedimento-relationship catalog (most common after a SAT catalog update — confirm the integration's catalog file date against SAT's published update date before blaming the data).
- `RFCRemitenteDestinatario` not matching the actual legal entity at that physical location (a frequent multi-entity defect — see `cfdi-40-anexo20-core.md` §2 for the parallel `Receptor` RFC-mismatch pattern).
- Missing Carta Porte when the 30 km/C2 exemption was incorrectly assumed to apply to a longer or heavier shipment.
