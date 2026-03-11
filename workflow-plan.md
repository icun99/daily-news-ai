# Daily News Agent – Workflow-Plan

## Architektur

```
[Schedule Trigger 09:00]
    → [Code Node: Topics definieren]
    → [Split in Batches: 1 Topic pro Iteration]
        → [HTTP Request: Google News RSS]
        → [Code Node: RSS parsen + filtern + formatieren]
        → [HTTP Request: Claude Haiku API]
        → [HTTP Request: Discord Webhook]
```

## Node-Details

### 1. Schedule Trigger
- Typ: `n8n-nodes-base.scheduleTrigger`
- Täglich 09:00 Uhr

### 2. Code Node: Topics definieren
- Typ: `n8n-nodes-base.code`
- Gibt Array zurück:
```javascript
return [
  {
    json: {
      name: "KI & Tech",
      keywords: "künstliche intelligenz OR AI news",
      webhook_url: "https://discord.com/api/webhooks/...",
      color: 5814783  // Blau (#58B9FF)
    }
  },
  // weitere Topics...
]
```

### 3. Split in Batches
- Typ: `n8n-nodes-base.splitInBatches`
- batchSize: 1

### 4. HTTP Request: Google News RSS
- Typ: `n8n-nodes-base.httpRequest`
- Method: GET
- URL: `https://news.google.com/rss/search?q={{ $json.keywords }}&hl=de&gl=DE&ceid=DE:de`
- Response: XML

### 5. Code Node: RSS parsen + filtern
- XML parsen
- Nur Artikel der letzten 24h
- Deduplizierung per URL
- Top 10 Artikel formatieren:
```javascript
{
  title: "...",
  url: "...",
  source: "...",
  published: "..."
}
```

### 6. HTTP Request: Claude Haiku API
- Typ: `n8n-nodes-base.httpRequest`
- Method: POST
- URL: `https://api.anthropic.com/v1/messages`
- Headers:
  - `x-api-key`: {{ $credentials.anthropicApiKey }}
  - `anthropic-version`: `2023-06-01`
  - `content-type`: `application/json`
- Body:
```json
{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 1024,
  "messages": [{
    "role": "user",
    "content": "Analysiere diese News zum Thema {{ $('Split in Batches').item.json.name }} und erstelle eine prägnante deutsche Zusammenfassung (max. 3 Sätze) plus die 3 wichtigsten Entwicklungen als Bullet Points:\n\n{{ $json.articles }}"
  }]
}
```

### 7. HTTP Request: Discord Webhook
- Typ: `n8n-nodes-base.httpRequest`
- Method: POST
- URL: `{{ $('Split in Batches').item.json.webhook_url }}`
- Body: Discord Embed
```json
{
  "embeds": [{
    "title": "📰 {{ $('Split in Batches').item.json.name }} – News vom {{ $now.format('DD.MM.YYYY') }}",
    "description": "{{ $json.content[0].text }}",
    "color": "{{ $('Split in Batches').item.json.color }}",
    "fields": [
      // Top 3 Artikel als Fields
    ],
    "footer": { "text": "Powered by Claude Haiku + Google News" }
  }]
}
```

## Kosten-Kalkulation
- Modell: claude-haiku-4-5 (~$0.00025/1K input tokens, ~$0.00125/1K output tokens)
- Pro Topic: ~500 Input + ~300 Output Tokens = ~$0.0005
- 5 Topics/Tag × 30 Tage = ~$0.075/Monat ≈ **0,07 €/Monat**

## Implementierungsreihenfolge
1. [ ] n8n health check nach MCP-Setup
2. [ ] Workflow erstellen (leerer Rahmen)
3. [ ] Schedule Trigger Node
4. [ ] Code Node: Topics
5. [ ] Split in Batches
6. [ ] HTTP Request: Google News RSS
7. [ ] Code Node: RSS parsen
8. [ ] HTTP Request: Claude Haiku
9. [ ] HTTP Request: Discord
10. [ ] Credentials: Anthropic API Key
11. [ ] Testen mit manuellem Trigger
12. [ ] Aktivieren

## Offene Punkte vom User
- [ ] Anthropic API Key hinterlegen
- [ ] Discord Webhook URLs pro Channel definieren
- [ ] Topics festlegen (welche Themen?)
