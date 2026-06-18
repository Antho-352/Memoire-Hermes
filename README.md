# Memoire-Hermes

Base de connaissance structurée d'Anthony Russo pour l'agent Hermes.

## Structure

```
HERMES.md                    ← Fichier maître (system prompt de Hermes)
identity.md                  ← Profil Anthony, règles de communication, mentions légales
infrastructure.md            ← Serveurs, sites, domaines, paths locaux
rules/
  execution.md               ← Discipline exécution, versioning, pipeline agents
  wordpress.md               ← Patterns WP, iteration, redirect, override FSE
  deployment.md              ← Zip WP/Astro, deploy linkttp, anthonyrusso.fr
  seo.md                     ← Pipeline éditorial SEO, voix, sources autorisées
projects/
  arw-pulse.md               ← Thème FSE ARW Pulse + packs niche
  link-ttp.md                ← Marketplace Linkttp (Next.js 15)
  seoflix.md                 ← Agrégateur YouTube SEO
  anthonyrusso-fr.md         ← Site perso Astro 6 + WP headless
  audit-logs.md              ← Pipeline crawl logs DuckDB + FastAPI
  immo.md                    ← Workspace multi-sites immobilier
  calcio.md                  ← Vai calcio (Astro 5 + Serie A)
  amenagement-labo.md        ← Site B2B labo (Astro 5)
  frigo-labo.md              ← Site B2B froid labo (Astro 5)
  web-factory.md             ← Système 6 agents création sites
credentials.template.yaml    ← Template credentials (versionné, sans valeurs)
credentials.age              ← Credentials chiffrés (non versionné)
.gitignore                   ← Exclut credentials.yaml déchiffré
```

## Usage avec Hermes

Charger `HERMES.md` en tant que system prompt ou contexte initial. Charger les modules détaillés selon le projet en cours.

## Gestion des credentials

### Setup initial

```bash
# Installer age (macOS)
brew install age

# Copier le template, remplir les valeurs
cp credentials.template.yaml credentials.yaml
# Éditer credentials.yaml avec toutes les valeurs

# Chiffrer (choisir une passphrase forte, la stocker dans ton gestionnaire de mots de passe)
age -p -o credentials.age credentials.yaml

# Supprimer le fichier déchiffré
rm credentials.yaml
```

### Déchiffrer pour consulter/modifier

```bash
age -d credentials.age > credentials.yaml
# Éditer...
rm credentials.age && age -p -o credentials.age credentials.yaml
rm credentials.yaml
```

### Sécurité

- `credentials.age` peut être commité (chiffré, inutilisable sans la passphrase)
- `credentials.yaml` ne doit **jamais** être commité (listé dans `.gitignore`)
- La passphrase doit être stockée dans un gestionnaire de mots de passe (Bitwarden, 1Password, etc.)
- Changer la passphrase régulièrement : `age -d credentials.age > credentials.yaml && rm credentials.age && age -p -o credentials.age credentials.yaml && rm credentials.yaml`

## Mise à jour de la base de connaissance

Mettre à jour les fichiers concernés quand :
- Nouveau projet créé → créer `projects/<nom>.md` + ajouter dans `HERMES.md`
- Infrastructure change (nouveau serveur, nouveau site) → mettre à jour `infrastructure.md`
- Nouvelle règle validée → ajouter dans le fichier `rules/` concerné
- Nouvelle credential → ajouter dans `credentials.template.yaml` + rechiffrer `credentials.age`
