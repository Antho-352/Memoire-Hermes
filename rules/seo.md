# Règles SEO — Contenu et Stratégie

## Pipeline éditorial obligatoire (tous sites SEO)

Ne jamais rédiger directement. Toujours passer par la chaîne :

**Agent SERP** → **Agent Brief/Rédaction** → **Agent Reviewer**

### Agent SERP
- Analyser 3-4 premiers résultats Google FR pour la requête cible
- WebSearch + WebFetch sur les URLs
- Extraire par concurrent : title, meta description, H1/Hn complète, longueur, champ sémantique, outils interactifs, UX/UI, schema markup
- Produire `docs/serp/{slug}.md`

### Agent Brief/Rédaction
- Brief synthétisé (H1/Hn cibles, mots-clés à couvrir, longueur, angle différenciant)
- Rédiger l'article : voix neutre, evergreen, factuelle

**Interdit dans les titres et texte visible (H2/H3/paragraphes)** :
- Superlatifs : "guide complet", "guide ultime", "définitif", "tout ce qu'il faut savoir"
- Dates dans les titres sauf réforme datée justifiée par la SERP
- Jargon SEO interne : "silo", "maillage interne", "cocon sémantique", "GEO", "listicle", "non-commodity", "page mère", "sub-cluster"
- Expressions interdites à inclure dans tout brief envoyé à un sous-agent

Formulations "À lire aussi" acceptables :
- ✅ "À lire aussi — Suisse", "Pour aller plus loin", "Sur le même thème", "Continuer la lecture"
- ❌ "À lire aussi — silo Alpes", "Maillage interne", "Cocon sémantique"

### Agent Reviewer
Vérifier **toutes** les données chiffrées, dates, citations, références juridiques.

**Sources autorisées uniquement** :
- service-public.fr
- impots.gouv.fr
- Legifrance
- Banque de France
- ADEME
- INSEE
- BRGM (Géorisques)
- Etalab
- Banque des Territoires
- Sites officiels ministériels
- Sites officiels d'organisations internationales

Aucune source secondaire non sourcée. Aucun fait personnel inventé sur Anthony.

**Checklist reviewer obligatoire** :
- [ ] Données chiffrées sourcées et vérifiables
- [ ] Aucun jargon SEO dans le texte lisible : `grep "silo \|maillage\|cocon\|GEO\|listicle" fichier.md`
- [ ] Aucun superlatif interdit
- [ ] Aucune affirmation personnelle sur Anthony inventée

## Format de livraison

Fichier Markdown par pilier dans `content/piliers/{slug}.md` (ou équivalent selon projet). Prêt à être collé dans Gutenberg.

## Politique maillage interne

- Liens de maillage dans le corps de l'article vers piliers connexes
- Ancres naturelles (pas de "cliquez ici", pas de l'URL brute)
- Pas de blocs "Maillage interne" ou "Silo" visibles en front

## Voix éditoriale (tous sites éditoriaux)

- Neutre, factuelle, evergreen
- Pas de date dans les titres (ni année, ni mois) sauf réforme datée justifiée par la SERP
- Pas de superlatifs
- Pas de jargon non expliqué (glose à la première occurrence)
- Sources officielles citées dans le corps du texte (pas en footnote)
- Mention non-conseil sur les sujets fiscaux/juridiques/médicaux/financiers
- Préciser la date de mise à jour de la source quand pertinente (loi, barème, taux)

## Auto-grep avant livraison de contenu

```bash
# Jargon SEO interdit dans texte visible
grep -rln "silo \|maillage interne\|cocon sémantique\|page mère\|sub-cluster\|>Maillage" content/

# Superlatifs interdits
grep -rln "guide ultime\|guide complet\|guide définitif\|tout ce qu'il faut savoir" content/
```

Tout match = retour copy/review, blocage livraison.

## Modèles recommandés par phase SEO

| Phase | Modèle | Justification |
|-------|--------|---------------|
| Collecte SERP (JSON structuré) | Haiku | 5× moins cher, suffisant |
| Rédaction article | Sonnet | Qualité éditoriale |
| Review indépendant | Sonnet | Catch hallucinations |

## Garde-fous juridiques (sites éditoriaux)

- **DGCCRF** (loi 2023-451 influence + Code consommation L.121-1) : transparence affiliation et sponsoring obligatoire. `rel="sponsored nofollow"` sur tous les liens affiliés.
- **CIF/IOBSP/médical/juridique** : aucun titre réglementé revendiqué par le persona rédacteur
- **RGPD** : pas de tracker tiers, newsletter double opt-in, mentions CNIL
- **LCEN** (art. 6.III.1) : éditeur, directeur de publication, hébergeur dans les mentions légales

## Personas rédacteurs

Chaque site éditorial a son persona unique généré via le skill `/persona-redacteur`. Fichier dans `<projet>/docs/persona-redacteur.md`. Inclure dans tout brief envoyé à un agent externe la consigne : "Ne jamais inventer de fait personnel sur l'auteur."
