# Mexico filter values

Spellings to try in `supplier_country`, `company_country`, `customs_office` and
`custom_regime`. Names in the pedimento data are Spanish, title case, usually without
accents. These are the conventional Spanish forms; the index itself is the authority.
When a filter returns zero rows, run the same power query with only the product filter
and `page_size: 1`, read the spelling from the returned row, and retry.

## Countries (Spanish, as commonly written in customs data)

| English | Spanish |
|---|---|
| United States | Estados Unidos De America |
| China | China |
| Germany | Alemania |
| Japan | Japon |
| South Korea | Corea Del Sur (also seen as Republica De Corea) |
| Canada | Canada |
| Spain | España (also seen as Espana) |
| Italy | Italia |
| France | Francia |
| United Kingdom | Reino Unido |
| Brazil | Brasil |
| India | India |
| Taiwan | Taiwan |
| Vietnam | Vietnam |
| Netherlands | Paises Bajos |
| Switzerland | Suiza |
| Malaysia | Malasia |
| Thailand | Tailandia |
| Colombia | Colombia |
| Chile | Chile |

Try the exact form first; if it fails, use a wildcard on the stable part, e.g.
`Corea*` or `*Unidos*`.

## Common aduanas (customs offices)

Import `customs_office` examples use the plain name (`Manzanillo`). Export
`customs_office` values may carry the SAT office code, e.g. `440 - Ciudad Acuna, Coahuila`.
Use a wildcard such as `*Manzanillo*` when unsure.

- Sea: Manzanillo, Lazaro Cardenas, Veracruz, Altamira, Ensenada, Progreso
- Northern border (truck and rail): Nuevo Laredo, Ciudad Juarez, Tijuana, Reynosa,
  Matamoros, Mexicali, Nogales, Piedras Negras, Ciudad Acuna, Colombia (Nuevo Leon)
- Inland and air: Aeropuerto Internacional De La Ciudad De Mexico, Guadalajara,
  Monterrey, Toluca, Queretaro, Pantaco

## Customs regimes (`custom_regime`)

Values may be a code alone (`A1`) or code plus description
(`RT - Retorno De Mercancias (Immex)`). Use a wildcard on the code, e.g. `A1*` or `IN*`.

- A1: definitive import
- A3: definitive import, specific cases
- IN: temporary import for IMMEX (maquiladora) programs
- RT: return of goods under IMMEX
- AF: definitive import of fixed assets by IMMEX companies
- F4: transfer within a bonded or strategic regime
- F5: change of regime, temporary to definitive
- H1: temporary import of goods to be returned in the same state
- H8: temporary import for vehicles and related

## Incoterms (`incoterm`)

Three-letter codes as declared: EXW, FCA, FOB, CFR, CIF, CPT, CIP, DAP, DPU, DDP.

## Transport (`transport_type`)

`Truck`, `Railway`, `Sea`. Air declarations may use `Air`; check with a 1-row query.

## Who pays freight (`who_pays_freight`)

Exactly `Foreign entity` or `Mexico entity`.
