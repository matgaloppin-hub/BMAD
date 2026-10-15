---
title: POC géolocalisation GPS — Train Jaune
status: draft
created: 2026-10-15
updated: 2026-10-15
---

# Product Brief : POC Géolocalisation GPS — Train Jaune

## Résumé exécutif

Le Train Jaune (ligne de Cerdagne, Pyrénées-Orientales) circule en voie unique avec gares de croisement, sous régime de cantonnement téléphonique/radio : la sécurité repose sur le fait qu'un train ne reparte d'une gare de croisement qu'après avoir reçu son **ordre de départ**. Un départ sans cet ordre constitue un risque de collision frontale — le précédent de la collision de Zoufftgen (2006, 6 morts) illustre la gravité d'une faille de ce type sur voie unique.

L'exploitant du Train Jaune souhaite, à terme, une alarme capable de détecter qu'un train a quitté une gare de croisement sans ordre de départ. Avant de construire cette alarme, il faut d'abord savoir si la géolocalisation GPS embarquée est **suffisamment précise et fiable**, sur un terrain difficile (montagne, tunnels, virages serrés), pour détecter de façon fiable qu'un train approche d'une gare, y est entré, puis en est reparti.

Ce brief porte sur un **POC d'évaluation GPS**, pas sur l'application finale de sécurité. Il consiste en une application Angular/Spring à deux interfaces : l'une permet aux conducteurs d'enregistrer leurs trajets réels (position GPS collectée en continu), l'autre permet d'analyser ensuite ces trajets — tracé sur carte, précision, zones de signal dégradé — afin de déterminer empiriquement si et comment un périmètre de détection fiable peut être défini autour de chaque gare.

## Le problème

Sur une ligne à voie unique comme le Train Jaune, l'espacement des trains dépend entièrement d'un échange humain : la gare de croisement doit confirmer que la voie est libre avant d'autoriser le départ. Si un conducteur oublie de demander cet ordre, ou part avant de l'avoir reçu, rien dans le système actuel ne le détecte automatiquement — c'est un angle mort de sécurité qui dépend uniquement de la vigilance humaine.

Avant d'investir dans un système d'alarme, l'exploitant a besoin de savoir si une solution de géolocalisation GPS low-cost (smartphone/tablette embarqué) est seulement **envisageable** sur ce terrain. Le relief pyrénéen (tunnels, parois rocheuses, virages serrés, forte dénivellation) est précisément le type d'environnement où le GPS est reconnu pour être peu fiable (perte de signal en tunnel, multipath en vallée encaissée) — c'est pour cette raison que les systèmes ferroviaires de sécurité critiques (ETCS, PTC) ne font jamais confiance au GPS seul. Construire l'alarme sans avoir vérifié cette précision sur le terrain réel du Train Jaune serait prématuré.

## La solution

Un **POC applicatif** à deux interfaces, destiné aux conducteurs du Train Jaune, pour collecter et analyser des données GPS réelles sur la ligne :

**1. Interface d'enregistrement de trajet**
Le conducteur démarre un enregistrement en début de trajet. L'application capture la position GPS en continu (à chaque déplacement du train, cadence cible ~1 point/seconde) et la stocke pour analyse ultérieure. Environ 50 trajets seront enregistrés au total, sur une période non contrainte par un délai fixe.

**2. Interface d'analyse des trajets**
Une fois les trajets enregistrés, cette interface permet de :
- Visualiser le tracé GPS sur une carte, superposé au tracé réel de la ligne et à l'emplacement des gares.
- Calculer des métriques de précision par trajet et par zone (dispersion des points, écarts constatés).
- Identifier visuellement les zones où le signal GPS est le plus dégradé (ex. abords de tunnels, zones encaissées).
- Déterminer, pour chaque gare, si l'on peut distinguer de façon fiable les trois états **approche / présence en gare / départ**, indépendamment du fait que la gare soit ou non une gare de croisement — l'enjeu du POC est justement d'établir un périmètre géographique fiable autour de chaque gare, pas seulement les gares de croisement.
- Configurer le seuil de précision/distance utilisé pour cette détection : un réglage ajustable dans l'interface, avec une valeur par défaut proposée (voir Portée). Le POC a précisément pour but de déterminer empiriquement la bonne valeur — ce n'est pas une donnée connue à l'avance.

## Qui ce POC sert

**Utilisateurs directs : les conducteurs du Train Jaune.** Ils utilisent l'application en conditions réelles de conduite, sur tablette Apple ou téléphone, pour enregistrer leurs trajets sans perturber leur activité. Succès pour eux : une app simple, qui ne demande aucune attention au-delà de démarrer/arrêter l'enregistrement.

**Utilisateurs secondaires : l'équipe sécurité/exploitation de la ligne (côté métier) et l'équipe technique en charge d'une future alarme.** Ils consultent l'interface d'analyse pour statuer sur la viabilité d'une approche GPS avant d'engager le développement de l'alarme de sécurité. Succès pour eux : une réponse factuelle et chiffrée — le GPS est-il assez précis, où, et à quelles conditions.

## Critères de succès

- Au moins ~50 trajets réels enregistrés par les conducteurs sans incident d'usage (perte de données, app bloquante en cabine).
- Pour chaque gare, l'interface d'analyse permet de déterminer si les trois états (approche / en gare / départ) sont détectables avec un périmètre géographique cohérent — gare par gare, pas seulement en moyenne sur la ligne.
- Identification claire des zones où le signal GPS est insuffisamment fiable (ex. abords de tunnels), avec visualisation exploitable par une personne non technique.
- Un seuil de précision empirique proposé à l'issue du POC, remplaçant la valeur par défaut initiale, pour éclairer la décision de poursuivre (ou non, ou autrement) vers l'alarme de sécurité.

## Portée

**Dans le périmètre du POC :**
- Application Angular (front) + Spring (back), déployée pour usage par les conducteurs sur tablette/mobile.
- Interface 1 : démarrage/arrêt d'enregistrement de trajet, capture GPS continue (~1 pt/s), stockage côté serveur.
- Interface 2 : liste des trajets enregistrés, visualisation carte + tracé officiel de la ligne et des gares, métriques de précision, zones de signal faible, réglage configurable du seuil de détection (avec valeur par défaut — voir addendum pour la proposition initiale).
- Identité visuelle : réutilisation du design system **WCS** (SNCF), thème/tokens "Voyageurs", y compris pour les composants d'interface (ex. une barre de recherche composée à partir des briques WCS existantes — ce composant précis n'existe pas nativement dans WCS).
- Référentiel des gares du Train Jaune (nom, position, ordre) intégré pour l'affichage sur carte — détail des sources en addendum.

**Hors périmètre (explicitement) :**
- L'alarme de sécurité elle-même (détection temps réel d'un départ sans ordre) — c'est une étape ultérieure, conditionnée aux résultats de ce POC.
- Toute intégration avec les systèmes de signalisation ou de cantonnement existants.
- La distinction fonctionnelle entre gares de croisement et gares simples dans la logique applicative — le POC traite toutes les gares de la même façon pour établir une méthode générale.
- Le choix définitif de la stratégie de fiabilisation du signal (fusion de capteurs, balises, RTK) si le GPS seul se révèle insuffisant — ce sera une décision postérieure au POC, informée par ses résultats.

## Vision

Si les résultats sont favorables, ce POC devient le socle factuel d'une alarme de sécurité en gare de croisement, détectant un départ sans ordre à partir d'une géolocalisation GPS fiabilisée — et potentiellement, le même mécanisme de périmètres par gare pourrait être réutilisé pour d'autres besoins d'exploitation (suivi de ponctualité, alertes d'arrêt prolongé, etc.). Si les résultats montrent que le GPS seul est insuffisant sur ce relief, le POC aura statué sur ce point en amont — avant tout investissement dans l'alarme — et orientera la réflexion vers une solution hybride (balises, fusion de capteurs) documentée en addendum.
