# Example Workflows

Three end-to-end patterns to adapt. All import cleanly into n8n once
`n8n-nodes-scraperapi-official` is installed and a credential named
"ScraperAPI API" exists.

## 1. Daily Amazon price monitor → email if dropped

Schedule daily, scrape one ASIN, compare to last run, email on drop.
Strip-down of the public price-drop digest template — useful when the
user wants a working single-product version they can extend.

```json
{
  "name": "Daily Amazon price monitor",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            { "field": "hours", "hoursInterval": 24, "triggerAtHour": 9 }
          ]
        }
      },
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [240, 300],
      "id": "schedule-1",
      "name": "Daily 9am"
    },
    {
      "parameters": {
        "resource": "sde",
        "sdePlatform": "amazon",
        "operation": "amazonProduct",
        "sdeAsin": "B08N5WRWNW",
        "sdeAmazonProductOptions": {
          "tld": "com",
          "countryCode": "us",
          "outputFormat": "json"
        }
      },
      "type": "n8n-nodes-scraperapi-official.scraperApi",
      "typeVersion": 1,
      "position": [480, 300],
      "id": "scrape-1",
      "name": "ScraperAPI - Amazon",
      "credentials": {
        "scraperApi-Api": { "id": "REPLACE", "name": "ScraperAPI API" }
      }
    },
    {
      "parameters": {
        "language": "javaScript",
        "jsCode": "const parsed = JSON.parse($input.first().json.response.body);\nconst previous = $getWorkflowStaticData('global').lastPrice;\nconst current = parsed.pricing;\n$getWorkflowStaticData('global').lastPrice = current;\nreturn [{ json: { name: parsed.name, current, previous, dropped: previous != null && current < previous } }];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [720, 300],
      "id": "code-1",
      "name": "Compare to last run"
    },
    {
      "parameters": {
        "conditions": {
          "options": { "typeValidation": "strict" },
          "conditions": [
            {
              "leftValue": "={{ $json.dropped }}",
              "rightValue": true,
              "operator": { "type": "boolean", "operation": "equals" }
            }
          ],
          "combinator": "and"
        }
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [960, 300],
      "id": "if-1",
      "name": "Price dropped?"
    },
    {
      "parameters": {
        "sendTo": "you@example.com",
        "subject": "=Price drop: {{ $json.name }}",
        "message": "=From {{ $json.previous }} to {{ $json.current }}"
      },
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.1,
      "position": [1200, 240],
      "id": "gmail-1",
      "name": "Send email",
      "credentials": {
        "gmailOAuth2": { "id": "REPLACE", "name": "Gmail" }
      }
    }
  ],
  "connections": {
    "Daily 9am": {
      "main": [[{ "node": "ScraperAPI - Amazon", "type": "main", "index": 0 }]]
    },
    "ScraperAPI - Amazon": {
      "main": [[{ "node": "Compare to last run", "type": "main", "index": 0 }]]
    },
    "Compare to last run": {
      "main": [[{ "node": "Price dropped?", "type": "main", "index": 0 }]]
    },
    "Price dropped?": {
      "main": [
        [{ "node": "Send email", "type": "main", "index": 0 }],
        []
      ]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

For the **multi-product, multi-retailer** version (Amazon + Walmart +
Google Shopping, with history table and HTML digest), point the user at
the public template:
`https://n8n.io/workflows/15609-send-daily-price-drop-digest-emails-for-amazon-walmart-and-google-via-scraperapi/`

## 2. Webhook-triggered company enrichment

POST a company name to a webhook, get back a Google search summary.
Common building block for lead enrichment or as a tool for an AI agent.

```json
{
  "name": "Enrich company via Google",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "enrich",
        "responseMode": "responseNode"
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [240, 300],
      "id": "webhook-1",
      "name": "Webhook"
    },
    {
      "parameters": {
        "resource": "sde",
        "sdePlatform": "google",
        "operation": "googleSearch",
        "sdeQuery": "={{ $json.body.company }}",
        "sdeGoogleSearchOptions": {
          "tld": "com",
          "countryCode": "us",
          "outputFormat": "json"
        }
      },
      "type": "n8n-nodes-scraperapi-official.scraperApi",
      "typeVersion": 1,
      "position": [480, 300],
      "id": "scrape-1",
      "name": "ScraperAPI - Google",
      "credentials": {
        "scraperApi-Api": { "id": "REPLACE", "name": "ScraperAPI API" }
      }
    },
    {
      "parameters": {
        "language": "javaScript",
        "jsCode": "const parsed = JSON.parse($json.response.body);\nconst top = (parsed.organic_results || []).slice(0, 5).map(r => ({\n  title: r.title,\n  url: r.link,\n  snippet: r.snippet,\n}));\nreturn [{ json: { company: $('Webhook').first().json.body.company, top_results: top } }];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [720, 300],
      "id": "code-1",
      "name": "Shape result"
    },
    {
      "parameters": { "options": {} },
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [960, 300],
      "id": "respond-1",
      "name": "Respond"
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{ "node": "ScraperAPI - Google", "type": "main", "index": 0 }]]
    },
    "ScraperAPI - Google": {
      "main": [[{ "node": "Shape result", "type": "main", "index": 0 }]]
    },
    "Shape result": {
      "main": [[{ "node": "Respond", "type": "main", "index": 0 }]]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

Test by POSTing to the webhook URL:

```bash
curl -X POST https://your-n8n/webhook/enrich \
  -H 'content-type: application/json' \
  -d '{"company":"Anthropic"}'
```

## 3. AI Parser — create a reusable parser, then extract

Build a parser once from example product URLs, wait for it to finish
generating, then extract structured fields from a new URL. The `Wait` +
`Get` + `IF` loop handles the asynchronous generation (status goes
`GENERATING` → `FINISHED`). Adapt the poll interval to how long generation
takes for your pages.

```json
{
  "name": "AI Parser - build and extract",
  "nodes": [
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [240, 300],
      "id": "manual-1",
      "name": "Start"
    },
    {
      "parameters": {
        "resource": "aiparser",
        "operation": "aiParserCreate",
        "aiParserName": "product-parser",
        "aiParserUrls": ["https://example.com/p/1", "https://example.com/p/2"],
        "aiParserCreateFields": {
          "field": [
            { "name": "title", "description": "Product title", "type": "string" },
            { "name": "price", "description": "Current price in USD", "type": "number" }
          ]
        },
        "aiParserScraperParams": { "render": true, "country_code": "us" }
      },
      "type": "n8n-nodes-scraperapi-official.scraperApi",
      "typeVersion": 1,
      "position": [480, 300],
      "id": "create-1",
      "name": "Create parser",
      "credentials": {
        "scraperApi-Api": { "id": "REPLACE", "name": "ScraperAPI API" }
      }
    },
    {
      "parameters": { "amount": 30 },
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [720, 300],
      "id": "wait-1",
      "name": "Wait 30s"
    },
    {
      "parameters": {
        "resource": "aiparser",
        "operation": "aiParserGet",
        "aiParserId": "={{ $('Create parser').first().json.response.body.id }}",
        "aiParserVersion": -1
      },
      "type": "n8n-nodes-scraperapi-official.scraperApi",
      "typeVersion": 1,
      "position": [960, 300],
      "id": "get-1",
      "name": "Get status",
      "credentials": {
        "scraperApi-Api": { "id": "REPLACE", "name": "ScraperAPI API" }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": { "typeValidation": "strict" },
          "conditions": [
            {
              "leftValue": "={{ $json.response.body.status }}",
              "rightValue": "FINISHED",
              "operator": { "type": "string", "operation": "equals" }
            }
          ],
          "combinator": "and"
        }
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [1200, 300],
      "id": "if-1",
      "name": "Finished?"
    },
    {
      "parameters": {
        "resource": "aiparser",
        "operation": "aiParserParse",
        "aiParserId": "={{ $('Create parser').first().json.response.body.id }}",
        "aiParserVersion": -1,
        "aiParserUrl": "https://example.com/p/99",
        "aiParserScraperParams": { "render": true }
      },
      "type": "n8n-nodes-scraperapi-official.scraperApi",
      "typeVersion": 1,
      "position": [1440, 240],
      "id": "parse-1",
      "name": "Parse a URL",
      "credentials": {
        "scraperApi-Api": { "id": "REPLACE", "name": "ScraperAPI API" }
      }
    }
  ],
  "connections": {
    "Start": {
      "main": [[{ "node": "Create parser", "type": "main", "index": 0 }]]
    },
    "Create parser": {
      "main": [[{ "node": "Wait 30s", "type": "main", "index": 0 }]]
    },
    "Wait 30s": {
      "main": [[{ "node": "Get status", "type": "main", "index": 0 }]]
    },
    "Get status": {
      "main": [[{ "node": "Finished?", "type": "main", "index": 0 }]]
    },
    "Finished?": {
      "main": [
        [{ "node": "Parse a URL", "type": "main", "index": 0 }],
        [{ "node": "Wait 30s", "type": "main", "index": 0 }]
      ]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

The `Finished?` false-branch loops back to `Wait 30s`, re-polling until the
parser reports `FINISHED`. Extracted data lands under
`response.body.result` on the `Parse a URL` node. Reuse the same parser id
across many pages: the first parser per account is free but each additional
one costs **30,000 credits** to generate, whereas each `aiParserParse` call is
only the standard scraping cost (1 credit, more if you enable render/premium).
So generate once and parse many — never create parsers in a loop.

## Notes when adapting

- Both examples reference `"id": "REPLACE"` in credential blocks. The
  user re-selects the credential once after import — n8n cannot resolve
  credential IDs across instances.
- Static data (`$getWorkflowStaticData('global')`) persists between
  executions within the same workflow. It's the fastest way to track
  "last seen" values without an external store, but it's tied to the
  workflow — exporting/importing resets it.
- For the enrichment example, the field name `organic_results` comes
  from the Google Search SDE response. Other verticals use different
  field names — log one execution and inspect the JSON before writing
  Code-node logic.
- Gmail credentials require an OAuth flow; if the user wants SMTP
  instead, swap `n8n-nodes-base.gmail` for `n8n-nodes-base.emailSend`
  with an SMTP credential.
