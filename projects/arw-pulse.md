# ARW Pulse — Thème WordPress FSE Custom

## Identité

- **Path local** : `/Users/anthonyrusso/generatepress/arw-pulse/`
- **Type** : FSE (Full Site Editing) block theme, from scratch, vanilla CSS, 0 JS par défaut
- **Objectif** : thème adaptable sur tous les sites d'Anthony (moto, cuisine, pêche, ferme, etc.)
- **Premier site cible** : citymoto.fr (moto urbaine, affiliation)
- **Perf targets** : Lighthouse 100/100/100/100, <50kb CSS, 0 JS par défaut
- **Archétype** : Editorial Magazine (1er sur 3 prévus : Industrial, Craft)

## Architecture 3 couches

### 1. Thème ARW Pulse
Modules dans `/inc/` :
- `admin-menu.php`, `seo.php`, `structured-data.php`
- `form.php` — CPT `arw_submission` + wp_mail (pas de CF7)
- `products.php` — CPT `arw_product` + taxonomy `arw_product_cat`
- `front-page.php` — dispatcher shortcode `[arw_pulse_front_page]`
- `branding.php` — fallback logo SVG
- `cookie-consent.php`, `motion.php`
- Patterns citymoto par défaut dans `/patterns/`

**Point d'entrée home** : shortcode `[arw_pulse_front_page]` dans `templates/front-page.html` → filtre `arw_pulse_front_page_markup`

### 2. Skins (`/arw-pulse/styles/*.json`)
Style variations — tokens couleurs + fonts uniquement :
- `urban.json` — citymoto.fr (dark base + jaune `#ffd600`)
- `field.json` — lecampdubivouaqueur.fr (ivoire + vert forêt + terracotta)
- `citrus.json` — lesdelicesdeb.fr (orange `#ff5b1f` + vert spiruline `#0f5d3b` + crème `#faf4e4`, pop/risographique)

### 3. Packs de niche (plugins séparés)

Path : `/Users/anthonyrusso/generatepress/arw-pack-<niche>/`

Packs existants :
- `arw-pack-outdoor` → lecampdubivouaqueur.fr (CPT spot/trail, Leaflet, Overpass, Nominatim)
- `arw-pack-delices` → lesdelicesdeb.fr (CPT decode + menu_semaine, Macro Tool, tagline rotative)
- `arw-pack-maison` → en cours
- `arw-pack-cuendet` → tourisme suisse (cuendet.fr)

## Filtres d'extension (thème)

```php
arw_pulse_front_page_markup      // override markup home (prio ≥ 20 pour packs)
arw_pulse_hero_front_content     // override contenu hero default
arw_pulse_manifesto_content      // override contenu manifesto default
arw_pulse_meta_description       // override meta desc site-wide
arw_pulse_default_og_image       // URL OG:image fallback
arw_pulse_og_type                // override og:type par contexte
arw_pulse_form_types             // whitelist form_type
arw_pulse_form_redirect          // URL redirection post-submit par form_type
arw_pulse_enable_products        // toggle module produits
arw_pulse_breadcrumbs            // modifier fil d'ariane
```

## Règles d'intégration thème ↔ pack

- **Jamais** modifier le thème pour du contenu site-specific. Tout passe par packs + filtres
- **Jamais** de `require_once` entre thème et pack. Guards `function_exists()` / `defined()` systématiques
- **Shortcodes imbriqués** : `do_blocks()` ne processe PAS les shortcodes → toujours `do_shortcode(do_blocks(...))`
- **Override FSE** : pas via `get_block_template` filter. Utiliser le shortcode dispatcher
- **Assets conditionnels** : enqueue depuis callbacks shortcode (pas `wp_enqueue_scripts` + `has_shortcode`)
- **Patterns plugins** : auto-chargement via `glob()` + `register_block_pattern()` au hook `init`

## Structure d'un pack de niche

```
arw-pack-<niche>/
├── arw-pack-<niche>.php      # Plugin header + bootstrap + filter hooks home
├── package.sh                # Zip → ../arw-pack-<niche>.zip
├── inc/
│   ├── cpt-<nom>.php         # CPT + meta box admin
│   ├── taxonomies.php
│   ├── schema.php            # JSON-LD spécifiques
│   ├── <niche>-display.php   # Shortcodes rendu
│   ├── seo-integration.php   # Filtres breadcrumbs, meta desc
│   └── home-shortcodes.php   # Shortcodes home du pack
├── patterns/
│   └── home-<niche>.php      # Pattern home
├── templates/
│   └── single-<cpt>.html     # Templates FSE (slug-based)
└── assets/css/ js/ vendor/
```

## Création d'un nouveau site/niche — checklist

1. Copier ou utiliser ARW Pulse tel quel
2. Créer skin `/arw-pulse/styles/<niche>.json` (tokens couleurs + fonts)
3. Créer pack `/arw-pack-<niche>/` (copier structure outdoor)
4. Hook obligatoire :
   ```php
   add_filter('arw_pulse_front_page_markup', function() {
       return '<!-- wp:pattern {"slug":"arw-pack-<niche>/home-<niche>"} /-->';
   }, 20);
   ```
5. CPTs, taxonomies, templates FSE, shortcodes enqueue conditional

## Pitfalls connus

- **`ob_start()` sur `template_redirect`** → crash site blanc possible (incident 2026-04-24)
- **`get_block_template` filter** → fragile, cache, guard `source === 'custom'` silencieux
- **`do_blocks()` ne cascade pas** sur les shortcodes → `do_shortcode(do_blocks(...))`
- **Patterns plugins pas auto-chargés** → `glob()` + `register_block_pattern()` au hook `init`
- **Block templates plugins overridés** par ceux du thème si même slug → slug différent requis

## Pack arw-pack-delices — particularités

- **Taxonomy 3 piliers** : `arw_pillar` (Food / Nutra / Sport) — attachée aux posts + CPT decode
- **CPT `arw_decode`** : articles décodage produits commerce. Meta : marque, produit, verdict, équivalent fait-maison, prix/100g, date vérif. JSON-LD Review auto
- **CPT `arw_menu_semaine`** : 1 post/semaine sur la home (rubrique éditoriale)
- **Macro Equivalence Tool** `[arw_macro_tool]` : base statique `assets/data/foods.json` (114 aliments Ciqual ANSES + 13 suppléments). Zéro nom de marque → zéro risque légal EU 1924/2006. JS vanilla 9KB
- **Tagline rotative** "B comme Batch / Base / Bien / Brut / Bol" : `[arw_delices_tagline]` + JS 1KB

## Versioning + packaging

- Thème et pack ont chacun leur `package.sh` (zip à `../arw-pulse.zip` / `../arw-pack-<niche>.zip`, nom fixe, overwrite)
- Bump version : header `Version: X.Y.Z` + constante `ARW_OUTDOOR_VERSION` (équivalent par pack)
- Thème : bump `Version: X.Y.Z` dans `style.css`
