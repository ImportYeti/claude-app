---
name: mexico-trade
description: "Search Mexico import and export customs declarations (pedimentos) from ImportYeti: Mexican importers and exporters, their foreign suppliers and buyers, customs brokers, aduanas (customs offices), RFC tax IDs, incoterms, customs regimes such as IMMEX, and individual declarations. Use when the question involves Mexico, pedimento, aduana, RFC, maquiladora, IMMEX, nearshoring, customs broker or agente aduanal, or importing into or exporting from Mexico. Trigger words include Mexico, Mexican, pedimento, aduana, broker, RFC, maquiladora, nearshoring."
---

# Mexico trade

Use this skill for anything about goods entering or leaving Mexico: who imports, who
exports, who their partners are, which customs broker or aduana handles it, and the
declarations themselves.

Not this skill: US bills of lading (shipment-records), US importers or exporters
(find-buyers, find-suppliers, research-company).

There are no company profiles for Mexico. Do not call `search_companies`,
`search_suppliers`, `get_company` or `get_supplier` for a Mexican company; say that
Mexico data has rankings and declarations only, then use the tools below.

## Inputs to collect

- Direction: into Mexico (import) or out of Mexico (export).
- What they want: a ranking (importers, exporters, suppliers, buyers, brokers), a list of
  declarations, or one declaration by ID.
- Filters: product wording (Spanish terms match best), HS code, company or supplier
  name, country, aduana, broker, RFC, incoterm, regime, transport type, period.

See `references/mx-values.md` for Spanish country names, common aduanas and regime codes.
Country and office names in this data are Spanish and title case, e.g.
`Estados Unidos De America`. When unsure of a spelling, run the matching power query with
`page_size: 1` (0.1 credit) and read the spelling from the returned row before filtering.

## Tools

| Need | Import side | Export side |
|---|---|---|
| Declarations list | `power_query_mx_import` | `power_query_mx_export` |
| One declaration by ID | `get_mx_import_declaration` | `get_mx_export_declaration` |
| Rank Mexican companies | `top_mx_import_companies` (Mexican importers) | `top_mx_export_suppliers` (Mexican exporters) |
| Rank foreign partners | `top_mx_import_suppliers` (foreign suppliers) | `top_mx_export_companies` (foreign buyers) |
| Rank customs brokers | `top_mx_import_brokers` | `top_mx_export_brokers` |

Field meaning differs by side. Import: `company_name` is the Mexican importer,
`supplier_name` the foreign supplier, `supplier_country` the origin, `company_state` the
importer's state, `customs_office` the destination aduana. Export: `supplier_name` is
the Mexican exporter, `company_name` the foreign buyer, `company_country` the
destination, `supplier_state` the exporter's state, `customs_office` the departure
aduana (often prefixed with its code, e.g. `440 - Ciudad Acuna, Coahuila`; use a
wildcard such as `*Acuna*`).

Shared filters on both sides: `product_description` (boolean syntax, e.g.
`Aluminio OR acero`), `hs_code` (wildcards, e.g. `8471*`), `weight`, `mx_value` and
`company_total_shipments` ranges (`"1000 TO *"`), `who_pays_freight` (`Foreign entity`
or `Mexico entity`), `pedimento_number`, `declaration_number`, `rfc_number`,
`transport_type` (`Truck`, `Railway`, `Sea`), `custom_broker`, `incoterm`,
`custom_regime`, `start_date`, `end_date`, `page_size`, `sort_by` (rankings only:
`weight` or `teu`).

## Steps

1. Ranking questions ("who imports the most", "largest exporters", "which brokers"):
   call the matching `top_mx_*` tool with the product or HS filter and any country, state
   or aduana filter, `page_size` 10. Do not page a power query and count rows.
2. Declaration lists ("show me shipments", "declarations by RFC", "what did X import"):
   call `power_query_mx_import` or `power_query_mx_export` with the narrowest filters,
   `page_size` 10. Report `totalCount` and ask before fetching page 2.
3. One declaration: call `get_mx_import_declaration` or `get_mx_export_declaration` with
   `declaration_id` taken from a power query row. Never guess an ID.
4. Omit dates unless the user named a period; "recent" means `start_date` 12 months ago
   in MM/DD/YYYY.
5. Use one tool per question. Stop when it is answered.

## Credits

Rankings and declaration lists cost 1 data credit per 10 rows; one declaration costs 0.1.
A spelling check with `page_size: 1` costs 0.1. Keep `page_size` at 10 and ask before
more pages. Mention the balance only when `creditsRemaining` is under 10 or the user
asks. On "Not enough credits", stop and point the user to https://www.importyeti.com.

## Present the result

Open with direction, filters and period. Rankings: a table with Company or broker |
State or country | Matching declarations | Total declarations | Weight or value.
Declarations: Date | Mexican party | Foreign party | Product | HS | Aduana | Broker |
Incoterm | Value. Keep product text to one line and show RFC when returned.

Mexican companies have no ImportYeti profile page, so show names in plain text. For a
foreign supplier that also ships to the US, offer to look it up with research-company.

Close with one or two observations (dominant aduana, broker concentration, freight payer
split). Say values are as declared on the pedimento. Do not invent contact details,
prices beyond declared value, MOQs or certifications.

## No results or errors

- Zero rows: the likeliest cause is an English or accented spelling. Check
  `references/mx-values.md`, or run the query with only the product filter and
  `page_size: 1` to read the real spelling, then retry once with the corrected filter.
- Declaration ID not found: say so and offer to re-run the list query that produced it.
- Tool error text: fix the argument it names and retry once, then report it plainly.
