# Sentinelle catnat: French natural-disaster decrees MCP server

**MCP server and pay-per-call API for AI agents.** For any French commune, it returns the natural-disaster decrees (*arrêtés de catastrophe naturelle*, "CatNat") published in the Journal officiel, and tells whether the 30-day window to file a home-insurance claim is still open, with the exact deadline and the number of days left.

- **MCP endpoint (streamable HTTP):** `https://catnat.amacunia.fr/mcp`
- **REST endpoint:** `GET https://catnat.amacunia.fr/v1/catnat?communes=75056,13055`
- **Free sample:** https://catnat.amacunia.fr/v1/exemple
- **Docs for agents:** https://catnat.amacunia.fr/llms.txt
- **Price:** 0.01 USDC per call (1 to 50 communes per call), paid with the [x402](https://x402.org) protocol on Base. No account, no API key, no subscription.
- **Official MCP Registry:** `io.github.Marcoco27/sentinelle-catnat`

## Why it exists

In France, damage from floods, mudslides, drought subsidence (*sécheresse*, clay shrink-swell) and other natural perils is covered by home insurance only once the State publishes a CatNat decree in the Journal officiel. From that day, the policyholder has **30 days** to file a claim (art. L125-2 Code des assurances). Miss it and the insurer may refuse.

The decree lists hundreds of communes at once, and nobody tells the residents. Brokers, loss adjusters, property managers and notaries have to watch the Journal officiel themselves. This server does the watching and the date arithmetic: one call, up to 50 communes, every decree, every open deadline.

## Tools

| Tool | Price | What it does |
|---|---|---|
| `catnat_lookup` | 0.01 USDC (x402) | Decrees and open claim deadlines for 1 to 50 INSEE commune codes |
| `catnat_example` | free | A real answer (Paris, Seine flood of 2018, computed as of 2018-03-01) to see the output shape |

**Input of `catnat_lookup`:** `communes`, a string of 1 to 50 INSEE codes separated by commas. INSEE codes are not postal codes: Paris is `75056` (its postal codes run from 75001 to 75020), Marseille is `13055`. Corsica uses `2A` and `2B`.

## Example output

The Seine flood of January-February 2018. The decree was published on 15 February 2018; the 30th day fell on a Saturday, so the deadline moves to Monday 19 March.

```json
{
  "date_calcul": "2018-03-01",
  "communes_avec_delai_ouvert": 1,
  "resultats": [{
    "code_insee": "75056",
    "commune": "Paris",
    "delais_ouverts": [{
      "arrete": "INTE1804348A",
      "risque": "Inondations et/ou Coulées de Boue",
      "evenement_debut": "2018-01-15",
      "evenement_fin": "2018-02-05",
      "publication_jo": "2018-02-15",
      "date_limite_declaration": "2018-03-19",
      "jours_restants": 18,
      "delai_ouvert": true
    }],
    "nombre_arretes_total": 18
  }],
  "marche_a_suivre": "Déclarer le sinistre à l'assureur par lettre recommandée…",
  "source": "Géorisques, base GASPAR (Licence Ouverte 2.0) ; délai : art. L125-2 du code des assurances",
  "avertissement": "Information, pas un conseil juridique. Vérifier l'arrêté au Journal officiel."
}
```

Field names are in French. `delai_ouvert` is true while a claim can still be filed; `jours_restants` is 0 once the window is closed; `marche_a_suivre` (how to file the claim, in French) appears only when a window is open.

## Connect

Any MCP client can connect and call `catnat_example`. Calling `catnat_lookup` returns an x402 *payment required* answer: the client must support x402 payments to pay it (see below).

**Claude Code**

```bash
claude mcp add --transport http sentinelle-catnat https://catnat.amacunia.fr/mcp
```

**Claude Desktop / claude.ai:** Settings, Connectors, *Add custom connector*, URL `https://catnat.amacunia.fr/mcp`.

**Cursor** (`~/.cursor/mcp.json`)

```json
{ "mcpServers": { "sentinelle-catnat": { "url": "https://catnat.amacunia.fr/mcp" } } }
```

**VS Code** (`.vscode/mcp.json`)

```json
{ "servers": { "sentinelle-catnat": { "type": "http", "url": "https://catnat.amacunia.fr/mcp" } } }
```

## Paying with x402

The paid tool follows the x402 MCP convention: the first call returns the payment requirements (scheme `exact`, network Base `eip155:8453`, asset USDC, amount `10000` = 0.01 USDC); the client signs a payment, puts it in the request `_meta` under `x402/payment`, and calls again. The payment is settled only if the tool succeeds: invalid input is never charged.

A minimal Python client with the official `x402` SDK is in [`examples/pay_mcp.py`](examples/pay_mcp.py). The same service is also sold over plain HTTP (`GET /v1/catnat`, HTTP 402 then retry), which every x402 HTTP client supports.

## Data and limits

- Source: GASPAR database of the French Ministry of Ecological Transition, read live through the Géorisques API (BRGM), under the Licence Ouverte 2.0 (Etalab). Data since 1982.
- Deadline rule: 30 days after publication in the Journal officiel, publication day excluded, extended to the next working day when it falls on a weekend or a public holiday (art. 641 and 642 Code de procédure civile). When readings differ by one day, the earliest date is returned.
- Mainland public holidays only: Alsace-Moselle extra holidays are not applied.
- Information only, not legal advice. The decree published in the Journal officiel prevails.

## Also listed on

- [data.gouv.fr reuse](https://www.data.gouv.fr/reuses/sentinelle-catnat-arretes-de-catastrophe-naturelle-et-delai-de-declaration-de-sinistre-pour-agents-ia)
- Coinbase x402 Bazaar
- Official MCP Registry: `io.github.Marcoco27/sentinelle-catnat`

## About this repository

This repository holds the public documentation, the MCP Registry manifest ([`server.json`](server.json)) and client examples. The service itself runs at `catnat.amacunia.fr`. Issues and questions are welcome.

---

### En français

Serveur MCP et API payable à l'appel pour les agents IA : pour une commune française, les arrêtés de catastrophe naturelle parus au Journal officiel et le délai de trente jours pour déclarer un sinistre à son assureur, avec la date limite exacte et les jours restants. Un centime de dollar par appel (protocole x402, USDC sur Base), jusqu'à 50 communes. Présentation : https://catnat.amacunia.fr
