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
The New Zealand **road network** dataset obtained from **LINZ** was imported into PostGIS and stored as the primary spatial dataset for subsequent spatial analysis.

**(1) Query the total number of records in the road dataset**
<img width="720" height="293" alt="image" src="https://github.com/user-attachments/assets/f54c6fb2-603e-4b8f-b0e7-5975b6b5cd09" />

**(2) Retrieve 100 Records from the Road Dataset**
<img width="1279" height="761" alt="image" src="https://github.com/user-attachments/assets/39e720fe-0e24-439b-a4af-456cd475c66e" />

**(3) Check the Coordinate Reference System (CRS) of the Road Dataset**

The imported LINZ road network was verified using PostGIS ST_SRID. The dataset was stored using NZGD2000 geographic coordinates (EPSG:4167) and will be transformed to NZTM2000 (EPSG:2193) for spatial analysis requiring distance and length calculations.

<img width="726" height="434" alt="image" src="https://github.com/user-attachments/assets/c31cbab3-e560-411b-ac94-31b15ba8d50f" />


