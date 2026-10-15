# Daily News Agent – Projekt-Kontext

## Ziel
Wöchentlicher automatischer News-Digest per Discord. Ein n8n-Workflow fetcht freitags die News der Woche via Perplexity AI (Sonar), erstellt einen deutschen KI-Digest und postet das Ergebnis als Discord Embed.

## Infrastruktur
- **n8n Server:** https://n.dennro.de (Selfhostet, gehostet)
- **MCP-Server:** n8n-mcp, nur bei Bedarf per `claude mcp add n8n -s local -- ~/.local/bin/n8n-mcp-start dennro`
- **News-API:** Perplexity Sonar (~$0.55/Monat für 3 Topics)

## Deployed Workflow
- **ID:** `o5arELTCG7jbyFxC`
- **Name:** Weekly News Digest
- **Status:** Deployed, aktiv
- **Topics:** KI & Tech, Linux & Open Source, Gaming
- **Trigger:** Wöchentlich Freitags 13:00 Uhr (Cron: `0 13 * * 5`)

## Workflow-Nodes (aktuell deployed, 7 Nodes)
1. `Schedule Trigger` – Freitags 13:00 (Cron `0 13 * * 5`)
2. `Topics definieren` – Code Node, gibt 3 Topics zurück (name, query, subtopics[], webhook_url, color, count)
3. `Split in Batches` – batchSize 1, Loop zurück von Discord Webhook
4. `Perplexity Request vorbereiten` – Code Node, baut Sonar API Body; subtopics werden als Pflicht-Verteilung übergeben
5. `Perplexity API` – HTTP POST zu `api.perplexity.ai/chat/completions` (model: sonar, search_recency_filter: week)
6. `Discord Payload vorbereiten` – Code Node, ersetzt [N]-Marker durch `[📖 Artikel lesen](url)`, Leerzeile nach jedem Item
7. `Discord Webhook` – HTTP POST, Discord Embed

## Topics (aktuell)
- **KI & Tech** – subtopics: Robotik, Large Language Models, KI-Regulierung – count: 5
- **Linux & Open Source** – subtopics: Kernel-Updates, Distributionen, Open Source Projekte – count: 5
- **Gaming** – subtopics: PC Gaming, Konsolen, Neue Releases – count: 5

## Kosten
- Perplexity: ~$0.55/Monat, Key ist im Node "Perplexity API" hinterlegt (https://www.perplexity.ai/settings/api)

## Letzter Stand (15.10.2026, versionId: 4d1d4017)
- Umzug auf neuen Discord-Server (3 neue Webhooks, je ein Kanal pro Topic) ✓
- Auf wöchentlichen Freitags-Digest umgestellt ✓
- Prompt: News der letzten Woche, Discord: "Wochenrückblick MO–FR" ✓
- Workflow ist aktiv (active: true)

## Credentials
- Discord Webhooks: In "Topics definieren" hinterlegt (neuer Discord-Server seit 15.10.2026, ein Kanal pro Topic)
- Perplexity API Key: In Node "Perplexity API" hinterlegt

## Git Repository
- **GitHub:** `icun99/daily-news-ai` (privat)
- **Remote:** `gh-privat:icun99/daily-news-ai.git` (SSH-Alias in `~/.ssh/config`)
- **Branch:** `main`

## Lokale Dateien
- `workflow.json` – Backup/Referenz des deployten Workflows (enthält Workflow-ID)
- `workflow-plan.md` – Ursprüngliche Architektur-Planung

## n8n API-Nutzung
- Direkter Zugriff via REST API (`curl`, Header `X-N8N-API-KEY`)
- API Key in `~/.config/n8n-mcp/dennro.env` (N8N_API_KEY, chmod 600; nie ausgeben)
- Wichtig: `active`-Feld ist bei POST `/api/v1/workflows` read-only (nicht mitsenden)

## MCP-Tools (konfiguriert, aber ggf. nicht in jeder Session aktiv)
- `search_nodes`, `get_node`, `validate_node`
- `n8n_create_workflow`, `n8n_update_partial_workflow`
- `n8n_validate_workflow`, `n8n_deploy_template`
- `n8n_health_check`
