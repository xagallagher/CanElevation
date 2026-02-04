# Configuration de l'environnement

Ce tutoriel vous guide dans la configuration de l'environnement nécessaire pour accéder et manipuler les données GeoParquet de la couche optimisée des bâtiments. Les étapes varient selon l'outil que vous souhaitez utiliser : **Python/GDAL** ou **QGIS**.

---

## Installation de l'environnement pour Python/GDAL

Cette section est destinée aux utilisateurs qui souhaitent manipuler les données avec **Python (GeoPandas)** ou **GDAL (ogr2ogr)**.

### Prérequis : Installation de Conda

L'environnement Python recommandé utilise **conda** pour gérer les dépendances. Si vous n'avez pas encore installé conda :

1. Téléchargez et installez [Miniconda](https://docs.conda.io/en/latest/miniconda.html) ou [Anaconda](https://www.anaconda.com/products/distribution)
2. Vérifiez l'installation en ouvrant un terminal et en tapant :

```bash
conda --version
```

Si la commande retourne un numéro de version (ex: `conda 24.1.2`), conda est correctement installé.

### Téléchargement du fichier d'environnement

Le fichier d'environnement `environment_geoparquet.yaml` contient toutes les dépendances nécessaires pour manipuler les données GeoParquet avec Python et GDAL.

!!! info "Téléchargement du fichier d'environnement"
    [📄 Télécharger environment_geoparquet.yaml](../../assets/env/environment_geoparquet.yaml){ .md-button .md-button--primary }

Enregistrez ce fichier dans un répertoire de travail sur votre ordinateur.

### Création de l'environnement

Ouvrez un terminal, naviguez vers le répertoire contenant le fichier `environment_geoparquet.yaml`, puis exécutez :

```bash
conda env create -f environment_geoparquet.yaml -n env_geoparquet
```

Cette commande va :
* Installer Python 3.12
* Installer les bibliothèques nécessaires : `geopandas`, `pyarrow`, `shapely`, `pandas`, `numpy`
* Installer GDAL ≥ 3.10 avec support GeoParquet

!!! tip "Temps d'installation"
    L'installation peut prendre quelques minutes selon votre connexion internet. Conda télécharge et installe automatiquement toutes les dépendances requises.

### Activation de l'environnement

Une fois l'environnement créé, activez-le avec :

```bash
conda activate env_geoparquet
```

Votre invite de commande devrait maintenant afficher `(env_geoparquet)` au début de la ligne, indiquant que l'environnement est actif.

### Vérification de l'installation

Pour vérifier que l'installation s'est déroulée correctement, exécutez les commandes suivantes :

**1. Vérifier la version de Python :**
```bash
python --version
```
Attendu : `Python 3.12.x`

**2. Vérifier la présence du driver GeoParquet dans GDAL :**
```bash
ogrinfo --formats | findstr Parquet
```
Attendu : Une ligne contenant `Parquet` (ex: `Parquet -raster,vector- (rw+): (Geo)Parquet`)

**3. Tester les imports Python :**
```bash
python -c "import geopandas; import pyarrow; print('OK - Tous les modules sont correctement installés')"
```
Attendu : `OK - Tous les modules sont correctement installés`

!!! warning "Résolution de problèmes"
    En cas de problème persistant, supprimez l'environnement avec `conda env remove -n env_geoparquet` et recommencez la création.
    Vous pouvez également consulter la documentation de GDAL détaillant la création d'un environnement supportant le GeoParquet [https://gdal.org/en/stable/tutorials/vector_geoparquet_tut.html](https://gdal.org/en/stable/tutorials/vector_geoparquet_tut.html)
---

<a id="prerequis-qgis"></a>
## Prérequis QGIS

Cette section est destinée aux utilisateurs qui souhaitent visualiser et manipuler les données avec **QGIS**.

### Installation de QGIS

Pour accéder aux données GeoParquet dans QGIS, nous suggérons d'utiliser la version suivante :

* **QGIS 3.40 LTR** (Long Term Release) : La version stable avec les dépendances nécessaires. 

!!! info "Téléchargement de QGIS"
    Téléchargez et installez QGIS depuis le site officiel : [https://qgis.org/download/](https://qgis.org/download/)

**Vérification de la version :**

1. Ouvrez QGIS
2. Allez dans le menu `Aide` → `À propos`
3. Vérifiez que la version affichée est ≥ 3.40


## Résumé

Vous avez maintenant configuré votre environnement pour accéder aux données GeoParquet :

* **Python/GDAL** : Environnement conda `env_geoparquet` avec GeoPandas, PyArrow et GDAL ≥ 3.10
* **QGIS** : QGIS ≥ 3.40 LTR avec support natif GeoParquet 

!!! tip "Prochaines étapes"
    Consultez les tutoriels d'utilisation :
    [Utilisation avec Python ou GDAL](acces-python-gdal.md)
    [Utilisation avec QGIS](acces-qgis.md)
