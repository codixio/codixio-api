# Codixio API

Codixio API is a REST API and a remote MCP server for three services: e-invoices (XRechnung, ZUGFeRD, Factur-X), EU tariff code suggestions and marketplace listing localization. One account and one API key cover all three services; calls are paid per call from prepaid credit.

This repository holds the public integration notes and directory metadata. It contains no source code of the service.

- Website: https://api.codixio.com/en (German: https://api.codixio.com/de)
- Documentation: https://api.codixio.com/en/docs
- llms.txt: https://api.codixio.com/llms.txt
- OpenAPI: https://api.codixio.com/openapi/v1.json

## Services

### E-invoice

Validates XRechnung, ZUGFeRD and Factur-X invoices with the official KoSIT validator and the XRechnung configuration, and returns the validation report. It also creates XRechnung (CII or UBL) and ZUGFeRD invoices from invoice data and renders invoices as readable HTML or PDF.

### Tariff classification

Suggests the EU tariff code (CN code and TARIC) for a product description, with reasoning, alternatives and applicable import measures. The reasoning is produced with a language model; the result is a suggestion, not a binding tariff information, and not legal or tax advice. Descriptions are accepted in any language, but confidence "high" is only assigned to descriptions in German.

### Listing localization

Localizes marketplace listings (title, bullet points, description and search terms) for a target marketplace and language. The listing texts are produced with a language model. The service aligns every listing with the publicly documented rules of the target marketplace, such as title length, and automatically checks the result against them before output. Each response states which rules were applied, with source and date. We do not guarantee acceptance by the marketplace; please review before publishing.

## MCP endpoints

The MCP server uses Streamable HTTP and signs in with the Codixio API account via OAuth 2.1 (authorization code with PKCE S256). Clients can register through Dynamic Client Registration or Client ID Metadata Documents, so no client ID has to be entered. Without sign-in, the endpoints answer `401` with a `WWW-Authenticate` header that points to the protected resource metadata.

| Endpoint | Tools |
| --- | --- |
| `https://api.codixio.com/mcp` | all tools: `localize_listing`, `suggest_tariff_code`, `get_job_result`, `validate_einvoice`, `create_einvoice`, `get_balance_and_quota` |
| `https://api.codixio.com/mcp/einvoice` | `validate_einvoice`, `create_einvoice`, `get_balance_and_quota` |
| `https://api.codixio.com/mcp/tariff` | `suggest_tariff_code`, `get_job_result`, `get_balance_and_quota` |
| `https://api.codixio.com/mcp/listing` | `localize_listing`, `get_job_result`, `get_balance_and_quota` |

The current tool list per endpoint is published in [llms.txt](https://api.codixio.com/llms.txt). The examples below use the endpoint with all tools; replace the URL with a service endpoint to expose only that service.

## Connect

You need a Codixio API account: https://api.codixio.com/account

### Claude

1. Open Customize > Connectors, choose "+ Add" and then "Add custom connector". On Team and Enterprise plans, an owner adds the connector under Organization settings > Connectors.
2. Enter a name and the URL `https://api.codixio.com/mcp`. Leave the OAuth client option at its default; no client ID is needed.
3. Choose "Connect", sign in with your Codixio API account and allow access.

### Claude Code

```bash
claude mcp add --transport http codixio-api https://api.codixio.com/mcp
```

Then run `/mcp` in Claude Code and sign in through the browser. Add `--scope user` to make the server available in all projects.

### Cursor

Add the server to `.cursor/mcp.json` in your project or to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "codixio-api": {
      "url": "https://api.codixio.com/mcp"
    }
  }
}
```

Cursor starts the OAuth sign-in when the server requires it.

### VS Code

Add the server to `.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "codixio-api": {
      "type": "http",
      "url": "https://api.codixio.com/mcp"
    }
  }
}
```

VS Code opens the browser for sign-in on the first connection. For the Copilot harness, VS Code also reads a workspace-root `.mcp.json` with a top-level `mcpServers` object; the entry is the same.

### ChatGPT

ChatGPT can add a custom remote MCP server with OAuth authentication. Follow the OpenAI guide [Connect from ChatGPT](https://developers.openai.com/plugins/deploy/connect-chatgpt) and use `https://api.codixio.com/mcp` as the connection URL. Availability and labels depend on the ChatGPT plan.

### Other MCP clients

Any client that supports Streamable HTTP and OAuth 2.1 can connect:

- URL: `https://api.codixio.com/mcp` (or a service endpoint above)
- Transport: Streamable HTTP
- Authentication: OAuth 2.1 with PKCE S256, discovered from the `resource_metadata` in the `WWW-Authenticate` header of the `401` response
- Client registration: Dynamic Client Registration or Client ID Metadata Documents

## REST quickstart

Create an API key in your account at https://api.codixio.com/account. Every call sends it as a bearer token; keys start with `apk_`.

```bash
# Balance and credit packages
curl https://api.codixio.com/v1/account \
  -H "Authorization: Bearer $CODIXIO_API_KEY"

# Validate an XRechnung
curl https://api.codixio.com/v1/einvoice/validate \
  -H "Authorization: Bearer $CODIXIO_API_KEY" \
  -H "Content-Type: application/xml" \
  --data-binary @invoice.xml
```

Listing localization and tariff classification work asynchronously: `POST /v1/{listing|tariff}/jobs` answers `202` with a job ID, `GET /v1/{listing|tariff}/jobs/{id}` returns status and result. The e-invoice service answers synchronously. The full interface is described in the [OpenAPI file](https://api.codixio.com/openapi/v1.json).

Each paid call reserves the current maximum price of the service from the credit and charges the actual price on success. The current maximum prices are available at `/v1/listing/max-price`, `/v1/tariff/max-price` and `/v1/einvoice/max-price`.

## Pricing

Prices: https://api.codixio.com/en/pricing

## Legal

- Terms: https://legal.codixio.com/apps/codixio-api/terms.en
- Privacy policy: https://legal.codixio.com/apps/codixio-api/privacy.en
- Data processing agreement: https://legal.codixio.com/apps/codixio-api/dpa.en
- Legal notice: https://legal.codixio.com/imprint.en

## Support

- Support page: https://api.codixio.com/en/support
- Email: support@codixio.com

## Directory listings

Listed in the official MCP Registry since 17 November 2026, version 1.0.0 each:

- `io.github.codixio/codixio-api` (all tools)
- `io.github.codixio/codixio-api-einvoice`
- `io.github.codixio/codixio-api-tariff`
- `io.github.codixio/codixio-api-listing`

Registry entry: https://registry.modelcontextprotocol.io/v0/servers/io.github.codixio%2Fcodixio-api/versions

## Deutsch

Die Codixio API bietet E-Rechnung (Prüfung mit dem offiziellen KoSIT-Validator und der XRechnung-Konfiguration, Erstellung und Darstellung), Zolltarif-Vorschläge (KN und TARIC) und Listing-Lokalisierung für Marktplätze, als REST-API und als Remote-MCP-Server unter `https://api.codixio.com/mcp`. Listing-Texte und Zolltarif-Begründungen entstehen mit einem Sprachmodell. Der Zolltarif-Vorschlag ist keine verbindliche Zolltarifauskunft und keine Rechts- oder Steuerberatung; hohe Sicherheit gibt es nur bei deutschen Beschreibungen. Deutsche Dokumentation: https://api.codixio.com/de/doku

---

Documentation © Codixio
