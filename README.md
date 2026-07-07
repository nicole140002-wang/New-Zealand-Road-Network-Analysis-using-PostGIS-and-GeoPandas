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

<img width="763" height="647" alt="image" src="https://github.com/user-attachments/assets/afa1b3d3-0200-42b3-a70d-a317a1667993" />

Example command:
```bash
ogr2ogr -f "PostgreSQL" \
PG:"host=localhost dbname=git_sample user=<username> password=<password>" \
/path/to/nz-addresses-roads.shp \
-nln public.nz_addresses_roads \
-overwrite \
-lco GEOMETRY_NAME=geom \
-nlt PROMOTE_TO_MULTI
```

## Step 3. Exploratory Spatial SQL Analysis
Before performing spatial analysis, the road network dataset was explored using PostGIS SQL queries to understand its structure, attributes, geometry characteristics, and data quality.  

### 1. Count the Total Number of Road Features

The total number of road segments was calculated to understand the dataset size.

```sql
SELECT COUNT(*)
FROM nz_addresses_roads;
```
<img width="425" height="197" alt="image" src="https://github.com/user-attachments/assets/b8e5f70a-c51f-41b5-95b8-260c9e949e58" />

### 2. Preview Sample Records

```sql
SELECT *
FROM nz_addresses_roads
LIMIT 100;
```
<img width="997" height="611" alt="image" src="https://github.com/user-attachments/assets/eeba9875-5c47-410c-b541-3aa04b8a6c9d" />

### 3. Check Coordinate Reference System (CRS)


```sql
SELECT ST_SRID(geom)
FROM nz_addresses_roads
LIMIT 5;
```
```text
4167
```

### 4. Check Geometry Types

The geometry types stored in the road network table were examined to verify the spatial data structure.

<img width="371" height="191" alt="image" src="https://github.com/user-attachments/assets/e275bf4a-0bb2-4ab8-a5de-9ea73bfa0396" />

```sql
SELECT DISTINCT ST_GeometryType(geom)
FROM nz_addresses_roads;
```
```text
ST_MultiLineString
```
### 5. Spatial Data Preparation
Transform EPSG:4167 → EPSG:2193

```sql
ALTER TABLE nz_addresses_roads
ADD COLUMN geom_2193 geometry(MultiLineString,2193);

UPDATE nz_addresses_roads
SET geom_2193 = ST_Transform(geom,2193);
```

### 6. Calculate Total Road Network Length

```sql
ALTER TABLE nz_addresses_roads
ADD COLUMN road_length float;

UPDATE nz_addresses_roads
SET road_length = ST_Length(geom::geography);
```
<img width="360" height="395" alt="image" src="https://github.com/user-attachments/assets/745f3bdf-fcfd-4362-ae60-a61d0fce06a8" />
