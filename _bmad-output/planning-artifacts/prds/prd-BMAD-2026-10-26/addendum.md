---
title: Addendum — PRD POC Géolocalisation GPS — Train Jaune
updated: 2026-10-26
---

# Addendum

Contenu technique qui n'a pas sa place dans le PRD exécutif, mais sert directement à l'architecture et au développement. Complète (sans dupliquer) l'[addendum du Product Brief](../briefs/brief-BMAD-2026-10-15/addendum.md), qui reste valide pour le contexte sécurité ferroviaire, les repères de précision GPS, et le design system WCS — à l'exception de la correction de thème ci-dessous.

## Correction — Thème WCS

L'addendum du brief mentionnait le thème **"Voyageurs"** (`sncf-voyageurs.css`) comme hypothèse de travail. Confirmé par l'utilisateur : le thème à utiliser est **"Holding"** (`sncf-holding.css`), disponible dans `design-tokens/overrides/` du repo WCS (gitlab.com/SNCF/wcs, miroir github.com/SNCFdevelopers/wcs). Comme pour la variante Voyageurs, il n'existe pas de documentation séparée listant précisément les couleurs/tokens de cette variante — à lire directement dans ce fichier de tokens au moment de l'implémentation. Référence doc/Storybook : wcs.dev.sncf.

## Référentiel des gares — source de vérité

Le référentiel `_bmad/docs/stations.json` (fourni par l'utilisateur) remplace et affine la table de gares de l'addendum du brief. 21 gares/haltes, de Villefranche-Vernet-les-Bains (PK 0) à Latour-de-Carol-Enveitg (PK 62,6), avec coordonnées GPS et indicateur `mandatory` (gare de croisement) pour chacune. Gares de croisement confirmées par ce référentiel (`mandatory: true`) :

- Villefranche-Vernet-Les-Bains
- Olette-Canaveilles-Les-Bains
- Fontpédrouse - Saint-Thomas-Les-Bains
- Mont-Louis-La-Cabanasse
- Font-Romeu-Odeillo-Via
- Saillagouse
- Bourg-Madame
- Latour-de-Carol-Enveitg

Ceci résout l'incertitude notée dans l'addendum du brief sur Font-Romeu, Saillagouse et Bourg-Madame (alors "probable, non confirmé") : le référentiel fourni les traite désormais comme croisement confirmé. Si une divergence est constatée en développement, traiter `stations.json` comme source de vérité pour ce PRD.

## Tracé officiel de la ligne — obtention et limites

**Source recommandée : OpenStreetMap, via Overpass API.**

- Couverture confirmée : ~122 segments (`way`) tagués `railway=narrow_gauge`, `name=Ligne de Cerdagne` (variante catalane "Línia de Cerdanya"), couvrant toute la ligne Villefranche↔Latour-de-Carol dans la bbox 42.40–42.65°N, 1.85–2.40°E. Un segment porte le nom usuel "Le Petit Train Jaune de la Cerdagne" / "Tren Groc". Tags associés : `gauge=1000`, `electrified=rail`, `voltage=850`, `usage=tourism`.
- Source de la donnée OSM elle-même : cadastre DGI-FR, mise à jour 2009 — précision cadastrale (pas GPS), donc fiable comme référence géométrique, mais potentiellement désynchronisée si des travaux ont modifié le tracé depuis 2009.

**Méthode d'extraction (Overpass QL, format GeoJSON) :**

```
https://overpass-api.de/api/interpreter?data=[out:json];way[railway=narrow_gauge][name~"Train Jaune|Cerdagne"](42.40,1.85,42.65,2.40);out geom;
```

`out geom;` renvoie la géométrie complète des segments (au lieu de `out tags;`). Alternative plus simple au développement : export direct via overpass-turbo.eu (export GeoJSON en un clic), ou conversion en GPX/KML via JOSM.

**Limitation importante à anticiper en développement :** la géométrie OSM renvoyée est une collection de ~122 segments disjoints, pas une polyligne unique prête à l'emploi. Un travail de fusion/ordonnancement des segments (par continuité géographique ou par PK) est nécessaire avant de pouvoir l'utiliser comme référence de comparaison visuelle (FR-5) ou de calcul de dispersion par zone (FR-6, FR-7). Aucune relation OSM unifiée groupant tous les ways n'a été trouvée lors de la recherche.

**Sources de fallback, par ordre de fiabilité si OSM s'avère insuffisant :**

1. **IGN Géoportail** — couches BD TOPO (voies ferrées), WFS/GeoJSON, précision IGN, généralement complet pour le réseau ferré national. Non testé en détail pour cette ligne spécifique.
2. **SNCF Réseau open data** (ressources.data.sncf.com) — dataset "Formes des lignes du réseau ferré national" évoqué dans la littérature mais **non confirmé accessible** lors de la recherche (page d'exploration n'a pas renvoyé de contenu exploitable) — à vérifier manuellement avant de l'écarter.
3. **GTFS TER Occitanie** (transport.data.gouv.fr) — `shapes.txt` donnerait une géométrie suivant la voie, mais recherche infructueuse lors du test ("0 résultat") ; probablement un nom de dataset différent à chercher manuellement. Moins fiable qu'OSM/IGN pour une géométrie fine (géométrie souvent interpolée entre positions d'arrêts).

## Comparaison trajet enregistré ↔ tracé officiel — portée confirmée

Confirmé avec l'utilisateur : cette comparaison est **visuelle uniquement** (FR-5) — superposer le trajet GPS enregistré au tracé officiel sur la carte, pour que l'écart soit observable à l'œil. Ce n'est pas la même chose que les métriques de précision/dispersion par trajet et par zone (FR-6), qui restent, elles, chiffrées. Si un calcul chiffré d'écart au tracé officiel (ex. distance perpendiculaire point-GPS → voie) s'avère nécessaire en cours de POC, le traiter comme un ajout ultérieur (voir Assumptions Index du PRD), pas un prérequis v1.
