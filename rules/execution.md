# Règles d'exécution — Discipline de travail

## Règle #1 : Exécuter le plan sans redemander

Quand Anthony dit "go" :
1. Annoncer le plan brièvement
2. L'exécuter intégralement sans redemander à chaque étape
3. **Ne jamais finir** par "lequel je traite en premier ?" ou "tu veux que je commence par X ou Y ?"
4. Si vraiment ambigu (choix structurant, irréversible) : poser UNE question claire — pas un menu
5. Si Anthony veut intervenir, il le fera

## Règle #2 : Versioning plugin/thème WP — obligatoire

**Avant chaque zip** : bumper la version dans DEUX endroits :
- Header du plugin/thème : `* Version: X.Y.Z`
- Constante interne : `const ARW_*_VERSION = 'X.Y.Z';` (ou équivalent)

Types de bump :
- **Patch** (X.Y.Z → X.Y.Z+1) : fix CSS, modification mineure, patch UI
- **Mineur** (X.Y.Z → X.Y+1.0) : nouvelle feature, nouveau shortcode, nouveau hook
- **Majeur** (X.Y.Z → X+1.0.0) : breaking change, refacto profonde

Sanity check avant zip :
```bash
grep -E "Version:|_VERSION = '" path/to/plugin.php
```
Les deux doivent matcher. Si ça matche ce qui est déjà livré : bumper avant de zipper.

**Ne jamais** :
- Rebuilder un zip avec la même version qu'avant
- Bumper une seule des deux références
- Bumper après le zip

## Règle #3 : Vérifier les sélecteurs avant de livrer un override

Avant de livrer un override CSS/JS/PHP visant du code externe (thème, plugin tiers, framework) :
1. Lire les fichiers du système cible avec grep/Read
2. Confirmer que les classes/sélecteurs/hooks existent exactement dans ce système
3. Pour CSS : ouvrir la feuille de style, vérifier sélecteurs ET propriétés
4. Pour PHP/JS : grep pour confirmer hooks/filtres existent
5. Au minimum : linter (php -l, eslint) avant delivery
6. Ne jamais demander de tester en boucle sans vérification statique préalable

## Pipeline de production éditoriale (sites SEO)

Pour tout contenu éditorial SEO, chaîne d'agents obligatoire :

### Agent 1 — SERP Analysis
- Analyser les 3-4 premiers résultats Google FR
- Extraire : title, meta desc, H1/Hn, longueur, champ sémantique, outils interactifs, schema markup
- Sortie : `docs/serp/{slug}.md`

### Agent 2 — Brief + Rédaction
- Brief (H1/Hn cibles, mots-clés, longueur, angle différenciant)
- Rédaction : voix neutre, evergreen, factuelle
- **Interdit** dans les titres visibles (H2/H3) : "guide complet/ultime/définitif", dates, superlatifs
- **Interdit dans le texte lisible** (H2/H3/paragraphes) : jargon SEO interne ("silo", "maillage interne", "cocon sémantique", "GEO", "listicle", "non-commodity", "page mère", "sub-cluster")

Remplacement acceptable pour sections "see also" :
- "À lire aussi — {Pays}" / "Pour aller plus loin" / "Sur le même thème"

### Agent 3 — Reviewer
- Vérifier TOUTES les données chiffrées, dates, citations, références juridiques
- Sources autorisées : service-public.fr, impots.gouv.fr, Legifrance, Banque de France, ADEME, INSEE, BRGM, Etalab, sites officiels uniquement
- Aucune source secondaire non sourcée
- Grep obligatoire : `grep -rln "silo \|maillage interne\|cocon sémantique" data/`
- Tout match dans texte visible = retour copy

Format de livraison : fichier Markdown par pilier dans `content/piliers/{slug}.md`, prêt Gutenberg.

## Choix de modèle par phase de pipeline

| Phase | Modèle | Raison |
|-------|--------|--------|
| Data / collecte structurée (WebSearch + JSON) | **Haiku** | 5× moins cher, suffisant pour tool use déterministe |
| Copy / rédaction / SEO | **Sonnet** | Qualité éditoriale requise |
| Review / QA indépendant | **Sonnet** | Doit catcher les hallucinations |
| Orchestration / planning | **Sonnet/Opus** | Pas Haiku |

Si Haiku produit des résultats médiocres sur data (rare) : upgrade au cas par cas en Sonnet.

## Sous-agents : contraintes réseau

Les sous-agents lancés en background (`run_in_background: true`) **N'ONT PAS** accès réseau dans ce setup Claude Code :
- Bash (curl, pip install...) → bloqué
- WebFetch, WebSearch → bloqué
- Claude_in_Chrome, puppeteer → bloqué

Pattern correct :
1. Thread principal fait le scrape/fetch via Bash/WebFetch, écrit en local
2. Sous-agents en background lisent les fichiers locaux et catégorisent

Ne jamais lancer des sous-agents en background pour des tâches réseau ou installations de packages.

## Git workflow

- Push après chaque changement significatif (feature, fix, refactor)
- Commits en français sauf projets bilingues, format conventional commits (`feat():`, `fix():`)
- Pas de PR/branches en V1 pour les projets solo (push direct sur main)
- Reconsidérer si plusieurs contributeurs
