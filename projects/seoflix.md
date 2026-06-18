# Seoflix — Agrégateur YouTube SEO

## Identité

- **Path local** : `/Users/anthonyrusso/seoflix/`
- **Domaine** : seoflix.fr
- **GitHub** : https://github.com/Antho-352/seoflix (auth HTTPS via macOS Keychain)
- **Hébergement** : Kimsufi cPanel
- **Stack** : WordPress + plugin custom `seoflix-core` + thème FSE custom

## Objectif

Plateforme Netflix-like d'agrégation de vidéos YouTube francophones sur le SEO.

## Architecture locale

```
seoflix/
├── plugin/seoflix-core/   # CPT, tax, YouTube API, affiliation
├── theme/seoflix/         # Thème FSE, vanilla CSS, noir + rouge
├── backlog/               # JSON d'import initial (yt-dlp)
├── scripts/fetch-backlog.py  # Script yt-dlp fallback
├── docs/                  # DEPLOY.md, IMPORT_FORMAT.md, LEGAL_PAGES.md
└── dist/                  # Zips livrables
```

## Plugin seoflix-core

Namespace PHP : `Seoflix\` / `Seoflix\Admin\`

CPTs : `seoflix_video`, `seoflix_channel`, `seoflix_product`
Taxonomies : `seoflix_topic`, `seoflix_format`, `seoflix_path`, `seoflix_product_category` (tous hiérarchiques)

Modules :
- `FeatureFlags` : `seoflix_user_accounts_enabled` (V2 dormant)
- `Affiliate` : tracker `/go/[slug]` → table `wp_seoflix_affiliate_clicks`
- `YouTube_API` : wrapper Data API v3 + protections quota
- `Channel_Meta` / `Video_Meta` : metaboxes + auto-détection produits par regex
- `Admin\Admin_Columns` : colonnes custom + bulk actions

## Tables custom

- `wp_seoflix_favorites` (V2 dormant)
- `wp_seoflix_watch` (V2 dormant)
- `wp_seoflix_affiliate_clicks` (V1 actif)

## Thème

- Palette : noir `#0B0B0F` + `#16161D` + rouge `#FF2D3F` + texte `#F5F5F7`
- Logo : triangle rouge pointant vers le haut sur carré noir (distinct de Netflix)
- Vanilla CSS, hybride classic + theme.json, pas de Tailwind

## Protections quota YouTube API

3 couches :
1. Cooldown 30s entre 2 calls
2. Cap horaire : 200 unités (transient `seoflix_yt_quota_hour`)
3. Cap journalier : 1000 unités (transient `seoflix_yt_quota_day`)

**IPv4 forcé** via `http_api_curl` filter — le serveur Kimsufi sort en IPv6 par défaut, ce qui casse la whitelist IP de la clé API Google. Ne jamais retirer ce filter.

## Politique IA (catégorisation)

- **PAS de clé API Anthropic en prod** (refus, coût)
- Backlog initial (~50 vidéos) : agents Claude Code → JSON pour import one-shot
- Ingestion continue : "Option B" — Anthony ouvre une session Claude Code quand le plugin a des vidéos en attente, agents catégorisent en lot

## Affiliation

- CPT `seoflix_product` lié aux vidéos
- Page `/outils` (catalogue)
- Liens affiliés wrappés via `/go/[slug]` pour tracking
- `rel="sponsored nofollow"` obligatoire
- Page `/affiliation` avec mentions légales DGCCRF

## V1 vs V2

V1 (actif) : 100% public, pas de comptes.
V2 (dormant) : comptes user (favoris, historique). Code écrit, derrière `FeatureFlags::user_accounts_enabled()`. Tables déjà créées.

## Workflow de livraison zip

```bash
# Plugin
cd plugin && zip -rq ../dist/seoflix-core.zip seoflix-core -x "*.DS_Store" -x "*.git*"

# Thème
cd theme && zip -rq ../dist/seoflix-theme.zip seoflix -x "*.DS_Store" -x "*.git*"
```

**Le zip DOIT contenir le dossier parent** (sinon WP traite comme nouveau plugin = duplicatas).
**Filenames fixes** (jamais de timestamp).

## Versioning

Bumper `SEOFLIX_VERSION` dans `seoflix-core.php` + header plugin ET `Version:` dans `theme/seoflix/style.css` à chaque release.

## Git workflow

Push après chaque changement significatif. Commits en français, conventional commits.
Pas de PR/branches en V1.

## Gotchas

- **WPS Hide Login** : si actif casse `/wp-admin`. À désactiver
- **Permaliens** : flush manuel après ajout CPT
- **Channels archive** : INNER JOIN sur `meta_key=_seoflix_subscriber_count` exclut nouvelles chaînes. Order by `title ASC` via `pre_get_posts`
- **Auto-création menu** : skip si menu déjà assigné → user doit ajouter items à la main

## Backlog initial (50 vidéos)

Wizards Podcast (7), Paul Vengeons (5), BHC France (4), Léo Poitevin (4), MVP Podcast (3), Areseo (3), Gabin Web (3), Sylvain Peyronnet (3), Jérôme Pasquelin (3), Linkuma (3), Linksgarden (3), Les Wizards (3), Florentin Doreau (2), Victor Collas (2), Pimpton SEO (2).
