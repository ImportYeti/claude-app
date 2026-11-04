---
name: find-suppliers
description: "Find overseas manufacturers, factories, vendors or suppliers of a product using US import shipment records from ImportYeti. Use when the user wants to source a product, find who makes or ships it, find factories in a specific country (or outside one), compare suppliers, see which US brands a factory ships to, or vet suppliers by real shipment history. Trigger words include manufacturer, factory, supplier, vendor, source, sourcing, who makes, who produces, alternative supplier."
---

# Find suppliers

Use this skill when the user wants manufacturers, factories, vendors or suppliers for a
product. It answers from ImportYeti's US import bills of lading, so every supplier shown has
actually shipped that product to a US buyer. It does not cover suppliers who only sell
domestically or only ship to other countries.

Not this skill: "who imports X" or "find buyers" (find-buyers); one named supplier's
profile, customers or trend (research-company); suppliers to Mexico (mexico-trade);
individual shipment records (shipment-records).

## Inputs to collect

- Product. Use the user's words as the keyword, e.g. "yoga mats". Do not add synonyms yet.
- Country preference, if any. Comma-separated, lowercase, e.g. "vietnam,india".
- Countries to avoid, if any, e.g. "china".
- Specialists or largest shippers. Specialists are suppliers whose shipments are mostly this
  product; largest are ranked by shipped weight. Default to specialists; that is what
  "find manufacturers" usually means.

If the product is ambiguous (e.g. "pumps", "cases"), ask one clarifying question before
spending credits.

## Steps

1. Specialists (default): call `top_suppliers_by_product` with `product`, optional
   `countries` and `exclude_countries`, and `page_size` 10. Each row gives
   `supplier_name`, `supplier_link` (the slug), `supplier_country`, `matching_shipments`,
   `specialization` (percent of the supplier's shipments matching the keyword),
   `supplier_experience`, `supplier_total_shipments`, `total_customers` and up to five
   `customer_companies`.
2. Largest by volume instead: call `top_overseas_suppliers` with `product_description` set
   to the keyword (boolean syntax allowed, e.g. `"yoga mat" OR "exercise mat"`), optional
   `supplier_country`, optional `hs_code`, and `page_size` 10. `sort_by` defaults to
   weight; pass `"teu"` to rank by containers. Before filtering by country, confirm the
   spelling with the free `us_import_values` (`list: "countries"`, `search: "korea"`).
   Omit `start_date` and `end_date` unless the user named a period; for "recent" or
   "this year" set `start_date` to 12 months ago in MM/DD/YYYY.
3. Stop here unless the user asks about specific suppliers. For up to three names, call
   `get_supplier` with `id` set to the slug and
   `fields: ["title", "address_country", "total_shipments", "date_range", "companies_table", "hs_codes"]`.
   Add `"time_series"` only if they ask whether the supplier is growing. Ask before
   profiling more than three. Never guess a slug: use `supplier_link` from step 1 or the
   `key` from `search_suppliers` (the part after `supplier/`).
4. Never run `power_query_us_import` for this question. Never call both step 1 and step 2
   for the same request unless the user asks for both views.

## Credits

Step 1 or 2 costs 1 data credit per 10 rows. Each profile in step 3 costs 1 credit. Keep
`page_size` at 10 and ask before fetching a second page. Mention the balance only when
`creditsRemaining` in the response is under 10 or the user asks. If a call returns
"Not enough credits", stop and tell the user to add credits at https://www.importyeti.com.

## Present the result

Open with one line: product, country filter, and the time window (the data defaults to
01/01/2021 to the end of last month).

Then a table, one row per supplier:
Supplier (linked to https://www.importyeti.com/supplier/<slug>) | Country | Matching
shipments | Specialization % | Total shipments | Example US customers.

Close with two or three sentences: which suppliers look strongest for this user and why
(high specialization with a solid total count, a known customer in the user's segment, or
the preferred country). Say that specialization is the share of the supplier's US
shipments matching the keyword and that customers are real importers of record.

Do not invent contact details, prices, MOQs or certifications; the data does not include
them. If the user needs them, point to the supplier's ImportYeti page and its website
field, if the profile has one.

## No results or errors

- Zero rows: retry once with a broader or alternative keyword (e.g. "mat" for "yoga mat"),
  or with `top_overseas_suppliers` and an `hs_code` if the user knows one. If still empty,
  say no US import shipments match and suggest an HS code or a nearby product term.
- Only one or two rows: show them and offer to drop the country filter.
- Error text from a tool: read it, fix the argument it names (slug, spelling, date) and
  retry once. Then report the error plainly.
