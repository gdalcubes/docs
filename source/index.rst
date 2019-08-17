
Earth Observation Data Cubes from GDAL Image Collections
========================================================================


.. toctree::
   :caption: Introduction
   :maxdepth: 2
   :hidden:

   why.rst
   overview.rst
   installation.rst
   FAQ.rst
   license.rst
   credits.rst
   

.. toctree::
   :caption: Basic Concepts
   :maxdepth: 2
   :hidden:
   
   
   image_collections.rst
   collection_formats.rst
   data_cubes.rst
   operations.rst
   streaming.rst
   distributed_computing.rst


.. toctree::
   :caption: Tutorials
   :maxdepth: 2
   :numbered:
   :hidden:

   S2R

 

gdalcubes is a C++ library and R package aiming at making processing large collections of satellite imagery *easier*, *faster*, and *more interactive*.

It provides functions to create and process *four-dimensional regular raster data cubes* from irregular image collections, hiding 
complexities in the data such as different map projections and spatial overlaps of images, or different spatial resolutions of spatial bands from users.

Built-in functions on data cubes include

- reduction over space and time, 
- filtering by space, time, and  bands,
- arithmetics on band values,
- moving window kernels or aggregation over time,
- filling mising values by time series interpolation,
- combining bands of identically shaped cubes, 
- execution of external processes on data cube chunks (streaming), and 
- export as netCDF or GeoTIFF file(s),

gdalcubes constructs data cubes on the fly, making it straightforward to go from low resolution experiments to full resolution analyses.