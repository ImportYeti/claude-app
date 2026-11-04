---
name: importyeti-get-started
description: "Explain what the ImportYeti plugin does and how to use it. Use when the user asks what ImportYeti can do, what data it has, how credits work, what it costs, whether they need an account, how to get started, or wants example questions. Covers US import and export bills of lading and Mexico customs declarations (pedimentos), finding suppliers and buyers, researching a company, and looking up shipment records."
---

# Get started with ImportYeti

Use this skill when the user asks what ImportYeti does, what data it covers, how credits work,
whether they need an account, or what to ask first.

Not this skill: any actual data question. Hand those to the matching skill below.

## What ImportYeti covers

ImportYeti answers from customs records, not from company self-reporting:

- US import bills of lading: every ocean shipment into the US, with importer, overseas
  supplier, product description, HS code, ports, vessel, carrier, containers and weight.
- US export bills of lading: ocean shipments out of the US, with exporter, destination
  country and product.
- Mexico import and export customs declarations (pedimentos): Mexican importers and
  exporters, their foreign partners, customs brokers, aduanas, incoterms and RFC numbers.

Default time window: 01/01/2021 to the end of last month. Records are ocean freight; air
and truck shipments into the US are not included. Contact details, prices, minimum order
quantities and certifications are not in the data.

## Jobs ImportYeti does well

| Job | Skill | Example |
|---|---|---|
| Find manufacturers or suppliers of a product | find-suppliers | "Find factories that make yoga mats in Vietnam" |
| Find buyers, importers, sales leads | find-buyers | "Who are the top US importers of cotton towels?" |
| Research one company or supplier | research-company | "Who supplies Walmart, and is its volume growing?" |
| Look up shipment records | shipment-records | "What did container DFSU4370223 carry?" |
| Mexico trade | mexico-trade | "Which brokers clear electronics at Manzanillo?" |

## Account and credits

ImportYeti needs an ImportYeti account with API credits. The user logs in when they first use
it. Every query spends the user's own credits:

- Searching for a company or supplier by name: 1 search credit per 10 results, no data
  credits.
- One company or supplier profile: 1 data credit.
- One bill of lading or declaration by its number: 0.1 data credit.
- Shipment lists and rankings (power queries, top importers, top suppliers, top brokers):
  1 data credit per 10 rows. The default page is 10 rows, so one page is 1 credit.
- Checking allowed spellings for a filter (countries, ports, carriers): free.

Every tool response reports requestCost and creditsRemaining. Mention the balance only
when creditsRemaining is under 10 or the user asks. If a tool answers "Not enough
credits", stop and tell the user to add credits at https://www.importyeti.com.

## Steps

1. Answer from this file. Do not call paid tools to explain ImportYeti.
2. If the user wants to confirm the connection works, call `us_import_values` with
   `list: "countries"` and `page_size: 3`. It is free and shows the top supplier countries
   by shipment count.
3. Offer three example prompts tailored to what the user said they do:
   - Sourcing: "Find suppliers of stainless steel water bottles outside China."
   - Sales: "Which US companies import the most commercial kitchen equipment?"
   - Supply chain or research: "Who supplies Home Depot and how has its volume changed
     since 2023?"
   - Logistics: "Which carriers and forwarders does Target use on the Yantian to Los
     Angeles lane?"
   - Mexico: "Who are the largest Mexican importers of aluminum?"

## Working rules the other skills follow

- Use the cheapest tool that answers the question, then stop.
- Never guess a company or supplier slug. Slugs come from `key` in search results or
  `supplier_link` in product rankings.
- Profiles are requested with `fields` so only the needed sections come back.
- Pages stay at 10 rows. The user is asked before more pages or more than 3 profiles.
- Dates are omitted unless the user named a period. "Recent" means the last 12 months.
- Companies and suppliers are shown as links to https://www.importyeti.com/company/<slug>
  or https://www.importyeti.com/supplier/<slug>, never as raw slugs.
- Contact details, prices, MOQs and certifications are never invented.

## If the user is not logged in or has no credits

Say that ImportYeti needs an ImportYeti account with API credits, that the login prompt
appears on the first data question, and that credits can be added at
https://www.importyeti.com. Do not retry a failed call more than once.
