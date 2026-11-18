# Carnet de filtration — Qikiqtarjuaq

Application de terrain monopage en HTML pour saisir des informations de campagne, des mesures de glace, des métadonnées de filtration et des estimations de volume de saumure. Les données sont conservées dans le stockage local du navigateur et peuvent être exportées en CSV.

> **Important :** le stockage local n’est pas une sauvegarde. Exportez régulièrement les CSV et conservez-en une copie ailleurs, notamment avant de changer de navigateur ou d’ordinateur.

## Sommaire

- [Démarrage rapide](#démarrage-rapide)
- [Fichiers](#fichiers)
- [Onglets et fonctions](#onglets-et-fonctions)
- [Calculs](#calculs)
- [Exports CSV](#exports-csv)
- [Importer un CSV](#importer-un-csv)
- [Enregistrement et précautions](#enregistrement-et-précautions)
- [Workflow de terrain recommandé](#workflow-de-terrain-recommandé)
- [Dépannage](#dépannage)
- [Mettre à jour le dépôt avec GitHub Desktop](#mettre-à-jour-le-dépôt-avec-github-desktop)
- [Structure et licence](#structure-et-licence)
- [Développement assisté par IA](#développement-assisté-par-IA)

## Démarrage rapide

1. Téléchargez ou clonez le dépôt.
2. Ouvrez `index.html` dans un navigateur récent. Aucun serveur local ni installation n’est nécessaire.
3. Utilisez les onglets et exportez les CSV pour conserver vos données.

Une publication GitHub Pages est plausible : `index.html` est à la racine et l’application principale est en HTML, CSS et JavaScript intégrés. L’adresse prévue est [https://nicoschiff.github.io/logbook/](https://nicoschiff.github.io/logbook/). Pour l’activer, ouvrez **Settings > Pages** dans le dépôt, choisissez la branche `main` et la racine (`/`) comme source, puis enregistrez. La disponibilité effective de l’URL n’a pas pu être vérifiée lors de la rédaction.

## Fichiers

- `index.html` — application principale (interface, calculs, stockage, imports et exports).
- `.gitattributes` — demande à Git de détecter les fichiers texte et de normaliser leurs fins de ligne (`* text=auto`).

## Onglets et fonctions

### Infos générales

Saisissez la date et la station (obligatoires à l’enregistrement), puis, au besoin, la campagne, latitude, longitude, heure d’arrivée, ciel, notes « Cast CTD », opérateur et commentaires. Le type de glace initial est `Sea ice pool`; d’autres types peuvent être ajoutés. Pour chaque type, consignez le ratio de dilution, le volume d’eau de mer filtrée, le volume après fonte et le nombre de carottes regroupées. **Enregistrer les informations** sauvegarde une ligne par type de glace. La table suggère un ratio sans remplacer la valeur saisie.

Les enregistrements généraux sont identifiés par une clé station-date. **Supprimer la sélection** supprime toutes les lignes partageant la clé des lignes sélectionnées.

### Épaisseurs glace

Ajoutez des mesures numérotées d’épaisseur de glace, d’épaisseur de neige ou de freeboard, en centimètres. Le freeboard peut être négatif; glace et neige doivent être positives ou nulles. L’aperçu et le résumé sont filtrés par date et station. Le résumé calcule la moyenne, le nombre de mesures et l’écart-type d’échantillon (dénominateur `n − 1`; laissé vide pour `n = 1`).

### Volumes filtrés

La grille associe paramètres, réplicats (A à E) et profondeurs/types d’échantillon. Les valeurs initiales configurées sont `Water` aux profondeurs `0 m`, `20 m`, `50 m`, et `Ice` à `0-3 cm`. Les colonnes et profondeurs peuvent être modifiées; des types, profondeurs et paramètres personnalisés peuvent être ajoutés. Les paramètres spéciaux (NUT, IFCB, MICRO par défaut) sont indiqués comme échantillonnés, sans volume. Pour enregistrer une ligne, indiquez date et station puis renseignez une cellule de volume positif, ou cochez le paramètre spécial. Les métadonnées apparaissent en format long. L’option d’export permet de prendre toutes les dates ou seulement la date sélectionnée.

### Volume saumure

Saisissez date, station, diamètre de carotte, limites « De/À » des sections, température et salinité de l’eau des trous. Huit sections de 10 cm sont initialisées; les sections peuvent être ajoutées ou supprimées. La hauteur est calculée comme `À − De`. Les résultats sont calculés en direct lorsque les entrées sont valides. L’aperçu rassemble les lignes sauvegardées pour les dates et stations présentes; la suppression depuis cet aperçu retire les lignes sélectionnées de leurs enregistrements.

## Calculs

### Ratio de dilution suggéré

Dans **Infos générales**, le ratio suggéré est calculé seulement si le volume après fonte est supérieur au volume d’eau de mer ajouté :

```text
ratio suggéré = volume après fonte / (volume après fonte − eau de mer ajoutée)
```

La suggestion n’écrase pas le champ `dilution_ratio` saisi.

### Volume de section et saumure (Cox & Weeks, 1983)

Le volume géométrique de section est en cm³, avec diamètre et hauteur en cm :

```text
V = 3,14159 × (diamètre / 2)² × hauteur
ρi = 917 − 0,1403 T
x = (ρi / 1000) × Ssi
Vb/V = x / [F1(T) − x × F2(T)]
Vb = V × (Vb/V)
```

`T` est la température en °C et `Ssi` la salinité saisie. Les polynômes sont `F1(T) = a1 + b1T + c1T² + d1T³` et `F2(T) = a2 + b2T + c2T² + d2T³`. Les coefficients codés sont :

| Intervalle de T | a1 | b1 | c1 | d1 | a2 | b2 | c2 | d2 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `−2 < T ≤ 0` | −0,041221 | −18,407 | 0,58402 | 0,21454 | 0,090312 | −0,016111 | 1,2291×10⁻⁴ | 1,3603×10⁻⁴ |
| `−22,9 ≤ T ≤ −2` | −4,732 | −22,45 | −0,6397 | −0,0174 | 0,08903 | −0,01763 | −5,330×10⁻⁴ | −8,801×10⁻⁶ |
| `−30 ≤ T < −22,9` | 9899 | 1309 | 55,27 | 0,7160 | 8,547 | 1,089 | 0,04518 | 5,819×10⁻⁴ |

Le code retourne un résultat non calculé hors de `−30 ≤ T ≤ 0`. À `T = −2 °C`, le deuxième intervalle s’applique. La salinité théorique de saumure affichée est :

```text
−1,2 − 21,8T − 0,9191T² − 0,0178T³
```

La colonne `Vb/V (%)` affiche le ratio multiplié par 100; `Vb` utilise le ratio non exprimé en pourcentage.

## Exports CSV

Les noms ci-dessous reçoivent la date du jour de l’export au format `AAAAMMJJ` (date locale du navigateur).

| Bouton dans l’interface | Nom téléchargé | Colonnes, dans l’ordre |
|---|---|---|
| **Télécharger le CSV général** | `infos_generales_AAAAMMJJ.csv` | `key`, `date`, `station`, `campaign`, `latitude`, `longitude`, `ice_type`, `arriving_time`, `sky`, `cast_ctd_notes`, `operator`, `comments`, `dilution_ratio`, `filtered_seawater_mL`, `volume_after_melting_mL`, `pooled_cores` |
| **Télécharger CSV long** (Épaisseurs glace) | `epaisseurs_glace_AAAAMMJJ.csv` | `Date`, `Station`, `Mesure`, `Num`, `Valeur_cm`, `Code` |
| **Télécharger résumé CSV** | `resume_epaisseurs_glace_AAAAMMJJ.csv` | `Date`, `Station`, `Mesure`, `Moyenne_cm`, `n`, `sd_cm` |
| **Télécharger metadata CSV** | `metadata_filtration_AAAAMMJJ.csv` | `Date`, `Station`, `Parametre`, `Sample type`, `Depth`, `Echantillonne?`, `Replicat`, `Volume`, `Code` |
| **Télécharger CSV** (Volume saumure) | `volume_saumure_AAAAMMJJ.csv` | `Date`, `Station`, `Section`, `Top_cm`, `Bottom_cm`, `Hauteur_carotte_cm`, `Diametre_cm`, `V_cm3`, `Temperature_C`, `Salinite_psu`, `VbV_pct`, `Vb`, `Brine_salinity` |

Les en-têtes metadata sont exportés sans espaces finaux (`Sample type`, `Depth`); l’import accepte aussi les anciens en-têtes avec espaces finaux. Le nom d’un export décrit le type de données, mais sa date est celle du téléchargement, pas nécessairement celle des mesures. Chaque ligne saumure conserve le diamètre de carotte saisi au moment de son ajout; ses calculs exportés emploient ce diamètre. Les anciennes lignes sans diamètre utilisent la valeur courante du champ.

### Format CSV et ouverture

Le code génère un CSV UTF-8 sans BOM, séparé par des virgules, avec point décimal, guillemets doubles autour des en-têtes et des champs texte (les guillemets internes sont échappés en double). Les champs numériques définis comme tels sont écrits sans guillemets. Les lignes sont terminées par LF (`\n`), y compris sous Windows; aucun séparateur `;` ni virgule décimale n’est généré.

- **Excel en français :** utilisez **Données > À partir d’un fichier texte/CSV**, choisissez la virgule comme délimiteur et vérifiez le point comme séparateur décimal / les paramètres régionaux avant le chargement. Évitez le double-clic si Excel interprète le fichier selon le séparateur régional.
- **R :** `readr::read_csv("nom_du_fichier.csv")` lit le séparateur virgule et le point décimal.

## Importer un CSV

Le bouton **Importer un CSV** accepte un fichier `.csv`/`text/csv`. La reconnaissance se fait à partir des en-têtes, non du nom du fichier. L’import reconnaît :

1. un CSV de métadonnées, avec `Station` facultatif; les en-têtes `Sample type` et `Depth` avec ou sans espace final sont acceptés;
2. un CSV général contenant les 16 colonnes générales;
3. un CSV d’épaisseurs **au format long** (`Date`, `Station`, `Mesure`, `Num`, `Valeur_cm`, `Code`);
4. un CSV saumure, y compris la colonne `Diametre_cm` ajoutée à l’export.

L’import ajoute les lignes aux données existantes et ignore les doublons exacts. Le CSV résumé des épaisseurs est reconnu puis ignoré avec un message : le résumé est recalculé à partir des mesures longues importées. Le parseur accepte le BOM UTF-8 et attend une virgule comme séparateur.

## Enregistrement et précautions

L’application sauvegarde automatiquement son état dans `localStorage` sous la clé `qikiqtarjuaq_logbook_v1`. Cela inclut notamment les données de général, de glace, les métadonnées, la configuration des types/profondeurs, les réplicats et les sections de saumure. Les valeurs de formulaire sont aussi conservées dans cet état. Le stockage est propre au navigateur et à son origine (fichier local ou site web); il ne se synchronise pas entre appareils et peut être effacé par le navigateur.

**Tout effacer** affiche une confirmation, retire cette clé `localStorage`, réinitialise les données enregistrées et remet à zéro les champs, tableaux visibles et messages de statut. Cette action est destructive et ne peut pas être annulée depuis l’application. Les CSV exportés sont les copies de sauvegarde à conserver hors du navigateur. Faites un export régulier et vérifiez qu’il s’ouvre correctement avant d’effacer ou de changer de poste.

## Workflow de terrain recommandé

1. Avant le départ, ouvrir la page sur l’appareil et vérifier le fonctionnement hors connexion, le navigateur utilisé et l’espace de stockage disponible.
2. Au début d’une station, remplir les informations générales avec date, station et coordonnées, puis saisir les données correspondantes dans les autres onglets.
3. Vérifier les unités, les signes (en particulier le freeboard), les profondeurs, réplicats, limites de sections, température et salinité.
4. À intervalles réguliers et en fin de journée, télécharger les exports pertinents. Ranger les fichiers dans un emplacement de sauvegarde distinct de l’appareil.
5. Vérifier quelques lignes/en-têtes des fichiers et ne pas utiliser **Tout effacer** avant d’avoir contrôlé les sauvegardes.

## Dépannage

- **Les données ne réapparaissent pas :** utilisez le même navigateur et la même origine (un fichier ouvert localement et l’URL Pages peuvent avoir des stockages distincts). Vérifiez que le stockage du site n’a pas été vidé.
- **Un CSV ne s’importe pas :** vérifiez la virgule comme séparateur et la présence des noms de colonnes requis. Les formats pris en charge sont décrits ci-dessus.
- **Une valeur de saumure affiche `—` :** vérifiez diamètre, hauteur, température (`−30` à `0 °C`) et salinité; les champs concernés doivent être renseignés et numériques.
- **Consulter les erreurs JavaScript :** ouvrez les outils de développement du navigateur. Sur Mac, le raccourci indiqué est **Cmd+Option+J** (Chrome/Chromium).
- L’application principale ne charge aucune ressource externe et fonctionne hors ligne après ouverture du fichier HTML.

## Mettre à jour le dépôt avec GitHub Desktop

Après avoir modifié des fichiers localement, ouvrez le dépôt dans GitHub Desktop, examinez les changements, saisissez un message de commit, puis créez le commit et utilisez **Push origin** pour l’envoyer vers GitHub. Pour récupérer les changements distants, utilisez **Fetch origin**, puis **Pull origin** si une mise à jour est proposée. Vérifiez la branche avant toute synchronisation.

## Structure et licence

Le dépôt contient l’application autonome (`index.html`) et une copie HTML (`logbook.html`), ainsi que `.gitattributes`. **Licence : à définir.**

## Développement assisté par IA

Ce projet a été développé avec l'aide d'outils d'intelligence artificielle
(« vibe coding »). Une grande partie du code (HTML, CSS, JavaScript) et de
la documentation, dont ce README, a été générée ou modifiée par une IA à
partir de mes instructions, puis relue et testée par moi.

Les besoins scientifiques, les choix de fonctionnement et la validation sur
le terrain sont de mon fait. Les formules utilisées (ratio de dilution,
volume de section, Cox & Weeks 1983) viennent de la littérature et ont été
vérifiées à partir des sources.

Malgré ces vérifications, des erreurs peuvent subsister. Contrôlez vos
données exportées avant de les utiliser pour une analyse ou une
publication. N'hésitez pas à ouvrir une *issue* si vous trouvez un problème.
