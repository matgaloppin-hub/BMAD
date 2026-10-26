---
title: POC Géolocalisation GPS — Train Jaune
status: draft
created: 2026-10-26
updated: 2026-10-26
---

# PRD : POC Géolocalisation GPS — Train Jaune

## 0. Document Purpose

Ce PRD s'adresse à l'équipe de développement (Angular/Spring) et à l'équipe sécurité/exploitation du Train Jaune qui statuera sur la suite à donner. Il part du [Product Brief](../briefs/brief-BMAD-2026-10-15/brief.md) finalisé le 15/10/2026 et de son [addendum technique](../briefs/brief-BMAD-2026-10-15/addendum.md), qu'il ne duplique pas — se référer à ces documents pour le contexte sécurité ferroviaire complet (cantonnement téléphonique/radio, précédent de Zoufftgen) et les repères de précision GPS en montagne. Ce PRD est structuré en features avec FR numérotées globalement (FR-1 à FR-N), un glossaire de référence, et des assumptions inline indexées en §9.

## 1. Vision

Un POC applicatif à deux interfaces qui permet aux conducteurs du Train Jaune d'enregistrer leurs trajets réels en géolocalisation continue, et à l'équipe sécurité/exploitation d'analyser ensuite ces trajets pour déterminer, de façon factuelle et gare par gare, si le GPS embarqué (smartphone/tablette) est suffisamment précis et fiable pour détecter qu'un train approche d'une gare, y est entré, puis en est reparti.

Ce POC ne construit pas l'alarme de sécurité finale — il construit la preuve (ou l'infirmation) empirique qui conditionne la décision d'investir dans cette alarme. Sur une ligne à voie unique où l'espacement des trains dépend d'un échange humain (l'ordre de départ), savoir si le GPS seul peut fiabiliser la détection d'un départ de gare est la question qui précède tout le reste.

À l'issue du POC, l'équipe sécurité dispose d'un périmètre géographique calibré empiriquement par gare, d'une cartographie des zones de signal dégradé le long de la ligne, et d'une réponse chiffrée sur la viabilité du GPS seul dans ce relief.

## 2. Target User

### 2.1 Jobs To Be Done

- **Conducteur du Train Jaune** : enregistrer mon trajet en conduisant, sans que cela demande de l'attention au-delà de démarrer/arrêter l'enregistrement.
- **Équipe sécurité/exploitation** : obtenir une réponse factuelle et chiffrée — le GPS est-il assez précis, où, et à quelles conditions — pour décider d'engager (ou non) le développement de l'alarme de sécurité.
- **Équipe technique (future alarme)** : disposer d'un périmètre géographique calibré par gare, réutilisable comme donnée d'entrée pour la conception de l'alarme si le POC est concluant.

### 2.2 Non-Users (v1)

- Les voyageurs du Train Jaune — aucune interface voyageur dans ce POC.
- Le grand public / la presse — le POC reste un outil interne SNCF.

### 2.3 Key User Journeys

- **UJ-1. Un conducteur enregistre un trajet sans y penser.**
  - **Persona + contexte :** un conducteur du Train Jaune, en cabine, sur tablette ou téléphone, avant un trajet régulier.
  - **État d'entrée :** app ouverte en cabine, pas d'authentification individuelle requise (trajets anonymes).
  - **Parcours :** il appuie sur "démarrer l'enregistrement" en début de trajet → l'app capture la position GPS en continu (~1 point/seconde) pendant toute la conduite, y compris en tunnel (buffer local si perte de connexion) → il appuie sur "arrêter l'enregistrement" en fin de trajet.
  - **Climax :** le trajet est confirmé enregistré (feedback visuel simple), sans avoir demandé d'attention supplémentaire pendant la conduite.
  - **Résolution :** le trajet est synchronisé vers le serveur (immédiatement ou dès retour de connexion) et disponible pour analyse.
  - **Cas limite :** si la connexion réseau est coupée (tunnel, zone blanche), les points GPS captés pendant la coupure sont conservés localement et synchronisés automatiquement dès le retour du réseau — le conducteur n'a rien à faire.

- **UJ-2. L'équipe sécurité détermine si une gare est détectable de façon fiable.**
  - **Persona + contexte :** un membre de l'équipe sécurité/exploitation, après qu'une quarantaine de trajets ont été enregistrés, consulte l'interface d'analyse pour une gare précise (ex. Fontpédrouse).
  - **État d'entrée :** interface d'analyse, liste des trajets disponibles.
  - **Parcours :** il sélectionne la gare → il visualise les trajets superposés sur la carte autour de cette gare, avec le tracé officiel de la ligne en référence → il consulte les métriques de précision/dispersion pour cette zone → il observe si les zones de signal dégradé affectent spécifiquement cette gare → il ajuste le seuil de périmètre configurable pour cette gare et observe l'effet sur la distinction des 3 états (approche/présence/départ).
  - **Climax :** il obtient une réponse visuelle et chiffrée claire : "oui, un périmètre de X mètres distingue fiablement les 3 états pour cette gare" ou "non, le signal est trop dégradé ici".
  - **Résolution :** la conclusion par gare s'accumule dans une vue d'ensemble permettant de statuer ligne par ligne sur la viabilité du GPS seul.

## 3. Glossary

- **Trajet** — Un enregistrement continu de positions GPS capturées par un conducteur entre un démarrage et un arrêt d'enregistrement. Associé à une date/heure, anonyme (pas de conducteur identifié).
- **Point GPS** — Une mesure de position (latitude, longitude, horodatage, et métadonnées de précision si disponibles) capturée à la cadence cible d'environ 1 point/seconde pendant un trajet.
- **Gare** — Un arrêt du référentiel de la ligne (gare ou halte à la demande), caractérisé par un nom, un PK (point kilométrique), une position GPS, et un indicateur de gare de croisement.
- **Gare de croisement** — Une gare disposant de plusieurs voies permettant à deux trains de se croiser sur la ligne à voie unique ; c'est à la sortie de ces gares que l'ordre de départ est requis en exploitation réelle (hors scope du POC).
- **Périmètre de gare** — Une zone géographique circulaire configurable (rayon en mètres) centrée sur la position d'une gare, utilisée pour déterminer si un point GPS se trouve en approche, en présence, ou en départ par rapport à cette gare.
- **État de gare** — Un des trois états détectés pour un trajet relativement à une gare donnée : **approche** (le train se dirige vers le périmètre), **présence** (le train est dans le périmètre), **départ** (le train quitte le périmètre). Indépendant du fait que la gare soit ou non une gare de croisement.
- **Seuil de précision/détection** — La valeur configurable (en mètres) du rayon du périmètre de gare, ajustable dans l'interface d'analyse, avec une valeur par défaut proposée de 50 mètres (voir addendum du brief).
- **Tracé officiel de la ligne** — La géométrie géographique de la voie ferrée elle-même (polyligne suivant le rail), distincte du référentiel des gares, utilisée comme référence visuelle pour comparer les trajets enregistrés au tracé réel.
- **Zone de signal dégradé** — Une portion de la ligne où la dispersion des points GPS d'un ou plusieurs trajets dépasse un niveau jugé normal, typiquement aux abords de tunnels ou en vallée encaissée.
- **Interface d'enregistrement** — L'écran conducteur permettant de démarrer/arrêter la capture GPS d'un trajet.
- **Interface d'analyse** — L'écran équipe sécurité/exploitation permettant de visualiser et d'analyser les trajets enregistrés.

## 4. Features

### 4.1 Enregistrement de trajet

**Description :** Interface conducteur, utilisée en cabine sur tablette ou smartphone. Un conducteur démarre un enregistrement en début de trajet ; l'application capture la position GPS en continu à la cadence cible d'environ 1 point/seconde tant que le train se déplace. Le trajet reste anonyme — aucune identification du conducteur n'est demandée ou stockée. En cas de perte de réseau (tunnel, zone blanche), les points GPS captés pendant la coupure sont conservés localement sur l'appareil et synchronisés automatiquement vers le serveur dès le retour de la connexion, sans action du conducteur. Réalise UJ-1.

**Functional Requirements :**

#### FR-1 : Démarrage et arrêt d'enregistrement

Un conducteur peut démarrer puis arrêter un enregistrement de trajet via deux actions explicites dans l'interface. Réalise UJ-1.

**Consequences (testable) :**
- Un seul trajet actif à la fois par appareil ; démarrer un nouveau trajet alors qu'un enregistrement est en cours n'est pas permis sans arrêter le précédent.
- L'arrêt de l'enregistrement clôture le trajet et déclenche la synchronisation (immédiate si réseau disponible).
- L'action de démarrage/arrêt ne nécessite pas plus de 2 interactions (ex. un bouton, une confirmation).

#### FR-2 : Capture continue de la position GPS

Pendant un enregistrement actif, le système capture la position GPS de l'appareil à une cadence cible d'environ 1 point/seconde. Réalise UJ-1.

**Consequences (testable) :**
- Chaque point capturé inclut au minimum : latitude, longitude, horodatage.
- La cadence de capture reste proche de 1 Hz en usage normal (tolérance à définir en test — hors scope du PRD, dépend des capacités GPS de l'appareil cible).
- La capture continue même en arrière-plan de l'application si l'écran de la tablette/téléphone s'éteint ou que l'app perd le focus (contrainte à valider en architecture selon la plateforme — voir §8).

#### FR-3 : Capture offline et synchronisation différée

Lorsque la connexion réseau est indisponible pendant un enregistrement actif, les points GPS captés sont stockés localement sur l'appareil, puis synchronisés automatiquement vers le serveur dès que la connexion redevient disponible. Réalise UJ-1.

**Consequences (testable) :**
- Aucun point GPS capté pendant une coupure réseau n'est perdu, quelle que soit la durée de la coupure, dans la limite du stockage local disponible sur l'appareil.
- La synchronisation différée s'effectue sans action du conducteur.
- Le conducteur n'a pas de retour visuel nécessitant une action pendant une coupure réseau (pas d'alerte bloquante).

**Out of Scope :**
- La résolution de conflits si deux appareils enregistrent simultanément le même trajet physique (un seul appareil par cabine est supposé).

**Feature-specific NFRs :**
- L'interface d'enregistrement doit rester utilisable sans attention soutenue du conducteur pendant la conduite — pas de notification, pas d'écran nécessitant une interaction au-delà du démarrage/arrêt.

**Notes :**
- [NOTE FOR PM] La durée de stockage local maximale avant risque de perte (espace disque de l'appareil) n'est pas chiffrée dans ce PRD — à valider en architecture selon la durée moyenne d'un trajet (62,6 km) et la cadence de capture.

### 4.2 Analyse des trajets

**Description :** Interface équipe sécurité/exploitation et équipe technique, permettant de visualiser les trajets enregistrés sur une carte, de les comparer au tracé officiel de la ligne et au référentiel des gares, de calculer des métriques de précision, d'identifier les zones de signal dégradé, et de déterminer — gare par gare — si les trois états (approche/présence/départ) sont détectables de façon fiable à un seuil de périmètre donné. Réalise UJ-2.

**Functional Requirements :**

#### FR-4 : Liste et sélection des trajets enregistrés

Un utilisateur de l'interface d'analyse peut consulter la liste des trajets enregistrés (horodatage, durée, nombre de points) et en sélectionner un ou plusieurs pour analyse. Réalise UJ-2.

**Consequences (testable) :**
- La liste affiche au minimum la date/heure de début et la durée de chaque trajet.
- La sélection multiple de trajets est possible pour une analyse comparative sur une même zone/gare.

#### FR-5 : Visualisation du trajet sur carte avec tracé officiel et gares

Un utilisateur peut visualiser le tracé GPS d'un ou plusieurs trajets sélectionnés, superposé au tracé officiel de la ligne (géométrie de la voie) et aux positions des gares du référentiel. Réalise UJ-2.

**Consequences (testable) :**
- La carte affiche simultanément : les points/tracé GPS du trajet, le tracé officiel de la ligne, et les marqueurs de gares (nom, position).
- Les gares de croisement (indicateur `mandatory` du référentiel) sont visuellement distinguées des autres gares/haltes sur la carte.
- L'écart visuel entre le trajet enregistré et le tracé officiel est observable à l'œil (pas de calcul de distance chiffré automatique sur cet axe — voir Out of Scope).

**Out of Scope :**
- Le calcul automatique d'un écart chiffré (distance perpendiculaire) entre chaque point GPS et le tracé officiel — cette comparaison reste visuelle dans ce POC. [ASSUMPTION : si une métrique chiffrée d'écart au tracé s'avère nécessaire en cours de POC, elle sera traitée comme un ajout, pas un prérequis v1.]

**Notes :**
- [NOTE FOR PM] Le tracé officiel de la ligne doit être obtenu en amont du développement via OpenStreetMap (way `railway=narrow_gauge`, `name=Ligne de Cerdagne`, ~122 segments couvrant Villefranche↔Latour-de-Carol) — voir addendum pour la méthode d'extraction (Overpass QL) et la nécessité de fusionner/ordonner les segments en une polyligne continue avant usage. Fallbacks si cette source s'avère insuffisante : IGN BD TOPO, SNCF Réseau open data, GTFS TER Occitanie (voir addendum).

#### FR-6 : Métriques de précision par trajet et par zone

Un utilisateur peut consulter des métriques de précision GPS (dispersion des points, écarts constatés) calculées par trajet et par zone géographique le long de la ligne. Réalise UJ-2.

**Consequences (testable) :**
- Les métriques sont consultables au minimum par trajet individuel et par zone (ex. découpage par segment entre gares, ou par gare).
- La dispersion est exprimée dans une unité compréhensible (ex. mètres) par une personne non technique.

#### FR-7 : Identification des zones de signal dégradé

Le système permet d'identifier visuellement, sur la carte, les zones où le signal GPS est le plus dégradé (ex. abords de tunnels, zones encaissées), à partir des métriques de précision calculées. Réalise UJ-2.

**Consequences (testable) :**
- Les zones de signal dégradé sont visuellement distinguées sur la carte (ex. code couleur) en fonction du niveau de dispersion.
- L'identification est exploitable par une personne non technique (équipe sécurité/exploitation), sans lecture de données brutes.

#### FR-8 : Détection des trois états par gare avec seuil configurable

Pour chaque gare du référentiel (croisement ou non), un utilisateur peut déterminer si les trois états (approche / présence en gare / départ) sont distinguables de façon fiable pour un trajet donné, en fonction d'un seuil de périmètre de gare configurable dans l'interface. Réalise UJ-2.

**Consequences (testable) :**
- Le seuil de périmètre est réglable par l'utilisateur dans l'interface, avec une valeur par défaut de 50 mètres au premier chargement.
- Le résultat de la détection des trois états est affiché gare par gare, pas seulement en moyenne sur la ligne.
- Changer le seuil recalcule et met à jour la détection des trois états sans rechargement de page complet.
- Le résultat est consultable aussi bien pour les gares de croisement que pour les gares/haltes simples (le POC traite toutes les gares de façon uniforme).

**Feature-specific NFRs :**
- L'interface d'analyse doit permettre une lecture par une personne non technique (équipe sécurité/exploitation), au-delà des métriques brutes.

## 5. Non-Goals (Explicit)

- Ce POC ne construit pas l'alarme de sécurité (détection temps réel d'un départ sans ordre) — c'est une étape ultérieure conditionnée aux résultats de ce POC.
- Ce POC ne s'intègre à aucun système de signalisation ou de cantonnement existant.
- Ce POC ne distingue pas fonctionnellement les gares de croisement des gares simples dans sa logique applicative — le traitement reste uniforme pour établir une méthode générale, même si l'indicateur de croisement est affiché à titre informatif (FR-5).
- Ce POC ne tranche pas la stratégie définitive de fiabilisation du signal (fusion de capteurs, balises, RTK) si le GPS seul s'avère insuffisant — décision postérieure, informée par les résultats du POC.
- Ce POC ne construit aucune interface voyageur.
- Ce POC ne calcule pas d'écart chiffré automatique entre le tracé enregistré et le tracé officiel (comparaison visuelle uniquement, voir FR-5).

## 6. MVP Scope

### 6.1 In Scope

- Application Angular (front) + Spring (back), déployée pour usage par les conducteurs sur tablette/mobile.
- Interface d'enregistrement : démarrage/arrêt, capture GPS continue (~1 pt/s), capture offline + sync différée, trajets anonymes.
- Interface d'analyse : liste des trajets, visualisation carte (trajet + tracé officiel + gares), métriques de précision par trajet/zone, identification des zones de signal dégradé, détection des 3 états par gare avec seuil configurable (défaut 50 m).
- Design system WCS (SNCF), thème **holding** (`sncf-holding.css`), y compris pour les composants composés (ex. barre de recherche à partir de `wcs-input` + `wcs-dropdown`/`wcs-list-item`, WCS n'ayant pas de composant natif).
- Référentiel des gares du Train Jaune intégré depuis `_bmad/docs/stations.json` (21 gares/haltes avec PK, position, indicateur de croisement).
- Tracé officiel de la ligne intégré depuis OpenStreetMap (voir FR-5, addendum pour méthode d'extraction).
- Environ 50 trajets réels enregistrés au total, sans échéance calendaire contraignante.

### 6.2 Out of Scope for MVP

- L'alarme de sécurité elle-même — différée à une étape ultérieure si le POC est concluant.
- Toute intégration avec les systèmes de signalisation/cantonnement.
- L'identification du conducteur par trajet.
- Le calcul chiffré automatique d'écart entre trajet enregistré et tracé officiel (visuel uniquement).
- La stratégie de fiabilisation du signal (fusion de capteurs, balises, RTK).

## 7. Success Metrics

**Primary**
- **SM-1** : Au moins 50 trajets réels enregistrés sans incident d'usage (perte de données, app bloquante en cabine). Valide FR-1, FR-2, FR-3.
- **SM-2** : Pour chaque gare du référentiel, l'interface d'analyse permet de statuer si les trois états sont détectables avec un périmètre cohérent — gare par gare. Valide FR-8.

**Secondary**
- **SM-3** : Identification claire des zones de signal GPS insuffisamment fiable, exploitable par une personne non technique. Valide FR-7.
- **SM-4** : Un seuil de précision empirique proposé à l'issue du POC (remplaçant la valeur par défaut de 50 m), par gare ou pour la ligne. Valide FR-8.

**Counter-metrics (do not optimize)**
- **SM-C1** : Ne pas optimiser la facilité d'enregistrement (UX du bouton start/stop) au point de masquer des pertes de points GPS réelles — un trajet "simple à enregistrer" mais avec des trous de données non signalés fausserait l'analyse. Contrebalance SM-1.

## 8. Open Questions

1. Hébergement/infrastructure de déploiement (cloud SNCF, on-premise, réseau interne) — non tranché dans ce PRD, à lever en architecture.
2. Durée de stockage local maximale avant risque de perte de données pendant une coupure réseau prolongée — à chiffrer en architecture selon l'espace disque cible des appareils.
3. Comportement exact de la capture GPS en arrière-plan selon la plateforme (iPadOS/Android) — contraintes système à valider.
4. Confirmation du statut de croisement pour les gares à plusieurs voies non confirmées explicitement en source (Font-Romeu, Saillagouse, Bourg-Madame figurent comme `mandatory: true` dans le référentiel fourni — à considérer comme la source de vérité actuelle, supersedant l'incertitude notée dans l'addendum du brief).
5. Fiabilité et fraîcheur du tracé OSM retenu (source cadastrale 2009) — à revalider si des travaux ont modifié le tracé depuis ; fallback IGN BD TOPO si besoin.

## 9. Assumptions Index

- [ASSUMPTION] §4.2 FR-5 — Si une métrique chiffrée d'écart entre trajet et tracé officiel s'avère nécessaire en cours de POC, elle sera traitée comme un ajout ultérieur, pas un prérequis v1.
