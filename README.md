# New Zealand Road Network Analysis using PostGIS and GeoPandas

## This project demonstrates a complete GIS workflow using:

- PostgreSQL
- PostGIS
- GeoPandas
- Python
- GeoJSON
- QGIS

## Step 1. Spatial Database Setup 
A Linux-based spatial database environment was established using **VM, PostgreSQL 18, and PostGIS 3.6 (18)**. The PostGIS extension was enabled to support spatial data storage, indexing, and spatial SQL operations. GDAL was installed through **Conda** to provide the **ogr2ogr** data conversion utility. 

## Step 2. Import Road Network Data into PostGIS 
The New Zealand **road network** dataset obtained from **LINZ** was converted and imported into the PostGIS database as the primary spatial dataset for subsequent spatial analysis.
The data import process was performed using `ogr2ogr`, a GDAL command-line utility for spatial data format conversion and database loading.
Example command:
```bash
ogr2ogr -f "PostgreSQL" \
PG:"host=localhost dbname=git_sample user=<username> password=<password>" \
/path/to/nz-addresses-roads.shp \
-nln public.nz_addresses_roads \
-overwrite \
-lco GEOMETRY_NAME=geom \
-nlt PROMOTE_TO_MULTI


## Step 3.
