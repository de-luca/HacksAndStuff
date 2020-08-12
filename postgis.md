# PostGIS

## gotchas
- Points are x/y => lon/lat

## shp files
- your shapefile will be projected on a coordinate system
- you need to know the projection of your shp file
- projection can be directly the SRID or some "esotheric" name, then you need to look up the SRID using at epsg.io
- projections are stored in table `spatial_ref_sys`

## lat/lon point storage
- use geography if most of the wor will be using lat/lon values

