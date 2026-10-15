---
title: Addendum — POC géolocalisation GPS — Train Jaune
updated: 2026-10-15
---

# Addendum

Contenu technique et sources de référence qui n'ont pas leur place dans le brief exécutif, mais qui serviront directement à la conception (PRD, architecture) et au développement du POC.

## Contexte sécurité ferroviaire

- Le Train Jaune (ligne de Cerdagne) fonctionne en **cantonnement téléphonique/radio**, sans signalisation automatique de type ETCS : les mouvements de train sont autorisés par échange explicite entre agents/gares.
- L'« ordre de départ » est la confirmation que la voie est libre, condition nécessaire avant qu'un train ne quitte une gare de croisement sur voie unique.
- Précédent documenté : **collision de Zoufftgen** (11 octobre 2006, frontière France-Luxembourg) — un agent de circulation a autorisé un train à franchir un signal fermé alors qu'un train circulait en sens inverse sur l'unique voie ouverte. 6 morts, 16 blessés. Rapport BEA-TT/AET (2009), poursuites pénales. C'est la référence qui illustre la gravité du risque adressé à terme par l'alarme (hors scope du POC).

## Précision GPS en milieu montagneux — repères

- Ciel ouvert, GPS/GNSS standard mono-fréquence : précision ~1-5 m (1-sigma). Multi-constellation : ~1-3 m RMS. Avec SBAS/EGNOS : ~1-2 m. RTK/PPP : submétrique à décimétrique (0,1-0,5 m) — mais nécessite une infrastructure dédiée, hors scope probable du POC initial.
- En tunnel : perte totale de signal (pas de traversée radio de la roche) — nécessite dead-reckoning (dernière position connue + vitesse/direction) pour ne pas perdre le fil pendant la traversée.
- En vallée encaissée / proximité de parois rocheuses : multipath et réduction du nombre de satellites visibles, précision pouvant se dégrader à plusieurs dizaines de mètres, en particulier aux abords immédiats des tunnels.
- Les systèmes ferroviaires de sécurité critique (ETCS niveau 3, PTC américain) ne considèrent jamais le GNSS seul comme suffisant : ils le combinent systématiquement avec IMU/odométrie, balises fixes (Eurobalises, RFID, Wayside devices), et appliquent un principe fail-safe (mode restrictif si la confiance de position est insuffisante). Référence pour une éventuelle itération ultérieure si le GPS seul s'avère limite sur le Train Jaune.
- Aucun POC public documenté trouvé sur une alarme de géolocalisation équivalente sur une ligne touristique/à voie unique française — ce projet serait une première en son genre pour ce cas d'usage précis (sous réserve d'une recherche plus approfondie si besoin, non garantie exhaustive).

## Tracé et gares du Train Jaune

Ligne Villefranche-Vernet-les-Bains → Latour-de-Carol-Enveitg, 62,5 km, voie métrique, ~21-22 arrêts. Gares de croisement **confirmées** par leur description officielle (plusieurs voies dédiées au croisement) : **Fontpédrouse-Saint-Thomas-les-Bains** et **Mont-Louis-La Cabanasse**. D'autres gares à plusieurs voies (Font-Romeu-Odeillo-Via, Saillagouse, Bourg-Madame, Latour-de-Carol) ont probablement une fonction de croisement/dépassement mais cela n'a pas été confirmé explicitement en source — rappel : le POC traite toutes les gares de façon uniforme, cette distinction n'est donc pas bloquante pour démarrer.

**Repères PK/altitude (ordre, de Villefranche à Latour-de-Carol) :**

| PK (km) | Gare | Altitude (m) | Type |
|---|---|---|---|
| 0 | Villefranche-Vernet-les-Bains | 427 | Gare, terminus |
| 4,99 | Serdinya | 526 | Halte à la demande |
| 6,23 | Joncet | 550 | Halte à la demande |
| 9,72 | Olette-Canaveilles | 608 | Gare |
| 11,61 | Nyer | 664 | Halte à la demande |
| 13,86 | Thuès-les-Bains | 747 | Halte à la demande |
| 16,27 | Thuès-Carança | 789 | Halte à la demande |
| 19,70 | Fontpédrouse-Saint-Thomas-les-Bains | 1051 | Gare, **croisement confirmé** |
| 22,73 | Sauto | 1224 | Halte à la demande |
| 25,22 | Planès | 1373 | Halte à la demande |
| 27,87 | Mont-Louis-La Cabanasse | 1511 | Gare, **croisement confirmé** |
| 30,21 | Bolquère-Eyne (point culminant) | 1593 | Gare |
| 34,93 | Font-Romeu-Odeillo-Via | 1534 | Gare, 3 voies (croisement probable, non confirmé) |
| 42,46 | Estavar | 1328 | Halte à la demande |
| 44,67 | Saillagouse | 1302 | Gare, 3 voies |
| 46,61 | Err | 1335 | Halte à la demande |
| 48,39 | Sainte-Léocadie | 1281 | Halte à la demande |
| 52,84 | Osséja | 1241 | Halte à la demande |
| 55,69 | Bourg-Madame | 1143 | Gare, 3 voies |
| 58,71 | Ur-Les Escaldes | 1188 | Halte à la demande |
| 61,38 | Béna-Fanès | — | Halte à la demande |
| 62,56 | Latour-de-Carol-Enveitg | 1232 | Terminus, 6 voies |

**Coordonnées GPS confirmées** (infobox Wikipédia, à revalider via l'API Wikipedia ou Wikidata avant intégration en production) :
- Villefranche-Vernet-les-Bains : 42°35′30″N, 2°22′11″E
- Thuès-Carança : 42°31′25″N, 2°13′39″E
- Fontpédrouse-Saint-Thomas-les-Bains : 42°30′50″N, 2°11′19″E
- Mont-Louis-La Cabanasse : 42,502094°N, 2,113092°E
- Bolquère-Eyne : 42,497948°N, 2,087476°E
- Font-Romeu-Odeillo-Via : 42°29′28″N, 2°02′18″E
- Saillagouse : 42°27′25″N, 2°01′56″E
- Err : 42°26′34″N, 2°01′44″E
- Bourg-Madame : 42°25′57″N, 1°56′55″E
- Latour-de-Carol-Enveitg : 42°27′32″N, 1°54′16″E

**Non confirmées** (ne pas utiliser de valeurs inventées) : Serdinya, Joncet, Olette-Canaveilles, Nyer, Thuès-les-Bains, Sauto, Planès, Estavar, Sainte-Léocadie, Osséja, Ur-Les Escaldes, Béna-Fanès.

**Recommandation pour fiabiliser le référentiel avant développement :** interroger directement l'API Wikipedia (`action=query&prop=coordinates`) ou Wikidata SPARQL sur l'item de la ligne (Q2164826), et/ou exécuter une requête Overpass QL dédiée sur OpenStreetMap (tag `railway=station|halt` le long du tracé `railway=narrow_gauge`) pour obtenir un jeu de coordonnées complet. Croiser si possible avec un flux GTFS TER Occitanie (transport.data.gouv.fr) qui contiendrait `stops.txt` avec `stop_lat`/`stop_lon` officiels — source la plus fiable identifiée, non confirmée disponible pour cette ligne spécifique au moment de la recherche.

## Design system WCS (SNCF)

- **WCS** ("Web Components SNCF") : design system open source (licence MIT) de la SNCF, implémenté en Web Components natifs via Stencil.js.
- Dépôt : gitlab.com/SNCF/wcs (miroir github.com/SNCFdevelopers/wcs). Documentation/Storybook : wcs.dev.sncf.
- Packages npm pertinents : `wcs-core` (composants de base), **`wcs-angular`** (intégration Angular, dernière version constatée 7.8.1), `wcs-react`, `wcs-formly`.
- **Tokens de marque** : le repo expose plusieurs variantes de tokens dans `design-tokens/overrides/` (`sncf-groupe.css`, `sncf-holding.css`, `sncf-reseau.css`, `sncf-voyageurs.css`). La demande d'un "thème Voyageur" correspond concrètement à l'utilisation de la variante **`sncf-voyageurs.css`**. Il n'existe pas de documentation séparée listant ses couleurs précises — elles sont à lire directement dans ce fichier de tokens au moment de l'implémentation.
- Architecture des tokens en deux couches : primitifs (ex. tailles, couleurs de base) et sémantiques (ex. `semantic.border.radius` référençant `{primitive.size.40}`), exposés en **CSS custom properties** plutôt qu'en variables SCSS brutes. Classes sémantiques confirmées : `.wcs-secondary`, `.wcs-success`, `.wcs-warning`, `.wcs-info`, `.wcs-critical`, `.wcs-danger`, `.wcs-dark`, `.wcs-light`.
- **Pas de composant "barre de recherche" natif** dans la bibliothèque (~74-78 composants recensés, aucun nommé search/search-bar). À composer à partir de `wcs-input` + `wcs-dropdown`/`wcs-list-item` lors de l'implémentation.
- Points à vérifier en pratique au démarrage du développement (non confirmés par la recherche documentaire) : procédure exacte d'installation Angular (doc dynamique non lisible par fetch), existence d'un composant de recherche plus récent non listé.

## Seuil de précision — valeur par défaut proposée

Le POC a pour but de déterminer empiriquement le bon seuil de périmètre par gare ; en attendant les premières données réelles, une valeur par défaut raisonnable à exposer dans le réglage configurable : **50 mètres** de rayon autour de chaque gare pour le périmètre "en gare". Ce choix se situe au-dessus de la précision GPS typique en ciel ouvert (1-5 m) pour absorber la dégradation attendue en zone encaissée, sans être si large qu'il engloberait les segments de voie entre gares rapprochées. À ajuster dès les premiers trajets réels analysés.
