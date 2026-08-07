# Journal des modifications

Le format suit [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/).

## [2.3.0] — 2026-08-07

### Ajouté
- Export **Le Médaillier** : reformate les données aux champs de la fiche pièce
  (métal, titre, poids, diamètre, tranche, axe de frappe, monnaie, référence…)
- Analyse de `composition` → **métal normalisé + titre en millièmes**
  (gère « Or (.900) », « Argent 900‰ », « 22 carats », « Bimétallique… »)
- Option **tirage & atelier** à l'enrichissement, via `/types/{id}/issues`
  (coût doublé, annoncé dans l'estimation)
- Alerte au moment de l'export sur le nombre de pièces sans métal identifiable

### Corrigé
- Le métal n'est plus laissé au défaut du formulaire : il est déduit de la
  composition réelle renvoyée par l'API

## [2.2.0] — 2026-08-04

### Ajouté
- Enrichissement **ciblé** : filtres titre / pays / plage d'années, avec
  estimation du coût en requêtes avant lancement
- Préréglages d'enrichissement : 2 € commémoratives, toutes les 2 €,
  toutes les pièces euro, zone euro depuis 2002
- Option d'exclusion des types courants (« 1re carte », « 1er type »)
- Conservation des champs métier `_medaillier` lors de l'enrichissement
- Colonnes supplémentaires dans l'export CSV (KM#, valeur faciale, clé, lien)

### Modifié
- L'enrichissement **exige** désormais un filtre (garde-fou anti-épuisement du quota)

## [2.1.0] — 2026-08-04

### Ajouté
- Trois modes d'extraction : par pays, par pièce (recherche), par devise
- Prise en charge du paramètre `object_type` (sous-catégories)
- Rapport « nouvelles / déjà en cache » après chaque extraction

## [2.0.0] — 2026-07-15

### Ajouté
- Import/fusion d'un JSON existant, sans régression des données
- Reconstruction du registre des émetteurs interrogés depuis les données
- `indexed_registry` embarqué dans l'export
- Arborescence régionale des émetteurs avec état par émetteur
- Réessai séparé des échecs

### Corrigé
- Affichage contradictoire des états d'émetteur
- Import lent sur gros fichiers (écriture par lots)

## [1.0.0] — 2026-07-14

- Version initiale : configuration, sélection d'émetteurs, index, enrichissement, export
