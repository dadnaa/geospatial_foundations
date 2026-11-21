‎
# Geospatial Foundations
**Geospatial Foundations** is a hands-on geospatial research notebook for comparing common spatial data formats, coordinate reference systems (CRS), distance methods, administrative boundaries, and raster elevation data.
The project accompanies `lab01.ipynb` and uses Algeria and the Gaza Strip as two contrasting geographic study areas. It emphasizes practical correctness: preserving Unicode attributes, validating geometry, selecting local UTM zones, avoiding mixed-CRS calculations, and distinguishing real from synthetic data.
## Research Goal
The goal is to build an end-to-end foundation for reproducible geospatial analysis and GeoAI work:
- represent point and polygon features in GeoJSON;
- validate geometry structure and polygon ring closure;
- inspect and transform geographic and projected coordinate systems;
- calculate local Euclidean and ellipsoidal distances;
- exchange vector data between GeoJSON and GeoPackage;
- inspect real administrative datasets and raster metadata;
- use windowed raster reads and nodata-aware statistics.
## Notebook Coverage
### A. GeoJSON and geometry validation
The notebook creates UTF-8 GeoJSON point datasets for Algerian and Gaza cities, including Arabic names and population or governorate properties. It also implements a geometry validator that checks:
- the root and feature types;
- required coordinate arrays;
- longitude and latitude ranges;
- supported geometry types;
- `GeometryCollection` contents;
- minimum ring size and polygon ring closure.
Valid datasets contain three features. The validator is also exercised against malformed geometry, an open polygon ring, and a nested `GeometryCollection`.
### B. CRS inspection and coordinate transformations
WGS84 (`EPSG:4326`) is a geographic CRS whose coordinates are in degrees. UTM is a projected CRS whose coordinates are in metres and whose accuracy depends on proximity to the local UTM zone.
The notebook computes the appropriate UTM zone from longitude and reprojection EPSG code. This is important for Algeria:
| Area | Example zones |
|---|---|
| Western Algeria | UTM 30N (`EPSG:32630`) |
| Central Algeria | UTM 31N (`EPSG:32631`) |
| Eastern Algeria | UTM 32N (`EPSG:32632`) |
| Gaza | UTM 36N (`EPSG:32636`) |
For pairwise analysis, coordinates are projected into a common UTM zone derived from the pair's mean longitude. UTM Euclidean distances are then compared with WGS84 geodesic distances calculated on the ellipsoid:
- all tested local pair differences were approximately `0.004%-0.022%`;
- the maximum difference was `0.022%`;
- the Algiers–Gaza City great-ellipse distance was `2941.46 km`, with a forward azimuth of `92.11°`.
The cross-region result demonstrates that UTM Euclidean calculations are not appropriate for distant points in different zones. WGS84/ellipsoidal calculations are the appropriate comparison method there.
### C. Vector formats and administrative boundaries
The notebook uses Fiona to download GADM 4.1 administrative boundaries and inspect driver, CRS, bounds, schema, geometry type, and feature properties.
The results show:
| Dataset | Source features | CRS | Converted GeoPackage |
|---|---:|---|---|
| Palestine/Gaza | 2 | `EPSG:4326` | 2 features |
| Algeria | 48 wilayas | `EPSG:4326` | 48 features |
Vector data is converted from GeoJSON to GeoPackage and reopened to verify the target driver, layer name, feature count, CRS, and geometry type.
If GADM is unreachable, the notebook creates a clearly labeled synthetic fallback polygon layer from buffered city points. The fallback is generated only at runtime and is not a substitute for real administrative boundaries.
### D. Raster metadata and elevation analysis
The notebook creates synthetic, `EPSG:4326` GeoTIFF DEMs for the Algiers area and Gaza Strip because real SRTM tiles require Earthdata authentication.
Both rasters are 360 × 360 pixels with a single `float32` band and a nodata value of `-9999`. Their metadata, affine transforms, resolution, bounds, valid-pixel statistics, distributions, and nodata masks are inspected with Rasterio.
The notebook also demonstrates 1×1 window reads and coordinate-to-pixel/pixel-to-coordinate conversions. It uses `PixelIsArea`-consistent GeoTIFF georeferencing and checks real nodata cells rather than relying on a metadata-only demonstration.
## What We Learned
- GeoJSON is easy to inspect and exchange, but real correctness requires explicit geometry and CRS validation.
- A geographic CRS uses angular units and should not be used for planar Euclidean distance calculations.
- UTM zones must be selected for the study extent; forcing one zone across distant or cross-zone features can produce misleading distances.
- UTM-based local distance estimates can closely agree with geodesic distances, while cross-region comparisons require ellipsoidal/geodesic methods.
- Format conversion can introduce different storage and compression tradeoffs: in the notebook, the GeoPackage copies were larger than the source GeoJSON files.
- Raster metadata, resolution, transform, bounds, CRS, band type, and nodata values must be interpreted together.
- Synthetic DEMs are useful for pipeline development, but their outputs must not be presented as real topographic measurements.
- Network-dependent boundary downloads need a reproducible fallback and a visible warning for offline or CI execution.
## Environment
The notebook was developed with Python 3 and the following packages:
```text
fiona
matplotlib
numpy
pyproj
rasterio
shapely
```
Create an isolated environment from the repository root:
```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install jupyter numpy matplotlib pyproj fiona rasterio shapely
jupyter notebook lab01.ipynb
```
The notebook creates a `data/` directory containing generated GeoJSON, plots, GeoPackages, and synthetic DEMs. GADM downloads are cached there; subsequent runs use the local files.
