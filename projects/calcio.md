# vai-calcio.fr — Site News Football Italien

## Identité

- **Path local** : `/Users/anthonyrusso/calcio/`
- **Domaine** : vai-calcio.fr
- **Email** : contact@vai-calcio.fr
- **Type** : News et informations Serie A
- **Stack** : Astro 5 (SSG/SSR hybrid) + WordPress headless (`wp.vai-calcio.fr`) + Tailwind CSS + Node.js

## Architecture rendu

**Default : statique** (`output: 'static'` dans astro.config.mjs)

**SSR** (nécessite `export const prerender = false`) :
- `/api/*` — toutes les routes API (revalidate, newsletter, scores, standings)
- `/sitemap.xml` — sitemap dynamique
- `/blog/[slug]` — articles individuels (SSR + cache prévu)

**Statique avec getStaticPaths** :
- `/equipes/[slug]` — 20 équipes Serie A
- `/arbitres/[slug]` — 15 arbitres
- `/joueurs/[slug]`

Démarrage dev : port **5432** (intentionnel, pas 4321 par défaut Astro).

## Design system

**Couleurs** (immuables, définies dans `tailwind.config.mjs`) :
- `background: #F5F5F0` (off-white)
- `accent: #2D7A3A` (green)
- `accent-light: #E8F5E9`
- `separator: #E0E0E0`

**Typographies** (self-hosted dans `/public/fonts/` WOFF2) :
- Titres : Barlow Condensed (600, 700) — condensé, sportif
- Corps : Inter (400, 500, 600)

Style : dense, éditorial, newspaper-like, mobile-first, grille 3-4 colonnes desktop.

## WordPress headless

- **URL** : `wp.vai-calcio.fr`
- Redirect subdomain configuré ✅ (voir rules/wordpress.md)
- GraphQL via WPGraphQL — queries centralisées dans `lib/wordpress.ts`
- Webhook `/api/revalidate` pour invalider le cache lors de publication WP (`REVALIDATE_SECRET`)

## Multi-API Football avec fallback

`lib/football-api.ts` — stratégie multi-API avec fallback automatique sur mock data.

**Ne jamais throw depuis les fonctions API**. Toujours retourner le mock data en fallback.

**TTLs** :
- Classements/Calendrier : 24h
- Résultats matchs : 6h
- Stats joueurs/équipes : 7 jours
- Scores live : 30s (memory uniquement pendant fenêtres de match)

Cache fichier `.cache/api-cache.json` — persiste entre les restarts PM2.

## Variables d'environnement

Obligatoires en prod :
- `WORDPRESS_GRAPHQL_URL` — `https://wp.vai-calcio.fr/graphql`
- `REVALIDATE_SECRET`
- `BREVO_API_KEY` + `BREVO_LIST_ID`

Optionnelles (fallback mock data) :
- `FOOTBALL_API_KEY`, `FOOTBALL_DATA_API_KEY`, `THESPORTSDB_API_KEY`

## Pitfalls connus

- **Dynamic routes sans getStaticPaths** : crash. Toujours ajouter `getStaticPaths()`
- **Template literals dans JSX** : erreur de syntaxe Astro. Pré-calculer dans le frontmatter
- **Serie A = 10 matchs par journée** (20 équipes / 2) dans les mock data
- **Zip déploiement** : `cd dist && zip -r ../vai-calcio-fr.zip .` (filename fixe, pas de timestamp)

## Organisation composants

- **Layouts** : `BaseLayout` → `PageLayout` → `ArticleLayout`
- **Islands** : Preact dans `components/islands/` (`client:load` sparingly)
- **Lib** : fonctions pures, sans side effects, toujours typées
