# Règles WordPress — Patterns, Iteration, Redirect

## Iteration discipline (sites WP live)

Pour tout bug sur un site WP live, cycle obligatoire :

1. **Diagnostic d'abord** : `curl` la page, screenshot headless, grep DOM pour les signatures attendues
2. **Une hypothèse, une version** : pas de bundle "je fixe logo + filtre + sécurité en une fois"
3. **Vérifier l'état avant diagnostic** : demander "upload fait ? plugin activé ? Cloudflare purgé ?"
4. **Solution minimale viable en premier** : pour override template FSE → `add_shortcode` (bulletproof) avant `get_block_template` filter (fragile)
5. **Documenter les guards défensifs** : un `source === 'custom'` oublié désactive silencieusement l'override
6. **Pour problème visuel** : screenshot live via Chrome headless. Ne jamais se fier au code "pensé"

Avant de ship : `curl` + screenshot + grep signatures.
Après livraison : confirmer upload/activation, re-fetch.
Si fix échoue 2 fois de suite : demander screenshot DevTools avant 3e fix.

## Favicon et logo

- **Favicon / Site Icon** : toujours **PNG** (WP refuse SVG pour Site Icon)
- **Custom Logo** : SVG OK
- Conversion SVG → PNG : `rsvg-convert`, `qlmanage -t -s 512`, ou Inkscape
- Format PNG : 512×512 minimum (WP génère 16/32/180/192 automatiquement)

## Redirect subdomain WP → domaine principal (.htaccess)

**Obligatoire** sur tout site Astro + WP headless. Sans ça, contenu dupliqué indexé sur `wp.site.fr`.

Placer AVANT les règles WordPress standard :

```apache
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /

# Rule 1: Homepage redirect (cas spécial — est un répertoire)
RewriteCond %{HTTP_HOST} ^wp\.site\.fr$ [NC]
RewriteCond %{REQUEST_URI} ^/$ [NC]
RewriteRule ^(.*)$ https://site.fr/ [R=301,L]

# Rule 2: Articles/pages redirect (exclut les fichiers physiques)
RewriteCond %{HTTP_HOST} ^wp\.site\.fr$ [NC]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_URI} !^/wp-admin [NC]
RewriteCond %{REQUEST_URI} !^/wp-login\.php [NC]
RewriteCond %{REQUEST_URI} !^/wp-json [NC]
RewriteCond %{REQUEST_URI} !^/wp-includes [NC]
RewriteCond %{REQUEST_URI} !^/wp-content [NC]
RewriteRule ^(.*)$ https://site.fr/$1 [R=301,L]

# WordPress standard routing...
</IfModule>
```

**Détails critiques** :
- 2 règles obligatoires : homepage séparée (contourne le check `!-d`), puis règle générale
- Sans `!-f !-d` : boucles de redirections infinies sur fichiers physiques
- Exceptions `/wp-admin`, `/wp-login.php`, `/wp-json`, `/wp-includes`, `/wp-content` = obligatoires
- Placer dans le `.htaccess` WordPress root (pas dans le site Astro)

**Problèmes fréquents** :
- Une seule règle sans exception homepage : homepage ne redirige pas (est un répertoire, bloquée par `!-d`)
- Mauvais emplacement `.htaccess` : doit être dans la racine WordPress, pas le site Astro

## ARW Pulse — Pattern override template FSE

**Méthode de référence** (ne pas utiliser `ob_start()` ni `get_block_template` filter) :

```php
// Thème expose le shortcode dispatcher
// [arw_pulse_front_page] dans templates/front-page.html

// Plugin override via filtre
add_filter('arw_pulse_front_page_markup', function() {
    return '<!-- wp:pattern {"slug":"arw-pack-niche/home-niche"} /-->';
}, 20);

// Toujours do_shortcode(do_blocks()) pour cascade
return do_shortcode(do_blocks($markup));
```

**Ne pas utiliser** :
- `ob_start()` sur `template_redirect` (peut crasher site blanc si interaction buffer)
- `get_block_template` filter (fragile, cache, guard `source === 'custom'` silencieux)

**Assets conditionnels** : enqueue depuis callbacks shortcode, pas depuis `wp_enqueue_scripts` avec `has_shortcode` (les FSE templates n'ont pas de `post_content`).

**Patterns plugins** : ne sont pas auto-chargés (contrairement aux thèmes). Itérer `glob()` + `register_block_pattern()` au hook `init`.

## Stack rules (tous projets WP d'Anthony)

- ❌ GeneratePress parent theme (ARW Pulse est from scratch)
- ❌ Yoast/RankMath (SEO built-in dans le thème ou plugin custom)
- ❌ Contact Form 7 (form natif WP block → CPT `arw_submission` + wp_mail)
- ❌ AdSense (affiliation + B2B lead-gen uniquement)
- ❌ Tailwind ou framework CSS sur les nouveaux thèmes WP
- ❌ jQuery sur le front (admin WP OK)
- ❌ Webfonts non self-hostées en runtime
- ❌ Dépendances externes en runtime

## Protections REST/Sécurité (standard tous plugins)

```php
// Rate limit IP via transients
// Path traversal via realpath() vs wp_get_upload_dir()['basedir']
// Sanitize tous les $_GET/$_POST
// Escape tous les outputs
// Nonces obligatoires sur tous les formulaires
// Capabilities obligatoires sur toutes les routes REST écrivant en DB
// REST /wp/v2/users désactivé (anti-énumération)
```

## WPS Hide Login

Si actif sur le serveur : casse `/wp-admin` et `/wp-login.php`. À désactiver.

## Permaliens

Flush manuel via Réglages → Permaliens → Save après ajout d'un CPT (sinon URLs en `?post_type=...`).
