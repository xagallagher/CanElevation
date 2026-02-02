# CanElevation - AI Coding Agent Instructions

## Project Overview

CanElevation is a bilingual (English/French) documentation repository for Canadian elevation data, including LiDAR point clouds, digital elevation models (DEMs), and vertical datum transformations. The documentation is built with MkDocs Material and uses the `mkdocs-static-i18n` plugin for bilingual support.

**Live documentation:** https://nrcan.github.io/CanElevation/

---

## Repository Structure

```
CanElevation/
├── docs/
│   ├── assets/           # Shared assets (images, sample data, env files)
│   │   ├── env/          # Global conda environment files
│   │   ├── images/       # Documentation images
│   │   ├── sample_data/  # Example datasets
│   │   └── scripts/      # Utility scripts
│   ├── en/               # English documentation
│   │   ├── index.md
│   │   ├── pointclouds/
│   │   ├── stac-dem-mosaics/
│   │   └── vertical-transformations/
│   ├── fr/               # French documentation (mirrors en/)
│   │   ├── index.md
│   │   ├── pointclouds/
│   │   ├── stac-dem-mosaics/
│   │   └── vertical-transformations/
│   └── overrides/        # MkDocs theme customizations
├── scripts/
│   ├── build-docs.bat    # Windows build script
│   ├── build-docs.sh     # Linux/macOS build script
│   └── convert_notebooks.py  # Jupyter → Markdown converter
├── site/                 # Generated static site (git-ignored for deployment)
├── mkdocs.yml            # Main MkDocs configuration
└── mkdocs-local.yml      # Local development configuration
```

---

## Adding New Tutorials

### Step 1: Choose the Tutorial Type

CanElevation supports two types of tutorials:

| Type | Use Case | Files Created |
|------|----------|---------------|
| **Jupyter Notebook** | Interactive Python code with outputs | `.ipynb` + auto-generated `.md` |
| **Markdown Document** | Step-by-step guides (QGIS, ArcGIS Pro, CLI) | `.md` only |

### Step 2: Determine the Category

Place tutorials in the appropriate category folder:

- `pointclouds/` - LiDAR data processing, COPC files, DEM generation
- `stac-dem-mosaics/` - STAC API access, DEM mosaic retrieval
- `vertical-transformations/` - Datum conversions (CGVD28 ↔ CGVD2013), epoch transformations

### Step 3: Create Bilingual Content

**IMPORTANT:** All tutorials must exist in both English and French.

#### File Naming Convention

| Language | Notebook | Markdown |
|----------|----------|----------|
| English | `Tutorial_Name_EN.ipynb` | `tutorial-name.md` |
| French | `Tutorial_Name_FR.ipynb` | `tutorial-name.md` |

#### Location Pattern

```
docs/en/<category>/Tutorial_Name_EN.ipynb
docs/en/<category>/tutorial-name.md
docs/fr/<category>/Tutorial_Name_FR.ipynb
docs/fr/<category>/tutorial-name.md
```

---

## Jupyter Notebook Tutorials

### Creating a New Notebook Tutorial

1. **Create the notebook** in the appropriate language folder:
   ```
   docs/en/pointclouds/My_New_Tutorial_EN.ipynb
   docs/fr/pointclouds/My_New_Tutorial_FR.ipynb
   ```

2. **Register in `scripts/convert_notebooks.py`** - Add entries to the `notebooks` list:
   ```python
   notebooks = [
       # ... existing entries ...
       {
           'source': 'docs/en/pointclouds/My_New_Tutorial_EN.ipynb',
           'output': 'docs/en/pointclouds/my-new-tutorial.md',
           'download': '../My_New_Tutorial_EN.ipynb'
       },
       {
           'source': 'docs/fr/pointclouds/My_New_Tutorial_FR.ipynb',
           'output': 'docs/fr/pointclouds/my-new-tutorial.md',
           'download': '../My_New_Tutorial_FR.ipynb'
       },
   ]
   ```

3. **Add to navigation** in `mkdocs.yml`:
   ```yaml
   nav:
     - Point Clouds:
       - My New Tutorial: pointclouds/my-new-tutorial.md
   ```

4. **Add translation** in `mkdocs.yml` under `plugins.i18n.languages[fr].nav_translations`:
   ```yaml
   nav_translations:
     My New Tutorial: Mon nouveau tutoriel
   ```

### Notebook Content Guidelines

- **First cell:** Title as H1 markdown heading
- **Include setup section:** Reference the appropriate conda environment
- **Use meaningful variable names** in code cells
- **Include output visualizations** (matplotlib, etc.) - they are preserved in conversion
- **Add explanatory markdown** between code cells

### Notebook Header Template

Start each notebook with this pattern:

```markdown
# Tutorial Title

Brief description of what users will learn.

## Prerequisites

- List required knowledge
- Reference to environment setup

## Steps:

1) [Section 1](#1-section-1)
2) [Section 2](#2-section-2)
...
```

---

## Markdown-Only Tutorials

For guides that don't require interactive Python (e.g., QGIS, ArcGIS Pro workflows):

### File Structure

```
docs/en/<category>/my-guide.md
docs/fr/<category>/my-guide.md
```

### Markdown Template

```markdown
# Guide Title

Brief description.

## Prerequisites

- Required software
- Required data

## Step 1: First Action

Instructions with screenshots if applicable.

!!! tip "Pro Tip"
    Use admonitions for tips, warnings, and notes.

## Step 2: Next Action

Continue with clear steps...
```

### Admonition Types

Use MkDocs Material admonitions for callouts:

```markdown
!!! info "Title"
    Information content

!!! warning "Title"
    Warning content

!!! tip "Title"
    Helpful tip

!!! note "Title"
    Additional notes
```

---

## Environment Files

### Global Environment

For general tutorials, use the shared environment:
```
docs/assets/env/environment.yml
```

### Category-Specific Environments

If a tutorial category needs specialized dependencies, create a local environment file:
```
docs/en/<category>/environment.yml
docs/fr/<category>/environment.yml  (copy of EN version)
```

### Environment File Template

```yaml
name: descriptive-env-name
channels:
  - conda-forge
dependencies:
  - python=3.11
  - jupyter
  - notebook
  - ipykernel
  # Add category-specific dependencies below
  - pdal          # For point cloud processing
  - python-pdal
  - rasterio      # For raster operations
  - geopandas
  - matplotlib
```

---

## Navigation Configuration

### mkdocs.yml Structure

The navigation is defined in `mkdocs.yml`:

```yaml
nav:
  - Home: index.md
  - Category Name:
    - Overview: category-folder/index.md
    - Tutorial 1: category-folder/tutorial-1.md
    - Tutorial 2: category-folder/tutorial-2.md
```

### Adding French Translations

Under `plugins.i18n.languages[fr].nav_translations`, add mappings:

```yaml
nav_translations:
  Category Name: Nom de catégorie
  Tutorial 1: Tutoriel 1
  Overview: Aperçu
```

---

## Building Documentation

### Prerequisites

```bash
pip install mkdocs mkdocs-material mkdocs-static-i18n nbconvert nbformat
```

### Build Commands

**Windows:**
```batch
scripts\build-docs.bat           # Build static site
scripts\build-docs.bat serve     # Local development server
```

**Linux/macOS:**
```bash
./scripts/build-docs.sh          # Build static site
./scripts/build-docs.sh serve    # Local development server
```

### Using Local Configuration

For local development with different settings:
```bash
scripts\build-docs.bat -f mkdocs-local.yml serve
```

---

## Common Code Patterns

### STAC API Access

```python
from pystac_client import Client

stac_api_url = "https://datacube.services.geo.ca/stac/api/"
catalog = Client.open(stac_api_url)

# Search for items
search = catalog.search(
    collections=["hrdem-mosaic-2m"],
    bbox=(west, south, east, north),
    max_items=10
)
items = search.item_collection()
```

### PDAL Pipeline (Point Cloud Processing)

```python
import pdal

pipeline = pdal.Pipeline(json.dumps({
    "pipeline": [
        {
            "type": "readers.copc",
            "filename": "input.copc.laz"
        },
        {
            "type": "filters.range",
            "limits": "Classification[2:2]"  # Ground points only
        },
        {
            "type": "writers.gdal",
            "filename": "output.tif",
            "resolution": 1.0,
            "output_type": "idw"
        }
    ]
}))
pipeline.execute()
```

### Vertical Transformation with GDAL

```bash
# CGVD28 to CGVD2013
gdalwarp -s_srs "EPSG:2950+5713" -t_srs "EPSG:2950+6647" input.tif output.tif
```

### Epoch Transformation with PROJ

Use PROJ URN notation for specific epochs:
```
urn:ogc:def:crs:EPSG::4617@2010.0+EPSG::6647
```

---

## Validation Checklist for New Tutorials

Before submitting a new tutorial, verify:

- [ ] **Bilingual:** Both EN and FR versions exist
- [ ] **File naming:** Follows convention (`_EN.ipynb` / `_FR.ipynb`)
- [ ] **Navigation:** Added to `mkdocs.yml` nav section
- [ ] **Translation:** French nav translation added
- [ ] **Notebook conversion:** Registered in `convert_notebooks.py` (if applicable)
- [ ] **Environment:** Dependencies documented or added to environment.yml
- [ ] **Build test:** Documentation builds without errors
- [ ] **Links:** All internal links work correctly
- [ ] **Images:** Stored in `docs/assets/images/` or category `assets/` folder

---

## Key Libraries Reference

| Library | Purpose | Documentation |
|---------|---------|---------------|
| `pdal` / `python-pdal` | Point cloud processing | https://pdal.io |
| `laspy` | LAZ/LAS file I/O | https://laspy.readthedocs.io |
| `rasterio` | Raster I/O and operations | https://rasterio.readthedocs.io |
| `rioxarray` | Xarray + rasterio integration | https://corteva.github.io/rioxarray |
| `pystac-client` | STAC API client | https://pystac-client.readthedocs.io |
| `geopandas` | Geospatial dataframes | https://geopandas.org |
| `pyproj` | CRS transformations | https://pyproj4.github.io/pyproj |

---

## Canadian Reference Systems

### Vertical Datums

| Code | Name | Notes |
|------|------|-------|
| EPSG:5713 | CGVD28 | Historical Canadian datum |
| EPSG:6647 | CGVD2013(CGG2013a) | Current Canadian standard |

### Common Compound CRS Examples

| Description | CRS String |
|-------------|------------|
| NAD83(CSRS) + CGVD28 | `EPSG:4617+5713` |
| NAD83(CSRS) + CGVD2013 | `EPSG:4617+6647` |
| MTM Zone 8 + CGVD2013 | `EPSG:2950+6647` |

---

## Troubleshooting

### Common Issues

**Notebook conversion fails:**
- Check notebook is valid JSON
- Ensure all cells execute without errors
- Verify file path in `convert_notebooks.py`

**Missing translations:**
- Add entry to `nav_translations` in `mkdocs.yml`
- Ensure French markdown file exists in `docs/fr/`

**Build errors:**
- Run `pip install --upgrade mkdocs mkdocs-material mkdocs-static-i18n`
- Check for broken internal links
- Verify all referenced files exist
