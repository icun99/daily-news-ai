# Daily News Agent – Projekt-Kontext

## Ziel
Täglicher automatischer News-Digest per Discord. Ein n8n-Workflow fetcht morgens Google News RSS, lässt Claude Haiku eine KI-Zusammenfassung erstellen und postet das Ergebnis als Discord Embed.

## Infrastruktur
- **n8n Server:** https://n8n-ai-u47147.vm.elestio.app (Elestio, gehostet)
- **MCP-Server:** n8n-mcp (global konfiguriert in ~/.claude/settings.json)
- **KI-Modell:** claude-haiku-4-5-20251001 (~0,07 €/Monat für 3 Topics)

## Deployed Workflow
- **ID:** `o5arELTCG7jbyFxC`
- **Name:** Daily News Digest
- **Status:** Deployt, inaktiv (Credentials fehlen noch)
- **Topics:** KI & Tech, Linux & Open Source, Gaming
- **Trigger:** Täglich 09:00 Uhr

## Workflow-Nodes (Reihenfolge)
1. `Schedule Trigger` – 09:00 täglich
2. `Topics definieren` – Code Node, gibt 3 Topics zurück (Name, Keywords, Webhook-URL, Farbe)
3. `Split in Batches` – batchSize 1, Loop zurück von Discord Webhook
4. `Google News RSS` – HTTP GET, responseFormat: text
5. `RSS parsen` – Code Node, Regex-Parsing, 24h-Filter, Top 10, Dedup per URL
6. `Claude Haiku API` – HTTP POST zu `api.anthropic.com/v1/messages`
7. `Discord Webhook` – HTTP POST, Discord Embed mit Summary + Top-3-Artikeln

## Noch ausstehend (User-Aktion)
1. Node "Claude Haiku API" → Header `x-api-key` = echter Anthropic API Key
2. Node "Topics definieren" → `DISCORD_WEBHOOK_KI_TECH`, `DISCORD_WEBHOOK_LINUX`, `DISCORD_WEBHOOK_GAMING` ersetzen
3. Workflow aktivieren (Toggle in n8n UI)

## Git Repository
- **GitHub:** `icun99/daily-news-ai` (privat)
- **Remote:** `gh-privat:icun99/daily-news-ai.git` (SSH-Alias in `~/.ssh/config`)
- **Branch:** `main`

## Lokale Dateien
- `workflow.json` – Backup/Referenz des deployten Workflows (enthält Workflow-ID)
- `workflow-plan.md` – Ursprüngliche Architektur-Planung

## n8n API-Nutzung
- MCP-Tools waren in der Session nicht verfügbar → direkter Zugriff via REST API (`curl`)
- API Key in `~/.claude/settings.json` unter `mcpServers.n8n.env.N8N_API_KEY`
- Wichtig: `active`-Feld ist bei POST `/api/v1/workflows` read-only (nicht mitsenden)

## MCP-Tools (konfiguriert, aber ggf. nicht in jeder Session aktiv)
- `search_nodes`, `get_node`, `validate_node`
- `n8n_create_workflow`, `n8n_update_partial_workflow`
- `n8n_validate_workflow`, `n8n_deploy_template`
- `n8n_health_check`
