# Utilisation avec Python et GDAL

Ce tutoriel présente l'accès et la manipulation de la couche optimisée des bâtiments en format GeoParquet avec **Python (GeoPandas)** et **GDAL (ogr2ogr)**. Ces outils permettent de filtrer, transformer et exporter les données de manière efficace.

!!! info "Prérequis"
    Assurez-vous d'avoir configuré l'environnement conda `env_geoparquet` en suivant le tutoriel [Configuration de l'environnement](environment-setup.md).

!!! warning "Activation de l'environnement"
    Avant de suivre ce tutoriel, activez l'environnement conda avec :
    ```bash
    conda activate env_geoparquet    
    ```


---

## Données utilisées

La couche optimisée des bâtiments est accessible via :

* **URL S3 (pour Python/GeoPandas)** :  
  `s3://ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet`

* **URL /vsis3/ (pour GDAL/ogr2ogr)** :  
  `/vsis3/ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet`


---

## Accès avec Python (GeoPandas)

GeoPandas, combiné avec PyArrow, permet de lire efficacement les données GeoParquet avec filtrage spatial. Le GeoParquet prend en charge la lecture à la demande (lazy-read), ce qui permet de ne lire que les portions nécessaires du fichier. 

Dans une chaine de traitement, nous pouvons uniquement lire en mémoire les emprsies voulues, sans importer au complet la donnée. Très pratique lors de l'automatisation de processus d'analyses. 



### Lecture distante avec filtrage par bbox 

L'exemple suivant permet de:

1. Lire la couche GeoParquet hébergée sur S3. 
2. Appliquer un filtrage spatial lors de la lecture
3. Retourner uniquement les emprsies dans notre filtre spatial. 


Ici, nous allons charger les bâtiments pour un bbox englobant Montréal:

```python
import geopandas as gpd

# URL S3 de la couche optimisée des bâtiments
url = "s3://ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet"

# Définir le bbox
bbox_mtl = (-74.0073, 45.3672, -73.4466, 45.7328)

# Lecture avec pré-filtrage bbox
t0 = time.time()
gpq_subset = gpd.read_parquet(url, bbox=bbox_mtl)
t1 = time.time()

print(f"Lecture S3 + bbox: {t1-t0:.2f} s")
print(f"Nombre de bâtiments: {len(gpq_subset)}")
# Nombre d'emprises récupérées
print(f"Nombre de bâtiments dans la zone : {len(gdf)}")

# Afficher les premières lignes
print(gdf.head())
```

### Affinement avec géométrie polygonale

Dans un context où le bbox utilisé en filtrage est trop grossier, nous pouvons affiner la sélection au besoin. Il s'agit d'un exemple plus près de la réalité. Les étapes suivantes sont faites dans l'exemple: 

1. Lire la couche GeoParquet hébergée sur S3. 
2. Appliquer un filtrage spatial lors de la lecture
3. Retourner uniquement les emprsies dans notre filtre spatial. 
4. Affiner la sélection avec une géométrie voulue plus précise (peut être n'importe quelle géométrie). 



```python
import geopandas as gpd
from shapely.geometry import box

#---|1) Filtrage à la lecture |-------
# URL du GeoParquet sur S3
url = "s3://ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet"

# Définir la bbox (Montréal)
bbox_mtl = (-74.0073, 45.3672, -73.4466, 45.7328)

# Lecture avec pré-filtrage bbox
gpq_subset = gpd.read_parquet(url, bbox=bbox_mtl)
print(f"Nombre de bâtiments: {len(gpq_subset)}")

#---|2.1) Affinage de la sélection |-------
# Définir la limite comme un polygone (bbox)
limite = (-73.565,45.543, -73.521,45.575)
polygon_limite = box(*limite)
gdf_limite = gpd.GeoDataFrame(geometry=[polygon_limite], crs=gpq_subset.crs)

# Effectuer la sélection plus précise. 
gpq_clip = gpd.sjoin(gpq_subset, gdf_limite, predicate="intersects", how="inner")

print(f"Après intersection avec la limite: {len(gpq_clip)} bâtiments")
```

**Résultat attendu :**

![Selection of footprints around Olympic Stadium](../../assets/images/geoparquet_subset_montreal.png)

*Figure : Emprises de bâtiments extraites autour du Stade Olympique de Montréal.*

### Sauvegarde des résultats

Vous pouvez exporter les données filtrées dans différents formats :

```python
# Sauvegarder en GeoPackage
gdf_affine.to_file("batiments_stade_olympique.gpkg", driver="GPKG")

# Sauvegarder en Shapefile
gdf_affine.to_file("batiments_stade_olympique.shp")

# Sauvegarder en GeoJSON
gdf_affine.to_file("batiments_stade_olympique.geojson", driver="GeoJSON")

# Sauvegarder en GeoParquet (pour conserver les performances)
gdf_affine.to_parquet("batiments_stade_olympique.parquet")
```

!!! note "Format de sortie recommandé"
    Le format **GeoParquet** est recommandé pour conserver les avantages de performance. Pour une compatibilité universelle, utilisez **GeoPackage** ou **GeoJSON**.

---

## Accès avec ogr2ogr (GDAL)

GDAL fournit l'outil en ligne de commande `ogr2ogr` pour manipuler les données vectorielles, y compris les fichiers GeoParquet. Les versions récentes supportent le filtrage à la lecture. 

!!! info "Utilisation ogr2ogr"
    L'utilisation de ogr2ogr ne permet pas de sauvegarder le résultats d'une sélection spatiale dans une variable, pour ensuite être réutilisée. Les données doivent être écrites sur le disque ou en utilisant des `/vsimem/`. Pour plus d'information (vsimem) : [Documentation GDAL — Virtual File Systems](https://gdal.org/en/stable/user/virtual_file_systems.html)

### Format d'adresse

Pour accéder aux données distantes avec GDAL, utilisez le préfixe `/vsicurl/` :

```
/vsicurl/ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet
```
### Définir variables environnement

Étant un hébergée sur un répertoire publique, le protocole de connection à AWS ne nécessite pas de compte usager ou de mot de passe. Pour limiter les possibles conflits, il est fortement recomandé de définir ces deux variables environnement avant d'utiliser ogr2ogr:

!!! info "Variables environnement AWS"
    Une fois votre environnement conda activée, vous devez définir ces deux variables:

    **Windows**
    ```powershell
    # Pour Windows PowerShell
    $env:AWS_NO_SIGN_REQUEST = "YES"
    $env:AWS_DEFAULT_REGION = "ca-central-1"

    # Pour Windows cmd 
    set AWS_NO_SIGN_REQUEST = "YES"
    set AWS_DEFAULT_REGION = "ca-central-1"
    ```
    
    **Linux ou MacOS**
    ```
    export AWS_NO_SIGN_REQUEST = "YES"
    export AWS_DEFAULT_REGION = "ca-central-1"
    ```



### Extraction avec filtrage bbox

Nous allons reprendre notre exemple pour filtrer les emprises dans le bbox de Montréal.  Nous allons sauvegarder le résultat en local. La commande sera construite avec les paramètres suivants:

- `Chemin de sortie` : Les emprises finales, filtrées par la géométrie de votre zone d'intérêt (pas le bbox)
- `/vsis3/...` : Chemin du GeoParquet distant sur S3
- `-spat xmin ymin xmax ymax` : Filtre spatial par votre *bounding box*
    - Pour l’exemple de Montréal, les valeurs sont : -74.0073 45.3672 -73.4466 45.7328
- `-nln` : Nom de la couche (layer) dans le fichier de sortie (optionel)

**Windows (PowerShell ou CMD) :**

```bash
# Sur windows, les ^ indiquent la prochaine ligne. Sur linux, il faut remplacer pour \. 
ogr2ogr ^
  Chemin/vers/votre/selection_batiments_ogr2ogr.gpkg ^
  "/vsicurl/ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet" ^
  -spat -74.0073 45.3672 -73.4466 45.7328 ^
  -nln batiement_opti
```

!!! warining "Pour linux et MacOS"
    Pour les utilisateurs des systèmes d'opération linux ou MacOS, raplacez les `^` pas des `\`. Cela indique les sauts de lignes. 


### Extraction avec filtrage par géométrie (clipsrc)

Les options disponibles nous permettent aussi de faire plus d'un traitement dans la même commande. Dans l'exemple qui suit, nous pouvons en une seule commande, filtrer à la lecture les emprises, puis sélectionner uniquement celles qui sont dans une géométrie voulue. Cette géométrie peut être un autre bbox ou encore le chemin vers un fichier polygonal. 

Pour découper les données avec une géométrie polygonale spécifique, utilisez l'option `-clipsrc`. Les paramètrs suivants sont utilisés

- Chemin de sortie : Les emprises finales, filtrées par la géométrie de votre zone d'intérêt (pas le bbox)
- `/vsis3/...` : Chemin du GeoParquet distant sur S3
- `-spat xmin ymin xmax ymax` : Filtre spatial par votre *bounding box*
    - Pour l’exemple de Montréal, les valeurs sont : -74.0073 45.3672 -73.4466 45.7328
- `-clipsrc` : Chemin vers le fichier servant à la découpe les géométries selon la limite fournie (ou un bbox).  
- `-nln` : Nom de la couche (layer) dans le fichier de sortie (optionel)

!!! warning "Utilisation de `-clipsrc`"
    Pour faciler le tutoriel, nous passons les coordonnées d'un bbox pour couper les emprises de la première sélection. Cependant, le paramètres supportes un chemin vers les fichiers qui contiennet des polygones. 

**Windows :**
```bash
# Sur windows, les ^ indiquent la prochaine ligne. Sur linux, il faut remplacer pour \. 
ogr2ogr ^
  Chemin/vers/votre/selection_batiments_ogr2ogr.gpkg ^
  "/vsis3/ftp-maps-canada-ca/pub/nrcan_rncan/extraction/auto_building/auto_building_opti_2/auto_building_opti_2.parquet" ^
  -spat -74.0073 45.3672 -73.4466 45.7328 ^
  -clipsrc -73.565 45.543 -73.521 45.575 ^
  -nln batiement_opti
```

!!! warning "Différence entre -spat et -clipsrc"
    * `-spat` : Sélectionne les géométries dont la bbox intersecte la bbox spécifiée (plus rapide)
    * `-clipsrc` : Découpe réellement les géométries à la bbox spécifiée (plus précis mais plus lent)

## Utilisation en local

Si vous avez téléchargé le fichier GeoParquet localement, vous pouvez l'utiliser de la même manière en remplaçant l'URL S3 par le chemin local :

**Python :**

```python
import geopandas as gpd
from shapely.geometry import box

#---|1) Filtrage à la lecture |-------
# URL du GeoParquet sur S3
url = "chemin/vers/votre/auto_building_opti_2.parquet"

# Définir la bbox (Montréal)
bbox_mtl = (-74.0073, 45.3672, -73.4466, 45.7328)

# Lecture avec pré-filtrage bbox
gpq_subset = gpd.read_parquet(url, bbox=bbox_mtl)
print(f"Nombre de bâtiments: {len(gpq_subset)}")

#---|2.1) Affinage de la sélection |-------
# Définir la limite comme un polygone (bbox)
limite = (-73.565,45.543, -73.521,45.575)
polygon_limite = box(*limite)
gdf_limite = gpd.GeoDataFrame(geometry=[polygon_limite], crs=gpq_subset.crs)

# Effectuer la sélection plus précise. 
gpq_clip = gpd.sjoin(gpq_subset, gdf_limite, predicate="intersects", how="inner")

print(f"Après intersection avec la limite: {len(gpq_clip)} bâtiments")

```

**ogr2ogr :**

```bash

ogr2ogr ^
  Chemin/vers/votre/selection_batiments_ogr2ogr.gpkg ^
  "chemin/vers/votre/auto_building_opti_2.parquet" ^
  -spat -74.0073 45.3672 -73.4466 45.7328 ^
  -clipsrc -73.565 45.543 -73.521 45.575 ^
  -nln batiement_opti
```


---


!!! tip "Meilleures pratiques"
    * **Toujours utiliser un filtrage spatial** (`bbox` ou `-spat`) pour les données volumineuses
    * **Utiliser GeoParquet pour les sorties** si vous comptez réutiliser les données (performances optimales)

---

## Résumé

Ce tutoriel a présenté deux approches pour accéder à la couche optimisée des bâtiments en format GeoParquet :

* **Python (GeoPandas)** : Lecture, filtrage et export avec du code Python flexible
* **GDAL (ogr2ogr)** : Manipulation en ligne de commande pour automatisation et batch processing

!!! tip "Prochaines étapes"
    Si vous préférez une interface graphique, consultez le tutoriel [Utilisation avec QGIS](acces-qgis.md).
