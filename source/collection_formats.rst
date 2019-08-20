.. _ref_collection_formats:

Image Collection Formats
==================================================

Image collection formats describe how image collections are composed from GDAL datasets, considering specific data product organizations.

A collection format is a JSON file defining rules, how to derive required metadata when a GDAL dataset 
(specified by its identifier, e.g. the path to a local GeoTIFF file) is added to an image collection.

When adding a GDAL dataset (given its identifier) to a collection, we must find out

- to which image the dataset belongs (identified by a unique image name),
- the recording date/time of the dataset, and
- how bands of the dataset relate to bands of the collection.



Regular Expressions
----------------------------------------------------------

Rules are mostly defined as regular expresions on the GDAL dataset identifiers. Some expressions
simply check whether a dataset *matches* the expression or not, whereas some expression return a captured substring (e.g. the datetime part). gdalcubes uses the `standard C++11 regular expression library <http://www.cplusplus.com/reference/regex>`_ with default `ECMAScript <http://www.cplusplus.com/reference/regex/ECMAScript>`_ syntax.



JSON Format
------------------------------------


Header Information
####################################

The JSON files for image collection formats start with some header information, 
including a short description of the particular product it adresses and some keywords. These header information are optional, but may be useful for searching.

For example, the following description and tags are used in the collection format
for Sentinel 2 Level 2A products.

.. code-block:: json
   :caption: Example header information of the Sentinel 2 Level 2A collection format.
   
    "description" : "Image collection format for Sentinel 2 Level 2A data as downloaded from the Copernicus Open Access Hub, expects a list of file paths as input. The format should work on original ZIP compressed as well as uncompressed imagery.",
    "tags" : ["Sentinel", "Copernicus", "ESA", "BOA", "Surface Reflectance"]
   


Global Dataset Pattern 
####################################

Collection formats include a global pattern as a regular expression 
such that input GDAL datasets not matching the regular expression will 
simply be ignored (quitly, without throwing exceptions). 
As a simple example, setting ``"pattern" : ".*\\.tif$"``
would ignore files that do not end with ".tif".  




Grouping Datasets by Image Names
####################################


Images can be composed from one or more GDAL datasets, 
where all datasets share the spatial extent, the spatial 
reference system, and acquisition date/time, 
but different datasets contain data for different bands. 

gdalcubes uses a regular expression to extract an *image name* 
from the GDAL dataset identifier. 
The extracted name must be identical for all datasets 
belonging to the same image.

To extract the image name from the identifier, 
the first *marked subexpression* (or *capturing group*) of the 
provided regular expression under ``"images" : {"pattern": "REGEX"}``
is used, i.e., a part in the expression within the first pair of parentheses. 



Example 1 (Landsat 8 surface reflectance)
************************************************

.. code-block:: json

   "images":{"pattern":".*(L[OTC]08_.{4}_.{6}_.{8}_.{8}_.{2}_.{2})[A-Za-z0-9_]+\\.tif"}


+------------------------------------------------------------------------------------+ 
| Example input GDAL dataset identifiers and extracted **image name** (bold)         |  
+====================================================================================+
| 1. "/path/to/**\ LC08_L1TP_226063_20140719_20170421_01_T1**\ _sr_band2.tif"        |
| 2. "/path/to/**\ LC08_L1TP_226063_20140820_20170420_01_T1**\ _sr_aerosol.tif"      |
| 3. "/path/to/**\ LC08_L1TP_227064_20130621_20170503_01_T1**\ _pixel_qa.tif"        |
+------------------------------------------------------------------------------------+ 




Example 2 (MODIS MOD13A2)
**************************************

.. code-block:: json

   "images":{"pattern":"HDF4_EOS:EOS_GRID:\".+(MOD13A2\\.A.+)\\.hdf.*"}


+---------------------------------------------------------------------------------------------------------------------------------------------+ 
| Example input GDAL dataset identifiers and extracted **image name** (bold)                                                                  |  
+=============================================================================================================================================+
| 1. "HDF4_EOS:EOS_GRID:\\"/path/to/**\ MOD13A2.A2013353.h00v08.006.2018226105756**\ .hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days NDVI"       |
| 2. "HDF4_EOS:EOS_GRID:\\"/path/to/**\ MOD13A2.A2013353.h20v03.006.2018226105924**\ .hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days NDVI"       |
| 3. "HDF4_EOS:EOS_GRID:\\"/path/to/**\ MOD13A2.A2015033.h23v10.006.2015296122819**\ .hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days VI Quality" |
+---------------------------------------------------------------------------------------------------------------------------------------------+ 






Extracting date/time information
####################################

In the current version of gdalcubes, the acquisition 
date/time of images is derived from the dataset identifiers. 
Similar to the extraction of image names, a pattern defines a regular expression where the first marked subexpression / capturing group within parentheses is extracted. The ``format`` field in ``datetime`` JSON object then defines how to convert the extracted string to a date/time object, according to the `strptime function <http://pubs.opengroup.org/onlinepubs/9699919799/functions/strphtml>`_..



Example 1 (Sentinel 2 Level 2A SAFE format, from .zip files)
*********************************************************************

.. code-block:: json

   "datetime" : {
        "pattern" : ".*MSIL2A_(.+?)_.*\\.zip.*",
        "format" : "%Y%m%dT%H%M%S"}   


+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
| Example input GDAL dataset identifiers and extracted **date/time string** (bold)                                                                                                                                                                               |  
+================================================================================================================================================================================================================================================================+
| 1. "/vsizip//path/to/S2A_MSIL2A\_\ **20180328T093031**\ _N0207_R136_T34UFD_20180328T145945.zip/S2A_MSIL2A_20180328T093031_N0207_R136_T34UFD_20180328T145945.SAFE/GRANULE/L2A_T34UFD_A014433_20180328T093032/IMG_DATA/R20m/T34UFD_20180328T093031_B8A_20m.jp2"  |
| 2. "/vsizip//path/to/S2A_MSIL2A\_\ **20180328T093031**\ _N0207_R136_T34UFD_20180328T145945.zip/S2A_MSIL2A_20180328T093031_N0207_R136_T34UFD_20180328T145945.SAFE/GRANULE/L2A_T34UFD_A014433_20180328T093032/IMG_DATA/R20m/T34UFD_20180328T093031_SCL_20m.jp2"  |
| 3. "/vsizip//path/to/S2A_MSIL2A\_\ **20180430T094031**\ _N0207_R036_T34UGD_20180430T114456.zip/S2A_MSIL2A_20180430T094031_N0207_R036_T34UGD_20180430T114456.SAFE/GRANULE/L2A_T34UGD_A014905_20180430T094109/IMG_DATA/R10m/T34UGD_20180430T094031_B08_10m.jp2"  |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 




Example 2 (MODIS MOD13A2)
**************************************

.. code-block:: json

    "datetime" : {
        "pattern" : ".*M[OY]D13A2\\.A(.{7}).*",
        "format" : "%Y%j"} 


+---------------------------------------------------------------------------------------------------------------------------------------------+ 
| Example input GDAL dataset identifiers and extracted **date/time string** (bold)                                                            |  
+=============================================================================================================================================+
| 1. "HDF4_EOS:EOS_GRID:\\"/path/to/MOD13A2.A\ **2013353**\ .h00v08.006.2018226105756.hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days NDVI"       |
| 2. "HDF4_EOS:EOS_GRID:\\"/path/to/MOD13A2.A\ **2013353**\ .h20v03.006.2018226105924.hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days NDVI"       |
| 3. "HDF4_EOS:EOS_GRID:\\"/path/to/MOD13A2.A\ **2015033**\ .h23v10.006.2015296122819.hdf\\":MODIS_Grid_16DAY_1km_VI:1 km 16 days VI Quality" |
+---------------------------------------------------------------------------------------------------------------------------------------------+ 



Defining Image Collection Bands
####################################


The ``bands`` object lists the bands of a data product as key value pairs, where the key is a unique band name, and the value is a JSON object with a pattern and optional other fields. The `pattern` field again defines a regular expression. If a GDAL dataset identifier matches the pattern, it is considered to contain data of the band.  An dataset identifier may match the pattern of several bands (sometimes, all bands even define the same pattern) if a single input GDAL dataset contains multiple bands. In this case, the additional field `band` can be used to describe which internal band corresponds to the band of the image collection. ``band`` can be a one-based integer number and is identical to the band number of GDAL (as from running ``gdalinfo``).


Some additional per-band metadata fields may be added to band definitions. In the current version, these include ``nodata``, ``scale``, ``offset``, and ``unit``. If these values are not provided, they are derived from the GDAL metadata (which may or may not be defined).


Example 1: One GDAL dataset - one band
***************************************

The following example can be used to define some of the bands of Landsat 8 surface reflectance data, where each band is stored in a separate GeoTIFF file.

.. code-block:: json
   :caption: Example band definitions for Landsat 8 surface reflectance collection format.
    
    "bands": {
        "B01" : {
            "pattern" : ".+sr_band1\\.tif",
            "nodata" : -9999},
        "B02" : {
            "pattern" : ".+sr_band2\\.tif",
            "nodata" : -9999},
        "B03" : {
            "pattern" : ".+sr_band3\\.tif",
            "nodata" : -9999},
        "B04" : {
            "pattern" : ".+sr_band4\\.tif",
            "nodata" : -9999},
        "B05" : {
            "pattern" : ".+sr_band5\\.tif",
            "nodata" : -9999},
        "B06" : {
            "pattern" : ".+sr_band6\\.tif",
            "nodata" : -9999},
        "B07" : {
            "pattern" : ".+sr_band7\\.tif",
            "nodata" : -9999},
        "..."
    }


Example 2: One GDAL dataset - many bands
******************************************

The following example can be used to define bands of PlanetScope surface reflectance data, where all bands (except a mask band) are stored in a single GeoTIFF file.

.. code-block:: json
   :caption: Example band definitions for PlanetScope surface reflectance collection format.
   
    "bands" : {
        "red" : {
            "nodata" : 0,
            "pattern" : ".+_AnalyticMS_SR\\.tif$",
            "band" : 3},
        "green" : {
            "nodata" : 0,
            "pattern" : ".+_AnalyticMS_SR\\.tif$",
            "band" : 2},
        "blue" : {
            "nodata" : 0,
            "pattern" : ".+_AnalyticMS_SR\\.tif$",
            "band" : 1},
        "nir" : {
            "nodata" : 0,
            "pattern" : ".+_AnalyticMS_SR\\.tif$",
            "band" : 4},
        ...
    }




Additional Image Metadata
####################################

It is possible to extract further per-image metadata key value pairs from GDAL datasets. The collection format may include an optional field "image_md_fields" to list metadata keys as a JSON array of strings. When GDAL datasets are opened, GDAL tries to find the corresponding metadata keys and stores corresponding values in the image metadata table of the image collection.

Metadata fields may be separated by domains (see GDAL metadata model at https://gdal.org/user/raster_data_model.html#metadata). If metadata fields from a specific domain are needed, you can use "DOMAIN:KEY", such as "IMAGERY:CLOUDCOVER". The example below could be used to get some per-image quality flags from MODIS metadata.

.. code-block:: json
   :caption: Example metadata fields for a MODIS (MOD13A2) product.
   
   "image_md_fields" : ["PERCENTLAND", "QAPERCENTGOODQUALITY", "QAPERCENTNOTPRODUCEDCLOUD"]


Complete Examples
-------------------------------------

Complete examples of image collection formats can be found in the sources. There is also a `dedicated GitHub repository <https://github.com/appelmar/gdalcubes_formats/tree/master/formats>`_.



Notes on Writing Custom Collection Formats
--------------------------------------------

When writing a collection format for new data products, please make sure 
to check the following notes:

- If available, read the data product handbook. Most official satellite image product handbooks include a section on filenaming conventions.
- For portability of local file-based image collections, make sure that preceding directory names (e.g. "C:\\Users\\", or "/home/user/data") do not matter to successfully create image collections.  
- If possible, avoid using path separators in regular expressions or use *non-capturing alternation* ``(?:/|\\)`` if you have to.
- Image collections do not need to include data for all bands. It is recommended to list all possible bands of a data product in the format. For example, Landsat 8 surface reflectance products may or may not include additional precomputed spectral index bands. To be able to use these bands if available, they must be listed in the collection format.