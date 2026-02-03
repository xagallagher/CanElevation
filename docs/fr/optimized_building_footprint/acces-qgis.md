# Utilisation avec QGIS

Ce tutoriel présente l'accès et la visualisation de la couche optimisée des bâtiments en format GeoParquet avec **QGIS**. Deux approches sont présentées : le chargement direct du fichier GeoParquet et l'utilisation du plugin **GeoParquet Downloader** pour extraire automatiquement une zone d'intérêt.

!!! info "Prérequis"
   Assurez-vous d'avoir installé QGIS ≥ 3.34 LTR en suivant le tutoriel [Configuration de l'environnement](environment-setup.md#prerequis-qgis).

---

<a id="chargement-direct-du-geoparquet"></a>
## Chargement direct du GeoParquet

QGIS 3.34 et versions ultérieures prennent en charge le format GeoParquet nativement. Vous pouvez charger directement un fichier GeoParquet local ou distant.

### Étapes pour le chargement

1. **Ouvrir le Gestionnaire de sources de données**  
   Allez dans `Couche` → `Gestionnaire de sources de données` ou utilisez le raccourci `Ctrl+L` (Windows/Linux) / `Cmd+L` (macOS).

2. **Sélectionner l'onglet "Vecteur"**  
   Dans le gestionnaire, cliquez sur l'onglet `Vecteur`.

3. **Type de source : "Fichier"**  
   Assurez-vous que le type de source est défini sur `Fichier`.

4. **Parcourir et sélectionner le fichier**  
   Cliquez sur le bouton `...` à côté du champ "Jeu de données vecteur" et naviguez vers votre fichier GeoParquet (`.parquet`).

5. **Ajouter la couche**  
   Cliquez sur `Ajouter` pour charger la couche dans QGIS.

6. **Fermer le gestionnaire**  
   Cliquez sur `Fermer` pour fermer le gestionnaire de sources.

![QGIS Data Source Manager](../../assets/images/geoparquet_qgis_data_source_manager.png)


*Figure : Interface du Gestionnaire de sources de données QGIS pour ajouter un fichier GeoParquet.*

!!! note "Fichiers locaux uniquement"
    Le chargement direct dans QGIS fonctionne principalement avec des fichiers GeoParquet téléchargés localement. Pour accéder à des données distantes (S3, HTTP), utilisez le plugin GeoParquet Downloader décrit ci-dessous.

---

## Plugin GeoParquet Downloader

Le plugin **GeoParquet Downloader** permet de télécharger automatiquement une sous-section du GeoParquet basée sur l'emprise du canevas QGIS. Cela est particulièrement utile pour extraire rapidement une zone d'intérêt sans manipuler le fichier complet.

!!! info "Prérequis du plugin"
    * QGIS ≥ 3.16 (QGIS 3.34+ recommandé)
    * Connexion internet pour télécharger les données distantes

### Installation du plugin

1. **Ouvrir le gestionnaire d'extensions**  
   Allez dans `Extensions` → `Installer/Gérer les extensions`.

2. **Rechercher le plugin**  
   Dans la barre de recherche, tapez `GeoParquet Downloader`.

3. **Installer le plugin**  
   Cliquez sur le plugin dans les résultats, puis cliquez sur `Installer le plugin`.

4. **Vérifier l'installation**  
   Une fois installé, le bouton du plugin devrait apparaître dans la barre d'outils de QGIS.

!!! tip "Lien direct vers le plugin"
    [Page du plugin GeoParquet Downloader](https://plugins.qgis.org/plugins/qgis_plugin_gpq_downloader/)

### Utilisation du plugin

#### Étape 1 : Ajuster le canevas sur la zone d'intérêt

Avant d'utiliser le plugin, **zoomez et centrez le canevas QGIS sur la zone géographique** que vous souhaitez télécharger. Le plugin utilisera l'emprise visible du canevas comme bbox pour le filtrage.

Exemple : Zoomez sur la région de Montréal pour extraire les emprises de bâtiments de cette zone.

#### Étape 2 : Activer le plugin

Cliquez sur le bouton **GeoParquet Downloader** dans la barre d'outils de QGIS :

![Plugin Button](../../images/geoparquet_plugin_button.png)

*Figure : Bouton du plugin GeoParquet Downloader dans la barre d'outils QGIS.*

#### Étape 3 : Configurer l'URL personnalisée

Dans la fenêtre du plugin :

1. **Sélectionner "Custom URL"** dans le menu déroulant
2. **Entrer l'URL du GeoParquet** :

```
https://download-telecharger.services.geo.ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet
```

3. **Cliquer sur "Download"** pour lancer le téléchargement

![Plugin URL Dialog](../../images/geoparquet_plugin_url_dialog.png)

*Figure : Fenêtre de configuration du plugin avec URL personnalisée.*

!!! warning "Emprise du canevas"
    Assurez-vous que le canevas QGIS est bien positionné sur la zone d'intérêt **avant** de cliquer sur "Download". Le plugin utilise l'emprise visible pour filtrer les données côté serveur.

#### Étape 4 : Résultat

Une fois le téléchargement terminé, la couche est automatiquement ajoutée au projet QGIS :

![Plugin Download Result](../../images/geoparquet_plugin_result.png)

*Figure : Emprises de bâtiments téléchargées et affichées dans QGIS via le plugin.*

!!! tip "Avantages du plugin"
    * **Téléchargement partiel** : Seules les données correspondant à l'emprise du canevas sont téléchargées (réduction de la bande passante)
    * **Intégration native** : La couche est directement ajoutée au projet QGIS

---

## Utilisation locale (fichier téléchargé)

Si vous avez téléchargé le fichier GeoParquet complet sur votre ordinateur, vous pouvez l'ouvrir directement dans QGIS :

1. **Télécharger le fichier** :  
   [https://download-telecharger.services.geo.ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet](https://download-telecharger.services.geo.ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet)

2. **Ouvrir dans QGIS** :  
   Suivez les étapes de la section [Chargement direct du GeoParquet](#chargement-direct-du-geoparquet) ci-dessus.

!!! note "Taille du fichier"
    Le fichier complet fait ~2.3 Go. Si vous n'avez besoin que d'une zone géographique spécifique, utilisez plutôt le plugin GeoParquet Downloader ou les outils Python/GDAL pour filtrer les données avant le téléchargement.

---

## Dépannage

### Problème : "GeoParquet format not supported"

**Cause :** Votre version de QGIS est antérieure à 3.34 ou GDAL n'a pas le support GeoParquet activé.

**Solution :**

1. Vérifiez votre version de QGIS : `Aide` → `À propos` (doit être ≥ 3.34)
2. Si la version est inférieure, mettez à jour QGIS vers la dernière LTR : [https://qgis.org/download/](https://qgis.org/download/)

### Problème : Le plugin GeoParquet Downloader ne télécharge aucune donnée

**Causes possibles :**

* L'emprise du canevas QGIS ne couvre aucune donnée (vérifiez que vous êtes bien sur le territoire canadien)
* Problème de connexion internet
* URL incorrecte

**Solutions :**

1. Vérifiez que le canevas QGIS est bien positionné sur une zone géographique couverte par les données (Canada)
2. Vérifiez l'URL dans le plugin (doit être exactement celle fournie ci-dessus)
3. Testez votre connexion internet en ouvrant l'URL dans un navigateur

### Problème : QGIS se fige lors du chargement du fichier complet

**Cause :** Le fichier GeoParquet complet contient 10+ millions d'emprises, ce qui peut ralentir QGIS si toutes les données sont chargées.

**Solution :**

* Utilisez le plugin GeoParquet Downloader pour télécharger uniquement la zone d'intérêt
* Utilisez Python/GDAL pour pré-filtrer les données avant de les charger dans QGIS (voir [Utilisation avec Python et GDAL](acces-python-gdal.md))

---

## Résumé

Ce tutoriel a présenté deux approches pour accéder à la couche optimisée des bâtiments en format GeoParquet dans QGIS :

* **Chargement direct** : Pour les fichiers GeoParquet locaux (≥ QGIS 3.34)
* **Plugin GeoParquet Downloader** : Pour télécharger automatiquement une zone d'intérêt basée sur l'emprise du canevas

!!! tip "Méthode recommandée"
    Pour les utilisateurs QGIS, le **plugin GeoParquet Downloader** est l'approche la plus simple et la plus efficace pour extraire rapidement une zone d'intérêt sans manipuler le fichier complet.
