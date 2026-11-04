---
name: research-company
description: "Profile one named US importer or overseas supplier from ImportYeti's US import records: who supplies it, who it sells to, what it ships (products and HS codes), shipping lanes, ports, carriers, freight forwarders (notify parties), shipment volume and how it has changed over time. Use when the user names a company, brand, retailer, factory or supplier and asks about its imports, exports, suppliers, customers, sourcing countries, trend, growth, decline, or logistics setup. Trigger words include who supplies, where does X source, X's suppliers, X's customers, how much does X import, has X's volume changed, which forwarder or carrier does X use."
---

# Research a company or supplier

Use this skill when the user names one organization and wants its trade activity: its
suppliers, customers, products, lanes, carriers, forwarders or trend.

Not this skill: a product with no company named (find-suppliers or find-buyers); a
single bill of lading or container (shipment-records); Mexican companies (mexico-trade,
which has no profiles).

## Inputs to collect

- The organization name. Decide whether it is a US importer (retailer, brand,
  distributor) or an overseas supplier (factory, exporter). If unsure, treat it as an
  importer first.
- What the user wants: suppliers, customers, products, trend, logistics, or an overview.
- A period, only if they named one.

## Steps

1. Find it. Call `search_companies` (US importers) or `search_suppliers` (overseas
   suppliers) with `name`, `page_size` 10. Pass the name without punctuation (`HM` for
   H&M, `Johnson Johnson` for Johnson & Johnson). Website or phone can be used instead
   of `name` when the user gives one. Each hit returns `title`, `address`,
   `totalShipments`, `mostRecentShipment`, `topSuppliers` (or top customers) and `key`.
   The slug is the part of `key` after `company/` or `supplier/`. Never guess a slug.
   - Several hits share the name: show the top three with addresses and ask which one,
     unless one clearly dominates by shipments.
   - No hit: retry once with the legal name (e.g. "Target Corporation" for Target) or
     a shorter form. Then try the other search tool. Then say it is not in the data.
2. Stop if the hit answers the question (total shipments, most recent shipment, or the
   top partners shown in the hit).
3. Otherwise call `get_company` or `get_supplier` with `id` set to the slug and `fields`
   limited to what the question needs:
   - Overview: `["title", "address", "total_shipments", "date_range", "hs_codes"]`
   - Suppliers of an importer: `["title", "total_shipments", "suppliers_table"]`
   - Customers of a supplier: `["title", "total_shipments", "companies_table"]`
   - Products: `["title", "hs_codes"]`
   - Trend over time: `["title", "date_range", "time_series", "avg_teu_per_month"]`
   - Lanes and ports: `["title", "lane_permutations"]`
   - Carriers: `["title", "carriers_per_country", "container_types"]`
   - Forwarders: `["title", "notify_party_table", "notify_party_shipments_perc"]`
     (importers only)
   - Latest shipments: `["title", "recent_bols"]`
   Always pass `fields`. Profiles of large organizations are very long without it.
   If the tool says the organization "moved to" another path, retry once with the slug
   from that path.
4. A filtered shipment list instead (one product, port, supplier, carrier or period for
   this company): call `power_query_us_import` with `company: <slug>` (or
   `supplier: <slug>`) plus the filter, `page_size` 10. Do this instead of step 3, never
   in addition to it for the same question.
5. Trend questions: compare the last 12 months of `time_series` with the 12 months
   before, state both counts and the percent change, and name the months with the
   biggest moves. Do not call power queries by month to rebuild the series.

## Credits

Search costs 1 search credit per 10 results and no data credits. A profile costs 1 data
credit. A power query page costs 1 data credit per 10 rows. A normal question is 1
search credit plus 1 data credit. Keep `page_size` at 10, and ask before fetching more
pages or more than 3 profiles. Mention the balance only when `creditsRemaining` is under
10 or the user asks. On "Not enough credits", stop and point the user to
https://www.importyeti.com.

## Present the result

Open with the organization as a link (https://www.importyeti.com/company/<slug> or
https://www.importyeti.com/supplier/<slug>), its location, total shipments and the date
range covered.

Then the requested section as a table: suppliers or customers (name as link where a
slug is given, country, shipments, share), products (HS code, description, shipments),
lanes (origin port, destination port, shipments), carriers, or forwarders.

Close with two or three sentences of interpretation: concentration (how much depends on
the top supplier), country mix, direction of the trend, or what stands out. Say figures
are ocean shipments and that related-party shipments may appear as "internal".

Do not invent contact details, prices, MOQs or certifications. Use the website and phone
fields only if the profile returned them.

## No results or errors

- Name not found after the retries in step 1: say the organization has no US ocean
  import records under that name and ask for a legal name, website or a BOL number.
- Section empty (e.g. no `notify_party_table`): say the data does not record it for
  this organization rather than guessing.
- Tool error text: fix the argument it names and retry once, then report it plainly.
