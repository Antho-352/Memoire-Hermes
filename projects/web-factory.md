# Web Factory — Système Multi-Agents Création de Sites

## Identité

- **Path local** : `/Users/anthonyrusso/web-factory/`
- **Type** : Orchestrateur multi-agents (6 agents) pour construire des sites pros en mode pipeline

## Architecture 6 agents

| # | Agent | Type | Prompt |
|---|-------|------|--------|
| 1 | Superviseur | main context | — |
| 2 | Design/UX | general-purpose | agents/design.md |
| 3 | Code/Technique | general-purpose | agents/code.md |
| 4 | Contenu/SEO | general-purpose | agents/content.md |
| 5 | WordPress Integration | general-purpose | agents/wordpress.md |
| 6 | Resource Manager | general-purpose | agents/resources.md |

**Règle** : Seul le Superviseur parle au user. Les agents communiquent via des fichiers dans `projects/<project-name>/`. `project-state.json` = source de vérité unique.

## Workflow

1. User fournit un brief
2. Superviseur crée `projects/<name>/project-state.json`
3. Décomposition en tâches par agent
4. Phase 2 : Exploration parallèle (Design + Contenu + Ressources)
5. Phase 3 : Validation specs avec user
6. Phase 4 : Développement (Code + Contenu + WP en parallèle)
7. Phase 5 : Intégration et QA
8. Phase 6 : Livraison (repo, docs, guide déploiement)

## Structure projet

```
projects/<name>/
  project-state.json  # Shared state
  design/             # Design system, specs, moodboards
  content/            # Content briefs, articles, SEO mapping
  code/               # Code du site (Astro/Next.js)
  wordpress/          # WP config, CPT specs
  resources/          # Data cachées, scraping
```

## Stack par défaut

- Frontend : Astro (préféré) ou Next.js 14/15
- Styling : Tailwind CSS
- Backend : Node.js, API Routes
- DB : PostgreSQL + Drizzle ORM
- CMS : WordPress headless (toujours blog intégré)
- Hosting : AlmaLinux + WHM/cPanel
- Deploy : Git push → cPanel Git Version Control

## Priorités

1. Performance (Lighthouse > 90)
2. SEO (technique + contenu)
3. Sécurité
4. Ratio temps/qualité
