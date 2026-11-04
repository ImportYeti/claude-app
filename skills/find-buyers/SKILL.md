---
name: find-buyers
description: "Find companies that import or buy a product: sales leads, prospects, the top US importers of a product or HS code, Mexican importers, or the largest US exporters, using ImportYeti customs records. Use when the user sells something and wants buyers, wants to know who imports a product into the US or Mexico, wants a prospect list, or asks which companies, brands or distributors bring in a product. Trigger words include buyers, importers, leads, prospects, who imports, who buys, top importers, customers for, distributors of, exporters of."
---

# Find buyers

Use this skill when the user wants the companies that import a product: sales leads,
competitor lists, market share by importer, or the largest US exporters of a product.

Not this skill: who makes or supplies a product (find-suppliers); one named company's
own suppliers or customers (research-company); individual shipments (shipment-records);
Mexican exporters, brokers or declarations (mexico-trade, except Mexican importers, which
this skill covers in step 3).

## Inputs to collect

- Product keyword or HS code. Keep the user's wording. Boolean syntax is allowed in
  `product_description`, e.g. `"cotton towel" OR "bath towel"`, and wildcards in
  `hs_code`, e.g. `6302*`.
- Market: US importers (default), Mexican importers, or US exporters.
- Specialists or largest. Specialists are companies whose shipments are mostly this product
  (good for niche products and distributors). Largest are ranked by shipped weight (good
  for "top importers" and market size). Default: "top" or "biggest" means largest; "who
  imports" or "which brands" means specialists.
- Optional narrowing: supplier country, US entry port, period.

## Steps

1. US specialists: call `top_companies_by_product` with `product`, optional `countries`
   and `exclude_countries` (supplier countries, comma-separated, lowercase), `page_size`
   10. Rows give company name, slug, matching shipments, specialization %, total
   shipments and top suppliers.
2. US largest by volume: call `top_us_importers` with `product_description` or `hs_code`,
   optional `supplier_country`, `entry_port`, `sort_by` (`"weight"` default or `"teu"`),
   `page_size` 10. Before filtering on a country, port or carrier, confirm the spelling
   with the free `us_import_values` (`list` is `countries`, `entry_ports`,
   `shipping_ports` or `carriers`; pass `search`). Omit dates unless the user named a
   period; "recent" or "past 12 months" means `start_date` 12 months ago in MM/DD/YYYY.
3. Mexican importers: call `top_mx_import_companies` with `product_description` (Spanish
   terms work best, e.g. `Aluminio OR acero`) or `hs_code`, optional `supplier_country`
   in Spanish (e.g. `Estados Unidos De America`), `page_size` 10. See the mexico-trade
   skill for spellings.
4. US exporters: call `top_us_export_companies` with `product_description` or `hs_code`,
   optional `destination_country`, `page_size` 10.
5. One named lead the user wants to dig into: `search_companies` with `name` (punctuation
   removed, e.g. `HM` for H&M), then `get_company` with the `key` slug and only the
   needed `fields` (e.g. `["title", "address", "total_shipments", "suppliers_table", "hs_codes"]`).
   Ask before profiling more than three. Never guess a slug.

Use one ranking tool per question. Do not page `power_query_us_import` and count rows to
build a ranking.

## Credits

Each ranking page costs 1 data credit per 10 rows; each profile costs 1 credit. Keep
`page_size` at 10 and ask before fetching page 2 or going past 20 leads. Mention the
balance only when `creditsRemaining` is under 10 or the user asks. On "Not enough
credits", stop and point the user to https://www.importyeti.com.

## Present the result

Open with one line: product, market, filters and time window.

Then a ranked table: Company (linked to https://www.importyeti.com/company/<slug>) |
Location | Matching shipments | Total shipments | Top suppliers or supplier countries.
For Mexican importers link is not available; show name, state and RFC if returned.

Close with two or three sentences: the strongest leads for this user and one suggested
narrowing filter (country, port, period) if the list is generic. Say the figures count
ocean shipments, not revenue. Do not invent contact names, emails, phone numbers or
buying intent. If the user asks for contacts, point to the company's ImportYeti page.

## No results or errors

- Zero rows: loosen one filter (drop the country, widen the keyword, try the HS chapter
  with a wildcard) and retry once. Then say no shipments matched and suggest an HS code.
- Results dominated by freight forwarders or NVOCCs (names containing Logistics,
  Forwarding, Shipping): say so and offer the specialists view, which filters them better.
- Tool error text: fix the argument it names and retry once, then report it plainly.
