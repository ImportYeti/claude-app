---
name: shipment-records
description: "Look up individual US import or export shipments (bills of lading) from ImportYeti by BOL number, container number, vessel, carrier SCAC, port of entry or exit, destination country, product description, HS code, notify party or date. Use when the user wants actual shipment records rather than rankings or profiles: a specific bill of lading, what a container or vessel carried, what arrived at a port, recent shipments of a product, or exports of a product to a country. Trigger words include bill of lading, BOL, B/L, container, vessel, shipment records, arrivals, exports to, manifest, SCAC."
---

# Shipment records

Use this skill when the user wants the shipments themselves: one bill of lading, or a
list of US import or export shipments matching filters.

Not this skill: rankings of importers or suppliers (find-buyers, find-suppliers); an
overview, supplier list or trend for one company (research-company); Mexico declarations
(mexico-trade).

## Inputs to collect

- A BOL number, if the user has one. Import BOL numbers look like `CCLLMILS17011590`;
  export BOLs may be house or master numbers.
- Otherwise the filters: product wording or HS code, port, country, vessel, carrier,
  container, notify party, importer or supplier slug, and a period if named.
- Direction: into the US (import, default) or out of the US (export).

## Steps

1. One BOL by number: call `get_us_import_bol` with `bol_number`, or `get_us_export_bol`
   with `bol_number` for an export. If the direction is unknown, try import first. 0.1
   credit each.
2. Check spellings before filtering (US import only). Call the free `us_import_values`
   with `list` set to `countries` (feeds `supplier_country`), `entry_ports` (feeds
   `entry_port`), `shipping_ports` (feeds `exit_port`), `carriers` (feeds
   `carrier_scac_code`) or `bill_types` (feeds `bol_type`), and `search` for the term
   (e.g. `"korea"`, `"long beach"`). Use the spelling it returns.
3. US import list: call `power_query_us_import` with the narrowest set of filters and
   `page_size` 10. Useful filters: `product_description` (boolean syntax, e.g.
   `"lithium battery" AND NOT toy`), `product_description_raw` (original wording, for
   exact codes like `"Hs Code:850760"`), `hs_code` (wildcards, e.g. `8507*`),
   `entry_port`, `exit_port`, `supplier_country`, `vessel_name`, `carrier_scac_code`,
   `container_id`, `notify_party`, `weight` and `teu` ranges (`"500 TO *"`), and
   `company` or `supplier` set to a slug from `search_companies` / `search_suppliers`
   (never guessed). Omit `start_date` and `end_date` unless the user named a period;
   "recent" means `start_date` 12 months ago in MM/DD/YYYY, "last month" means that
   month's first and last day.
4. US export list: call `power_query_us_export` with `product_description`, `hs_code`,
   `destination_country`, `exit_port` (US port), `entry_port` (foreign port),
   `company_name` (exporter, plain name), `vessel_name`, `carrier_scac_code` or
   `bol_number`, `page_size` 10.
5. The response carries `totalCount`. Report it and ask before fetching another page
   (`page: 2`). Do not page through results to count or rank; use find-buyers or
   find-suppliers for that.

## Credits

A BOL lookup costs 0.1 data credit. A power query costs 1 data credit per 10 rows, so
one default page is 1 credit. Keep `page_size` at 10 unless the user asked for more.
Mention the balance only when `creditsRemaining` is under 10 or the user asks. On
"Not enough credits", stop and point the user to https://www.importyeti.com.

## Present the result

One BOL: a short record with arrival date, importer (linked to
https://www.importyeti.com/company/<slug> when a slug is present), supplier (linked to
/supplier/<slug>), product description, HS code, origin and entry ports, vessel, carrier,
container numbers, weight and quantity.

A list: open with the filters used, the period and `totalCount`, then a table with Date |
Importer | Supplier | Product | HS | From port | To port | Weight | Containers | BOL
number. Keep product descriptions to one line. Close with one or two observations (most
frequent importer, port or supplier country in the page) and offer the next page.

Do not invent values the record does not hold (prices, declared value, contact details).
Say weight is as filed on the bill and may be gross.

## No results or errors

- Zero rows: loosen the most specific filter first (drop the port, widen the keyword,
  use the HS chapter with a wildcard), re-run once, and name the filter that was too
  narrow. If a country or port spelling was guessed, check it with `us_import_values`.
- BOL not found: say so, suggest checking for a house versus master number, and offer to
  search by container number or vessel instead.
- Tool error text: fix the argument it names and retry once, then report it plainly.
