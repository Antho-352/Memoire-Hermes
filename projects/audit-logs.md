# audit-logs — Pipeline Analyse Crawl SEO/GEO

## Identité

- **Path local** : `/Users/anthonyrusso/audit-logs/`
- **Objectif** : Analyser comment Google et les bots LLM crawlent 24 sites WP éditoriaux d'Anthony
- **Usage** : comprendre fréquence, fenêtre, budget, erreurs, pages orphelines, divergence Google vs LLMs

## Stack

- Python 3.11+ / DuckDB (single-file) / FastAPI + Jinja2 + Chart.js / YAML config
- Dashboard local sur `localhost:8765`

## Commandes

```bash
audit init          # Init DuckDB + tables
audit run-all       # Ingest logs + classif + compute métriques
audit serve         # Lance webapp localhost:8765

# Sync logs depuis serveur (scripts/sync_logs.sh)
bash scripts/sync_logs.sh   # SERVER=root@IP défini dans le script (ne pas committer l'IP)

# Import GSC
audit ingest-gsc --site X --type performance file.csv
```

## Infrastructure serveur

- OVH dédié Debian, cPanel/WHM 134.0.20, Apache 2.4.66
- **Cloudflare proxy** sur tous les sites mais `mod_remoteip` auto-configuré → vraies IPs loggées
- LogFormat combined : `%a %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i"`
- Logs courants : `/usr/local/apache/domlogs/<domain>` + `<domain>-ssl_log`
- Archives mensuelles : `/home/<cpanel_user>/logs/<domain>-Mois-Année.gz`
- Le log global `/usr/local/apache/logs/archive/access_log-*.gz` ne contient PAS le vhost → inutilisable par site

## 24 sites prioritaires

Sur 44 sites hébergés — config dans `config/sites.yaml` — 7 catégories :
editorial_loisirs, cuisine, b2b_industrie, services_reparation, maison_deco, voyage_tourisme, business

## Données disponibles

- Avril 2026 minimum (avant = perdu car "Keep stats logs" était OFF dans WHM)
- À partir de mai 2026 : archives mensuelles s'accumulent

## Pipeline

| Module | Rôle |
|--------|------|
| `pipeline/parse_logs.py` | Regex Apache combined + IPv4/IPv6 + malformed fallback |
| `pipeline/classify_bots.py` | UA matching (40+ bots bots.yaml) + reverse DNS vérifié |
| `pipeline/normalize_urls.py` | 18 types d'URL (article, tag, asset, attack_pattern, wp_admin...) |
| `pipeline/ingest_gsc.py` | Parser CSV GSC bilingue EN/FR |
| `pipeline/compute_metrics.py` | 11 vues SQL analytiques |
| `pipeline/cli.py` | CLI Click unifié |

## Vues SQL analytiques

- `v_hits_enriched`, `v_site_daily`, `v_bot_crawl_stats`
- `v_crawl_budget_waste`, `v_top_crawled_urls`, `v_orphan_urls`
- `v_bot_errors`, `v_url_crawl_frequency`, `v_crawl_window`
- `v_google_vs_llm_overlap`, `v_weekly_evolution`

## Webapp

- FastAPI + routes : `/`, `/category/{name}`, `/site/{domain}` + 5 sous-routes
- 8 templates Jinja2, thème dark CSS custom
- 3 endpoints JSON pour Chart.js (daily, window, weekly)

## GSC export

Export manuel : 4 CSV × 24 sites = ~3-4h de travail. Guide : `docs/gsc-export-guide.md`
