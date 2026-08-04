<p align="center">
  <img src="assets/logo.svg" width="120" alt="Extracteur Numista">
</p>

<h1 align="center">Extracteur Numista</h1>

<p align="center">
  Outil hors-ligne d'extraction et de consolidation du catalogue numismatique Numista,<br>
  via l'API officielle. Fichier HTML unique, sans dépendance, sans serveur.
</p>

<p align="center">
  <a href="#installation">Installation</a> ·
  <a href="#utilisation">Utilisation</a> ·
  <a href="#gestion-du-quota">Quota</a> ·
  <a href="#format-des-donnees">Format</a>
</p>

---

## Présentation

Cet outil interroge l'**API officielle Numista** pour constituer localement une base
de données numismatique exploitable (Le Médaillier, tableur, archivage personnel).

Il est conçu autour d'une contrainte : le quota de l'API est limité
(2 000 requêtes/mois sur une clé standard). Chaque fonctionnalité vise donc à
**ne jamais dépenser deux fois la même requête**.

### Principes

- **Un seul fichier HTML** — aucune dépendance, aucune installation, aucun build
- **Hors-ligne d'abord** — cache local IndexedDB, tout persiste entre les sessions
- **Incrémental** — ce qui est déjà récupéré n'est jamais redemandé
- **Reprise complète** — l'export JSON est ré-importable, y compris sur une autre machine
- **Interface française**

---

## Installation

### Option A — GitHub Pages (recommandé)

L'API Numista peut refuser les requêtes émises depuis une page ouverte en `file://`
(restriction CORS). Servir le fichier en `https://` règle le problème.

1. Déposer `numista-extracteur.html` à la racine du dépôt
2. *Settings → Pages → Source: Deploy from branch → `main` / root*
3. Ouvrir `https://<utilisateur>.github.io/<depot>/numista-extracteur.html`

### Option B — ouverture locale

Télécharger le fichier et l'ouvrir dans le navigateur. Si le test de clé échoue
avec une erreur réseau/CORS, basculer sur l'option A.

### Clé API

Se procurer une clé sur [en.numista.com/api/api_key.php](https://en.numista.com/api/api_key.php).
Elle est stockée **uniquement dans le navigateur** (`localStorage`) et n'est
transmise qu'à `api.numista.com`.

---

## Utilisation

L'interface se parcourt en cinq étapes, déverrouillées progressivement.

### 1. Configuration & reprise

Saisie de la clé, langue, catégories (pièces / billets / exonumia), budget mensuel.
Le bouton **Enregistrer & tester la clé** valide la connexion par une requête.

Le sélecteur **Reprendre depuis un fichier** importe un JSON existant. La fusion
ne régresse jamais : une pièce déjà enrichie le reste. Si le fichier contient un
registre (`indexed_registry`), il est restauré ; sinon il est **déduit** des
données, de sorte que les émetteurs déjà parcourus ne soient pas réinterrogés.

### 2. Choisir les émetteurs

Arborescence par régions (Europe de l'Ouest, Îles Britanniques, États allemands
historiques, États italiens, Empires & Antiquité, Japon, etc.).

Chaque émetteur affiche un état :

| État | Signification |
|---|---|
| `✓ N pièces` | extrait, présent en cache |
| `à extraire` | jamais récupéré |
| `interrogé · vide` | entrée technique sans pièce propre (ex. une section parente) |

Raccourcis : **Tout l'Europe**, **Japon**, **Ce qui reste** (sélectionne uniquement
le non-couvert).

> Le classement régional est heuristique (basé sur les noms). Le groupe
> **Non classés** rassemble le reste — rien n'est masqué.

### 3. Extraire

Trois modes de ciblage :

| Mode | Usage |
|---|---|
| **Par pays** | extraction complète des émetteurs cochés à l'étape 2 |
| **Par pièce** | recherche par mots-clés, avec pays et année optionnels |
| **Par devise** | une devise à travers tous les pays qui l'utilisent (euro, franc, mark…) |

Chaque exécution rapporte `N nouvelles · M déjà en cache` — c'est le moyen de
**vérifier** qu'une section est complète : relancer, et constater qu'il n'y a
plus de nouveauté.

> **Sur le mode devise :** l'API n'expose pas de filtre « devise » sur la
> recherche (l'information n'existe que dans le détail d'une pièce). Le mode
> s'appuie donc sur le libellé. Très fiable pour l'euro ; à ajuster pour des
> devises au nom variable selon l'époque.

### 4. Enrichir

Récupère le détail complet d'une pièce (composition, poids, diamètre, tirage,
années, images haute définition). **Coût : 1 requête par pièce.**

Un filtre est **obligatoire** — sans lui, l'opération porterait sur la base
entière. Une boîte d'estimation annonce, avant de lancer :

```
508 pièce(s) correspondent · 0 déjà enrichies · 508 à enrichir = autant de requêtes.
✓ Tient dans le quota restant (2000).
```

Préréglages fournis : **2 € commémoratives**, **Toutes les 2 €**,
**Toutes les pièces euro**, **Zone euro depuis 2002**.

L'option *exclure les types courants* écarte les motifs de circulation
(« 1re carte », « 1er type ») pour ne conserver que les commémoratives.

### 5. Exporter

- **JSON** — format natif, ré-importable à l'étape 1 (registre inclus)
- **CSV** — colonnes à plat, séparateur `;`, BOM UTF-8 (Excel-compatible)

L'export ne consomme aucune requête.

---

## Gestion du quota

La barre supérieure suit la consommation du mois en cours, avec remise à zéro
automatique le 1er.

- Index et enrichissement **s'interrompent** avant d'épuiser le budget
- Au lancement suivant, ils **reprennent sur le reliquat**
- Le cache n'est jamais purgé automatiquement

Ordres de grandeur :

| Opération | Coût |
|---|---|
| Liste des émetteurs | 1 requête |
| Index d'un émetteur | 1 requête / 50 pièces |
| Enrichissement | 1 requête / pièce |

---

## Format des données

### Export JSON

```jsonc
{
  "source": "Numista API v3",
  "exported_at": "2026-08-04T00:00:00.000Z",
  "count": 53969,
  "indexed_registry": {           // émetteurs déjà parcourus
    "france": { "cats": ["coin"], "date": 1754300000000 }
  },
  "types": [
    {
      "id": 105,
      "title": "1 centime",
      "category": "coin",
      "issuer": { "code": "allemagne", "name": "Allemagne" },
      "min_year": 2002,
      "max_year": 2026,
      "obverse_thumbnail": "https://…",
      "reverse_thumbnail": "https://…",
      "_detailed": false,          // true si enrichie via /types/{id}
      "_issuerCode": "allemagne",  // code réellement interrogé
      "_medaillier": {             // champs métier conservés à la fusion
        "cle": "de-105-1-centime",
        "numeroKM": "KM# 207",
        "valeurFaciale": "1 cent",
        "axeFrappe": "medaille",
        "lienNumista": "https://fr.numista.com/catalogue/pieces105.html"
      }
    }
  ]
}
```

### Champs internes

| Champ | Rôle |
|---|---|
| `_detailed` | la pièce a été enrichie via `/types/{id}` |
| `_failed` / `_failReason` | échec mémorisé, réessayable séparément |
| `_issuerCode` | code d'émetteur effectivement interrogé (peut différer de `issuer.code`) |
| `_medaillier` | données métier issues d'un catalogue externe, jamais écrasées |
| `_source` | provenance, si autre que l'API |

Les champs préfixés `_` sont propres à l'outil et ignorés par l'API.

---

## Confidentialité

- La clé API ne quitte pas le navigateur (hors appels à `api.numista.com`)
- Aucune télémétrie, aucun serveur tiers, aucun script externe
- Les données restent dans l'IndexedDB locale jusqu'à export ou purge manuelle

---

## Limites connues

- Ouverture en `file://` : l'API peut refuser les requêtes (voir Installation)
- Le classement régional des émetteurs est heuristique
- Le filtre « devise » repose sur le libellé, pas sur un champ structuré
- Le filtre année du mode devise s'applique après réception (n'économise pas de requêtes)
- Le paramètre `object_type` (sous-catégories) attend un code à relever sur la
  recherche avancée du site Numista

---

## Crédits & licence

Données : [Numista](https://fr.numista.com) — consultez leurs
[conditions d'utilisation](https://fr.numista.com/aide/conditions-generales-d-utilisation.html)
avant toute réutilisation, en particulier hors usage personnel.

Cet outil n'est pas affilié à Numista. Il utilise l'API publique dans le respect
du quota associé à votre clé.

Code sous licence MIT — voir [LICENSE](LICENSE).
