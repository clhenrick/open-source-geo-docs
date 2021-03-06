OGR SQL queries
===============

###resources:
*  http://www.gdal.org/ogr/ogr_sql.html
*  https://github.com/nvkelso/geo-how-to/wiki/OGR-to-reproject,-modify-Shapefiles

*note: use ogrinfo to get stats and ogr2ogr to create a new shapefile.

###return all values for a field
ogrinfo -sql "SELECT DISTINCT field_name FROM polylayer" polylayer.shp

###this command queries all admin 0 capitals (country capitals) and creates a new shapefile called 'ne_50m_pop_place_admin0_cap.shp':
  ogr2ogr -sql "SELECT * FROM ne_50m_populated_places_simple WHERE featurecla='Admin-0 capital'"  ne_50m_pop_place_admin0_cap.shp ne_50m_populated_places_simple.shp

###similar as above but returns a new shapefile only of admin 1 capitals 
  ogr2ogr -sql "SELECT * FROM ne_50m_populated_places_simple WHERE featurecla IN ('Admin-1 capital', 'Admin-1 region capital')"  ne_50m_pop_place_admin1_cap.shp ne_50m_populated_places_simple.shp

###this query returns each value in the featurecla field with no duplicates and does not create a new shapefile:
  ogrinfo -sql "select DISTINCT featurecla from ne_50m_populated_places_simple"  ne_50m_populated_places_simple.shp ne_50m_populated_places_simple
  
###this query returns the number of values in ADMO_A3 without duplicate values:
  ogrinfo -sql "select COUNT(DISTINCT ADM0_A3) from ne_50m_populated_places_simple"  ne_50m_populated_places_simple.shp ne_50m_populated_places_simple
  

Working with OSM shapefile extract data
---------

###return the attributes in the type field and order them alphabetically:
ogrinfo -sql "select distinct type from roads order by type" roads.shp roads


##The following commands separate an OSM extract roads.shp file containing every type of OSM road tag into different classes to make styling roads easier in Mapublisher:

* To Do: create a bash shell script that runs all of these commands.

* note: being able to perform the 'group by' function in ogr sql would be helpful for joining road geometry based on field names,
         ie: ogr2ogr -sql "select * from roads group by ('name', 'type', 'ref')" roads_grouped.shp roads.shp 
         but ogr sql does not support the 'group by' SQL aggregation command.
         apparently this is possible in postgresql with postGIS, then one could export the table result to a .shp format.
  
###select all freeways from a osm roads.shp and create a new .shp file:
ogr2ogr -sql "select * from roads where type in ('motorway', 'trunk')"  superhwy_osm.shp roads.shp
    
###same as above and project to EPSG:2274 (Tennessee State Plane, feet):
ogr2ogr -sql "select * from roads where type in ('motorway', 'trunk')" -t_srs EPSG:2274 superhwy_osm_2274.shp roads.shp

###select freeway ramps / links:
ogr2ogr -sql "select * from roads where type in ('motorway_link', 'trunk_link')"  superhwy_links_osm.shp roads.shp
ogr2ogr -sql "select * from roads where type in ('motorway_link', 'trunk_link')"  -t_srs EPSG:2274 superhwy_links_osm_2274.shp roads.shp

###select all main roads... :
ogr2ogr -sql "select * from roads where type in ('primary','secondary','tertiary')"  main-rd_osm.shp roads.shp
ogr2ogr -sql "select * from roads where type in ('primary','secondary','tertiary')" -t_srs EPSG:2274 main-rd_osm_2274.shp roads.shp

###select all other types of road links (to keep if needed at larger scale maps):
ogr2ogr -sql "select * from roads where type in ('primary_link','secondary_link','tertiary_link')"  main-rd_links_osm.shp roads.shp 
ogr2ogr -sql "select * from roads where type in ('primary_link','secondary_link','tertiary_link')" -t_srs EPSG:2274 main-rd_links_osm_2274.shp roads.shp

###or use the like operator:
   ogr2ogr -sql "select * from roads where type like '%link' and type != 'motorway_link'" main-rd_links_osm.shp roads.shp 

###select all local roads:
ogr2ogr -sql "select * from roads where type in ('residential', 'service', 'living_street', 'unclassified')"  other-rd_osm.shp roads.shp 
ogr2ogr -sql "select * from roads where type in ('residential', 'service', 'living_street', 'unclassified')" -t_srs EPSG:2274 other-rd_osm_2274.shp roads.shp 

###select all dirt roads:
ogr2ogr -sql "select * from roads where type = 'track'"  dirt-rd_osm.shp roads.shp
ogr2ogr -sql "select * from roads where type = 'track'" -t_srs EPSG:2274 dirt-rd_osm_2274.shp roads.shp 

###select all pedestrian roads / paths / cycleways / etc:
ogr2ogr -sql "select * from roads where type in ('bridleway', 'cycleway', 'footway', 'path', 'pedestrian', 'steps')"  ped-trail_osm.shp roads.shp 
ogr2ogr -sql "select * from roads where type in ('bridleway', 'cycleway', 'footway', 'path', 'pedestrian', 'steps')" -t_srs EPSG:2274 ped-trail_osm_2274.shp roads.shp 

------------

###querying geojson data after skeletroning:
ogr2ogr -sql "select * from OGRGeoJSON where highway in ('motorway', 'trunk')" -f "ESRI Shapefile" -t_srs EPSG:2274 superhwy_osm_gen_z14_w13_2274.shp nashville_z14_w13.json
ogr2ogr -sql "select * from OGRGeoJSON where highway in ('primary','secondary')" -f "ESRI Shapefile" -t_srs EPSG:2274 main-rd_osm_gen_z14_w13_2274.shp nashville_z14_w13.json


*************************************
##OSM natural features

###list type attributes:
ogrinfo -sql "select distinct type from natural order by type" natural.shp natural

###create a water polygon layer from natural.shp:
ogr2ogr -sql "select * from natural where type in ('riverbank', 'water')" -t_srs EPSG:2274 water-poly_osm_2274.shp natural.shp 

###create a parks polygon layer from natural.shp:
ogr2ogr -sql "select * from natural where type = 'park'" -t_srs EPSG:2274 parks_osm_2274.shp natural.shp 

#Flickr shapes
* to do: how to query out a substring and create a new field from it?

###query out all neighborhoods in Nashville, TN, US
ogr2ogr -sql "select label as name from OGRGeoJSON where (label LIKE '%Nashville, TN, US%')" -f "ESRI Shapefile" -t_srs EPSG:2274 hoods_flickr_nv_test_2274.shp flickr_shapes_neighbourhoods.geojson 
