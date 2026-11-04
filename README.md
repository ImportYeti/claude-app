# ImportYeti

![ImportYeti logo](assets/logo.png)

See who's importing what, who their suppliers are, and how their shipping activity has changed, using ImportYeti's US import and export bills of lading and Mexico customs declarations (pedimentos). Look up any company or overseas supplier, rank the top importers or suppliers for a product, and pull individual shipment records, all without leaving your conversation with Claude.

Useful for finding new suppliers, researching competitors' sourcing, spotting sales leads among importers, and tracking shifts in trade activity. Built for sourcing teams, supply chain professionals, freight and logistics companies, and market researchers.

## What's included

- **ImportYeti connector**: connects Claude to ImportYeti's MCP server at `https://data.importyeti.com/mcp`. All tools are read-only.
- **Skills** that teach Claude how to answer common trade questions accurately and with as few credits as possible:
  - `find-suppliers`: find manufacturers and factories for a product
  - `find-buyers`: find importers, buyers and sales leads for a product
  - `research-company`: profile one importer or supplier (suppliers, customers, products, lanes, trend)
  - `shipment-records`: look up individual bills of lading
  - `mexico-trade`: Mexican importers, exporters, customs brokers and declarations
  - `importyeti-get-started`: what ImportYeti covers, how credits work, and example questions

## Requirements

An ImportYeti account with API credits. The first time Claude uses ImportYeti, you sign in with your ImportYeti account. Every query uses your own account's credits.

## Example questions

- "Which US companies import the most solar panels?"
- "Who are Patagonia's top overseas suppliers, and which countries do they ship from?"
- "Find factories outside China that make yoga mats."
- "Who are the top importers of auto parts into Mexico, and which customs brokers handle them?"

## Data and privacy

This plugin sends your trade questions, as tool arguments (company names, products, HS codes, ports, dates and similar filters), only to ImportYeti's server at `data.importyeti.com`. It does not run local code, read your files, or contact any other service.

- Privacy policy: https://www.importyeti.com/policies/privacy
- Terms: https://www.importyeti.com/policies/terms
- Support: https://www.importyeti.com/contact
