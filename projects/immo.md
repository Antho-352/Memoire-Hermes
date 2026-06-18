# Workspace Immo — Sites Éditoriaux Immobilier

## Identité

- **Path local** : `/Users/anthonyrusso/immo/`
- **Type** : Workspace multi-projets immobilier / maison / déco
- **Stratégie git** : multi-repos (chaque sous-projet = son propre repo GitHub `Antho-352/<nom>`)
- **Hébergement** : OVHcloud Kimsufi (LEMP)

## Structure workspace

```
/immo/
├── CLAUDE.md                          # Invariants multi-projets
├── _boilerplate/plugin/               # Briques mutualisées starter
└── simulation-immobilier/             # Projet actif
    └── CLAUDE.md, wp-content/, content/, docs/, dist/
```

## Projet actif : simulation-immobilier.fr

- Repo : github.com/Antho-352/simulation-immobilier
- Type : site d'outils de simulation immobilière
- Thème + plugin custom (namespace `Simimmo\`)

## Stack obligatoire (tous projets)

- WordPress full (pas Astro pour ce workspace)
- Thème custom from scratch (pas `_s`, pas thème téléchargé)
- Plugin custom par site (namespace propre)
- PHP 8.1+ strict (strict_types, enums, readonly)
- MySQL 8.0+
- Pas de plugins tiers (toute exception documentée dans le CLAUDE.md projet)

## Interdictions formelles

- ❌ Tailwind ou framework CSS
- ❌ Page builders (Elementor, Divi, WPBakery, Beaver, Bricks)
- ❌ Webfonts non self-hostées (pas de Google Fonts en runtime)
- ❌ jQuery sur le front (admin WP OK)
- ❌ Dépendances externes en runtime (pas GA, pas CDN externe)
- ❌ Plugins tiers non justifiés

## Conventions PHP/WP par site

- Namespace racine : `Brand\` (PSR-4 manuel, sans Composer pour Kimsufi)
- Préfixe DB tables : `wp_brand_*`
- Préfixe options/transients : `brand_*`
- Endpoints REST : `/wp-json/brand/v1/*`
- `declare(strict_types=1);` en tête de chaque fichier PHP
- Nonces obligatoires sur tous les formulaires
- Capabilities obligatoires sur toutes les routes REST

## Frontend CSS/JS

- CSS manuel, variables CSS sur `:root`
- Architecture : `theme.css` + `components.css` + `pages.css`
- JS vanilla ES2020+, esbuild minimal max
- Lazy-load systématique des assets lourds
- Aucun appel externe en runtime côté navigateur

## Boilerplate plugin (`_boilerplate/plugin/`)

Briques réutilisables à copier pour chaque nouveau site :
- **SeoMeta** : panel Gutenberg title + meta description par post
- **Schema** : JSON-LD (Organization, WebSite, BreadcrumbList, Article, WebApplication)
- **Newsletter** : double opt-in + rate limit + hash IP + FluentSMTP
- **MdToBlocks** : parser Markdown → blocs Gutenberg
- **ImportContent** : page admin "Import 1-clic" depuis `content/<cpt>/*.md`
- **Security** : headers (CSP, HSTS, X-Frame-Options), désactivation generator/RSD/wlwmanifest
- **RestRateLimit** : middleware rate limit IP hashée

Démarrage nouveau projet : copier boilerplate, renommer `Brand\` → `NouvelleClasse\`, ajuster préfixes.

## Versioning

- SemVer thème + plugin
- Bump version à CHAQUE modification visible
- Re-générer zip à chaque bump
- Commit + push GitHub immédiatement

## Voix éditoriale (sites éditoriaux)

- Neutre, factuelle, evergreen
- Pas de date dans les titres
- Pas de superlatifs
- Sources officielles dans le corps du texte
- Mention non-conseil sur sujets fiscaux/juridiques/médicaux/financiers
- **Ne jamais inventer de fait personnel sur Anthony** (notamment dans les pages "À propos")

## Garde-fous juridiques

- DGCCRF : transparence affiliation obligatoire, `rel="sponsored nofollow"`
- Aucun titre réglementé revendiqué (pas CIF, IOBSP, médecin, avocat, notaire, expert-comptable)
- RGPD : pas de tracker tiers, newsletter double opt-in
- LCEN : éditeur + directeur de publication + hébergeur dans les mentions légales
- Pages légales obligatoires : mentions-légales, politique-confidentialité, contact, méthodologie

## Perf targets

- LCP < 1.5s sur 4G
- CLS < 0.05
- Lighthouse > 90 tous critères
- CSS total < 30 Ko gzip
- JS critique < 50 Ko gzip

## Workflow nouveau projet

1. Créer `/immo/<nom-projet>/`
2. Repo GitHub `Antho-352/<nom>`, git init, remote add
3. Skill `/persona-redacteur` → `docs/persona-redacteur.md`
4. Copier boilerplate plugin, renommer namespace
5. Créer thème custom from scratch
6. CLAUDE.md projet (domaine, repo, design system, fonctionnalités spécifiques)
7. Premier commit + push
