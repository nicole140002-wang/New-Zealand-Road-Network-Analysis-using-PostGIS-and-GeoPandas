# New Zealand Road Network Analysis using PostGIS and GeoPandas

## This project demonstrates a complete GIS workflow using:

- PostgreSQL
- PostGIS
- GeoPandas
- Python
- GeoJSON
- QGIS

## Workflow

```mermaid
flowchart TD
A[LINZ Road Network Shapefile] --> B[PostGIS Database]

C[Stats NZ SA1 2025 Boundaries] --> D[PostGIS Database]

B --> E[GeoPandas Processing]
D --> E

E --> F[Spatial Intersection]
F --> G[Road Length Aggregation]

G --> H[Road Density Calculation]

H --> I[QGIS Visualization]
H --> J[GeoJSON Export]
```

## Step 1. Spatial Database Setup 
A Linux-based spatial database environment was established using **VM, PostgreSQL 18, and PostGIS 3.6 (18)**. The PostGIS extension was enabled to support spatial data storage, indexing, and spatial SQL operations. GDAL was installed through **Conda** to provide the **ogr2ogr** data conversion utility. 

## Step 2. Import Road Network and SA1 Data into PostGIS 
The New Zealand road network dataset obtained from LINZ and the Statistical Area 1 (SA1) boundary dataset obtained from Stats NZ were converted from Shapefile format and imported into a PostGIS database as the primary spatial datasets for subsequent spatial analysis.

The data import process was performed using `ogr2ogr`, a GDAL command-line utility designed for vector data format conversion and loading spatial data into spatial databases.

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
<img width="487" height="648" alt="image" src="https://github.com/user-attachments/assets/914916cb-cb07-4702-bebc-34d5771cfc26" />

```bash
ogr2ogr -f "PostgreSQL" \
PG:"host=localhost dbname=git_sample user=wwj_postgis password=123456" \
/home/postgres/statsnz-statistical-area-1-2025-SHP/statistical-area-1-2025.shp \
-nln public.statistical-area-1-2025 \
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

## Step 6. Spatial Overlay and Road Density Analysis
Integrate the road network with Statistical Area 1 (SA1) boundaries to calculate road distribution and road density at the small-area level.
Instead of simply assigning roads to SA1 polygons using a spatial join, this step applies spatial intersection to split road geometries at SA1 boundaries. This ensures that road lengths are accurately allocated to the corresponding statistical areas.  

1) Spatial Intersection between Roads and SA1

```python
roads_sa1_gdf = gpd.overlay(roads_gdf, sa1_gdf, how="intersection")
```
The resulting dataset contains:

- Road segment geometry
- Corresponding SA1 identifier
- Attributes inherited from both datasets

<img width="860" height="207" alt="image" src="https://github.com/user-attachments/assets/d41f8460-c8b3-4109-9d1d-0df3e0ceee6d" />
<img width="908" height="200" alt="image" src="https://github.com/user-attachments/assets/28b8a926-5e9b-4822-8903-4b3f0a705f61" />
<img width="694" height="212" alt="image" src="https://github.com/user-attachments/assets/2fc13114-abc2-431b-9633-6c993805fccb" />

2) Calculate Road Segment Length
Because the data is stored in NZTM2000 projection (EPSG:2193), geometric length calculations are performed in metres.

```python
roads_sa1_gdf["roads_length_intersected"] = roads_sa1_gdf["geometry"].length
```
<img width="202" height="488" alt="image" src="https://github.com/user-attachments/assets/d95ea783-396e-4ec2-ba36-0bcbed27db1a" />

3) Aggregate Road Length by SA1
Road segments were aggregated by SA1 identifier to calculate the total road length within each statistical area.

```python
road_summary = roads_sa1_gdf.groupby("sa12025_v1").agg(
    total_road_length=('roads_length_intersected', 'sum')).reset_index()
```
<img width="307" height="359" alt="image" src="https://github.com/user-attachments/assets/30a58445-769b-4052-a105-63ef5166fa2b" />

4) Calculate Road Density

```python
sa1_gdf['road_density'] = sa1_gdf['total_road_length'] / sa1_gdf['geom'].area
sa1_gdf['road_density']
```

<img width="169" height="227" alt="image" src="https://github.com/user-attachments/assets/9158e02e-22c1-42d4-8c27-455e45ea670a" />
