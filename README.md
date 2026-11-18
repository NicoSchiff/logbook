# Carnet de filtration — Qikiqtarjuaq

Cette application est un carnet de terrain autonome au format HTML pour saisir, prévisualiser, sauvegarder et exporter les données de filtration de la campagne de Qikiqtarjuaq.

## Fichiers

- `index.html` : page principale du carnet.
- `logbook.html` : copie identique de secours ou ancienne copie nommée du carnet.

Ouvrir `index.html` dans un navigateur moderne suffit pour utiliser l’application.

## Fonctionnalités

### Informations générales

- Saisie de la date, de la station, de la campagne, des coordonnées et des informations météorologiques.
- Saisie des informations générales par type d’échantillon de glace.
- Calcul d’un ratio de dilution suggéré.
- Export au format CSV.
- Suppression des enregistrements par clé date-station.

### Épaisseurs glace

- Saisie des épaisseurs de glace, de neige et du freeboard.
- Prévisualisation des mesures par date et station.
- Résumé statistique avec moyenne et écart-type.
- Export du tableau long et du résumé au format CSV.

### Volumes filtrés

- Configuration des types d’échantillons et des profondeurs.
- Ajout de paramètres personnalisés.
- Gestion de réplicats A à E.
- Prévisualisation en format matriciel.
- Export metadata en CSV.
- Option d’export limité à la date sélectionnée.

### Volume saumure

- Saisie du diamètre de carotte.
- Ajout et suppression de sections de carotte.
- Saisie automatique des limites et calcul des hauteurs.
- Saisie de la température et de la salinité de l’eau des trous.
- Calcul en direct :
  - volume de la section ;
  - rapport Vb/V ;
  - volume de saumure ;
  - salinité théorique de la saumure.
- Prévisualisation cumulée par date et station.
- Sélection et suppression de lignes dans la prévisualisation.
- Export de toutes les dates au format CSV.

Les calculs du volume saumure utilisent les équations de Cox et Weeks (1983) et les coefficients thermodynamiques documentés directement dans l’onglet.

## Sauvegarde des données

Les données sont enregistrées automatiquement dans le `localStorage` du navigateur.

Points importants :

- les données restent associées au navigateur utilisé ;
- effacer les données du navigateur peut supprimer les saisies ;
- **Tout effacer** supprime toutes les données enregistrées ;
- les exports CSV permettent de conserver une copie des données hors du navigateur.

## Importation

Le bouton **Importer un CSV** permet d’importer les formats reconnus par l’application :

- metadata ;
- informations générales ;
- épaisseurs de glace au format long ;
- autres formats pris en charge par la fonction d’import.

## CSV

Les exports sont téléchargés directement par le navigateur. Le nom du fichier contient généralement la date ou le type de données exportées.