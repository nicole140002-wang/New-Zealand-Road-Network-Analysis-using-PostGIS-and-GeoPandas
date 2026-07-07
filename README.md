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

1) Count the Total Number of Road Features

The total number of road segments was calculated to understand the dataset size.

```sql
SELECT COUNT(*)
FROM nz_addresses_roads;
```
<img width="425" height="197" alt="image" src="https://github.com/user-attachments/assets/b8e5f70a-c51f-41b5-95b8-260c9e949e58" />

2) Preview Sample Records

```sql
SELECT *
FROM nz_addresses_roads
LIMIT 100;
```
<img width="997" height="611" alt="image" src="https://github.com/user-attachments/assets/eeba9875-5c47-410c-b541-3aa04b8a6c9d" />

3) Check Coordinate Reference System (CRS)


```sql
SELECT ST_SRID(geom)
FROM nz_addresses_roads
LIMIT 5;
```
```text
4167
```

4) Check Geometry Types

The geometry types stored in the road network table were examined to verify the spatial data structure.

<img width="371" height="191" alt="image" src="https://github.com/user-attachments/assets/e275bf4a-0bb2-4ab8-a5de-9ea73bfa0396" />

```sql
SELECT DISTINCT ST_GeometryType(geom)
FROM nz_addresses_roads;
```
```text
ST_MultiLineString
```
5) Spatial Data Preparation
Transform EPSG:4167 → EPSG:2193

```sql
ALTER TABLE nz_addresses_roads
ADD COLUMN geom_2193 geometry(MultiLineString,2193);

UPDATE nz_addresses_roads
SET geom_2193 = ST_Transform(geom,2193);
```

6) Calculate Total Road Network Length

```sql
ALTER TABLE nz_addresses_roads
ADD COLUMN road_length float;

UPDATE nz_addresses_roads
SET road_length = ST_Length(geom::geography);
```
<img width="360" height="395" alt="image" src="https://github.com/user-attachments/assets/745f3bdf-fcfd-4362-ae60-a61d0fce06a8" />

## Step 4. Connect PostGIS with Python
Python was integrated with the PostGIS database using **SQLAlchemy** and GeoPandas. A database connection was established to retrieve the road network table as a GeoDataFrame for further spatial processing and analysis.  

The analysis geometry was transformed to NZTM2000 (EPSG:2193) and stored as geom_2193 for metric-based spatial analysis.

Example workflow:

```python
# Python connection to PostgreSQL/PostGIS database
# Database: git_sample
# Dataset: LINZ New Zealand road network

import geopandas as gpd
from sqlalchemy import create_engine

# Create a connection to the PostgreSQL database
engine = create_engine(
    "postgresql://<username>:<password>@<host>:<port>/<database>"
)

# Read road network data from PostGIS into a GeoDataFrame
roads_gdf = gpd.read_postgis(
    "SELECT * FROM nz_addresses_roads",
    engine,
    geom_col="geom_2193"
)

print(roads_gdf.columns)
```
<img width="866" height="539" alt="image" src="https://github.com/user-attachments/assets/c7ea3f65-d82e-4688-aaf3-167313720eab" />

## Step 5. Exploratory Spatial Analysis Using GeoPandas
After connecting Python with the PostGIS database, GeoPandas was used to explore the road network dataset and understand its spatial characteristics, attribute structure, and data quality.

1) Explore Spatial Extent
The bounding box of the road network was calculated using GeoPandas to understand the geographic coverage of the dataset.

```text
array([1114406.69907759, 4793577.86861259, 2467495.46022585,
       6190127.52043692])
```

2) Calculate Road Segment Length
Road segment lengths were calculated from the projected geometry. The results were stored as a new attribute for further analysis. Summary statistics were calculated:

```text
max 2145095.1329076597
min 8.087625506798597
mean 1561.8530384064586
95th percentile 6420.3725884673295
```

3) Visualize Road Length Distribution

<img width="868" height="545" alt="image" src="https://github.com/user-attachments/assets/cb6d9664-b8c8-45d7-b41c-e1bd783751b7" />

5) Check Missing Values

```python
roads_gdf.geom_2193.isnull().sum()
```
