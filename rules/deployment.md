# Règles de déploiement

## Zip WordPress plugin/thème — méthode obligatoire

### Structure correcte (dossier parent requis)

WP exige le dossier parent dans le zip (sinon WP traite ça comme un nouveau plugin/thème et ne propose pas "Remplacer l'existant") :

```bash
# Plugin
rm -f /path/to/dist/<plugin-slug>.zip
cd /path/to/wp-content/plugins
zip -r /path/to/dist/<plugin-slug>.zip <plugin-slug> --exclude "*/.DS_Store" "*/__MACOSX/*"

# Thème
rm -f /path/to/dist/<theme-slug>.zip
cd /path/to/wp-content/themes
zip -r /path/to/dist/<theme-slug>.zip <theme-slug> --exclude "*/.DS_Store" "*/__MACOSX/*"
```

**Structure attendue** :
- Plugin : `<plugin-slug>/<plugin-slug>.php` au top (jamais à la racine)
- Thème : `<theme-slug>/style.css` au top
- Header `Plugin Name:` / `Theme Name:` dans les 8 premiers Ko

**Vérification** : `unzip -l dist/<name>.zip | head -3` doit afficher le dossier en premier.

### ⚠️ TOUJOURS rm avant rezip

`zip -r` **AJOUTE** par défaut, ne remplace pas. Si le zip existe déjà avec une mauvaise structure, le nouveau zip va mélanger les deux structures → erreur "Cette extension ne dispose pas d'un entête valide" dans WP.

**TOUJOURS `rm -f` avant de recréer un zip.**

### Filename fixe (sans timestamp)

- Nom fixe : `seoflix-core.zip`, `arw-pulse.zip`, `psm-labo-fr.zip`
- **PAS de timestamp** : `site-20260324.zip` → interdit
- L'écrasement automatique = préférence explicite

## Zip Astro (déploiement via cPanel File Manager)

Pour les sites Astro déployés via zip (sans SSH ni deploy.sh) :

```bash
# Zipper le CONTENU de dist, pas le dossier dist lui-même
cd /path/to/project/dist && zip -r ../project-name.zip .
```

**Mauvais** : `zip -r site.zip dist/` → crée `dist/index.html` dans le zip
**Bon** : `cd dist && zip -r ../project-name.zip .` → `index.html` à la racine du zip

## Déploiement linkttp.com

```bash
./deploy.sh                # Full deploy (deps + db + build)
./deploy.sh --skip-deps    # Skip npm install
./deploy.sh --skip-db      # Skip db:push

# Logs en prod
pm2 logs link-ttp

# Rebuild si ENOENT .next cache errors
ssh server "cd /home/linkttp/repositories/link-ttp && rm -rf .next && npm run build"
```

**Important** :
- Toujours push avant deploy (le script pull depuis remote, discard local server changes)
- `.env.production` n'est PAS dans git — doit exister sur serveur ET être copié dans `.next/standalone/` après build
- Parenthèses dans les paths git : `git add "src/app/(public)/page.tsx"` (guillemets obligatoires)

## Déploiement anthonyrusso.fr

```bash
# Via GitHub (méthode recommandée)
git add . && git commit -m "message" && git push
ssh anth543@server "cd /home/anth543/anthonyrusso-fr && git pull && npm run build && npx pm2 restart anthonyrusso"

# Via deploy.sh (build local + scp + restart)
./deploy.sh
```

**Important** :
- PM2 doit lancer `dist/server/entry.mjs`, pas `astro preview`
- Changement env var : delete + recreate process PM2 (restart ne recharge pas l'env)
- Apache proxy : IPv6 `[::1]:3006` obligatoire
- Après modif Apache : `/scripts/ensure_vhost_includes && /scripts/rebuildhttpdconf && /scripts/restartsrv_httpd`

## Next.js — problèmes courants

- **Build hang** (jest-worker 100%+ CPU) : admin pages avec shell commands (exec, spawn) DOIVENT avoir `export const dynamic = 'force-dynamic'`
- **"column X does not exist"** : schema Drizzle désynchronisé → `npm run db:push` sur serveur après deploy
- **Prerendered pages pas à jour** : rebuild + restart PM2

## Script deploy.sh seoflix

Non disponible (upload zip via WP admin). Voir `rules/wordpress.md` pour la procédure zip.

## Checklist avant toute livraison WP

```bash
# 1. Version bumpée dans les 2 endroits
grep -E "Version:|_VERSION = '" path/to/plugin.php

# 2. Zip sans le précédent
rm -f dist/<name>.zip && cd ... && zip -r ...

# 3. Structure zip correcte
unzip -l dist/<name>.zip | head -3

# 4. Pas de fichiers orphelins
unzip -l dist/<name>.zip | awk 'NR>3 && NF>=4 {for(i=4;i<=NF;i++) printf "%s ", $i; print ""}' \
  | grep -v "^<slug>/" | grep -v "^-" | grep -v "files$"

# 5. Pas de .DS_Store ni __MACOSX
unzip -l dist/<name>.zip | grep -E "DS_Store|__MACOSX"
```
