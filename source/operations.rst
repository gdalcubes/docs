Data Cube Operations
==================================================

gdalcubes includes some built-in functions to process data cubes. Theese operations take one (or more) data cube(s) 
as input and produce a single result data cube. The table below lists available operations.


Built-in Operations
--------------------------------------------------

=====================  =====================  
Operation                Description   
=====================  =====================  
``select_bands``        Select a subset of a data cube's bands.  
``reduce_time``         Apply a reducer function over individual pixel time series. 
``reduce_space``        Apply a reducer function over spatial slices of a data cube.  
``apply_pixel``         Apply a arithmetic expression on band values per data cube pixel.    
``filter_pixel``        Filter pixels by a logical predicate on band values. 
``window_time``         Apply a moving window aggregate or kernel on individual pixel time series. 
``fill_time``           Fill missing values of a data cube by simple time series interpolation. 
``join_bands``          Combine bands of two identically shaped data cubes. 
=====================  =====================  


Internally these operations inherit from the general ``cube`` class. They provide a cheap ``create`` method, returning 
a *proxy* object without running any expensive computations on the data. 


This makes it possible to define and optimize chains of operations, where eexecution is delayed until users call a method
that reads data of the resulting data cube (such as plotting, or exporting a derived data cube as a file, see `Data Cube Export`_). 
Below, you can find examples in C++ and R, how data cube operations can be chained programmatically.


Examples
--------------------------------------------------


.. code-block:: c++
   :caption: Example (C++)

   #include <gdalcubes.h>
   
   int main() {

       // load an existing image collection
       auto ic = std::make_shared<gdalcubes::image_collection>("test.db");
       
       // define a data cube view
       gdalcubes::cube_view v;
       v.srs() = "EPSG:4326";
       v.left() = -180;
       v.top() = 60;
       v.bottom() = -60;
       v.right() = 180;
       v.t0() = gdalcubes::datetime::from_string("2018-01-01");
       v.t1() = gdalcubes::datetime::from_string("2018-12-31");
       v.dx(0.1);
       v.dy(0.1);
       v.dt(gdalcubes::duration::from_string("P1D"));
     
       // chain data cube operations
       auto x = gdalcubes::image_collection_cube(ic, v);
       auto x_bands = gdalcubes::select_bands_cube::create(x, {"B04", "B08"});
       auto x_ndvi = gdalcubes::apply_pixel_cube::create(x_bands, {"(B08-B04)/(B08+B04)"}, {"NDVI"});
       auto x_reduced =  gdalcubes::reduce_time_cube::create(x_ndvi, {{"max", "NDVI"}});

       // export result cube as a netCDF file
       x_reduced->write_netcdf_file("test.nc");

       return 0;
   }
   


.. code-block:: r
   :caption: Example (R)

   library(gdalcubes)
   library(magrittr) # for %>%

   ic = image_collection("test.db")
   v = cube_view(srs = "EPSG:4326", dx = 0.1, dy = 0.1, dt = "P1D", 
                 extent = list(left = -180, right = 180, top = 60, bottom = -60,
                               t0 = "2018-01-01", t1 = "2018-12-31"))
                            
   raster_cube(ic, v) %>%
     select_bands(c("B04", "B08")) %>%
     apply_pixel("(B08-B04)/(B08+B04)","NDVI") %>%
     reduce_time("median(NDVI)") %>%
     write_ncdf("test.nc")


Data Cube Export
--------------------------------------------------

Data cubes can be exported as single netCDF files, or as a collection of (possibly cloud-optimized) GeoTIFF files.

NetCDF Export
################

Produced netCDF files use the enhanced netCDF-4 data model, follow the `CF-1.6 conventions <http://cfconventions.org/cf-conventions/v1.6.0/cf-conventions.html>`_ and include bands of the data cube as variables. Dimensions are in the order *time*, *y*, *x* or *time*, *lat*, *lon* respectively.

Users can pass additional options to control properties of the resulting netCDF file:

- DEFLATE compression can be specified by integer values between 0 (no compression) and 9 (maximum compression).

- Users can furthermore enable or disable the inclusion of boundary variables in the output netCDF file. These variables give exact upper and lower limits of dimension values (e.g. start and end time of a pixel). 




GeoTIFF Export
################

Exporting a data cube as a collection of GeoTIFF files will produce one multiband GeoTIFF file per time slice of the data cube. Files will be named by a user defined prefix and the date/time of the slice, such as "cube_2018-01.tif". 

Users can pass additional options to control properties of the resulting GeoTIFF files:

- Setting ``overviews = true`` will generate GDAL overviews (image pyramids) for the resulting files. Overview levels reduce the pixel size by powers of 2, until width and height are lower than or equal to 256. The resampling algorithm for generating overviews can be selected from what GDAL offers (see `here <https://gdal.org/programs/gdaladdo.html#cmdoption-gdaladdo-r>`__, default is nearest neighbor).

- Setting ``COG = true`` will produce `cloud-optimized <https://www.cogeo.org/>`_ GeoTIFF files and includes ``overviews = true``. 

- Additional GDAL creation options can be passed, e.g., to define LZW or DEFLATE compression (see `here <https://gdal.org/drivers/raster/gtiff.html#creation-options>`__).



Packing Data Values
##################################################

Both, GeoTIFF as well as netCDF export support *packing data*, i.e. reducing the size of the output file(s) by conversion of double values to smaller integer types using an affine transformation of original values by an offset and scale, where values are transformed according to the formula: 

.. math::
   x_{unpacked} = x_{packed} \cdot scale + offset

Packing values reduces the precision of the data. Packing in gdalcubes requires users to pass 

- the output data type (one of ``uint8`` , ``uint16``, ``int16``, ``uint32``, ``int32``, ``float32``).
- scale, offset, and nodata values (unless type is ``float32`` ).

NaN values of data cubes will automatically converted to provided nodata values in the target data type. Individual scale, offset, and nodata values 
can be specified either once (applied for all bands), or individually per band.
Packing to type ``float32`` will ignore any of the offset, scale, and nodata values but implicitly cast 8 byte doubles to 4 byte float values.






JSON serialization
--------------------------------------------------

Data cube operations can be chained, producing a directed acyclic graph. This graph can be serialized
as JSON document, that can be useful to recreate data cubes, or keeping track of how a dataset has been generated. 
The following JSON document is an example, defining a median reduction of NDVI pixels on a yearly data cube over a small region in the Brazilian Amazon forest.  

.. code-block:: json
   :caption: Example process graph serialized as JSON document.
   
    {
    "cube_type": "reduce_time",
    "in_cube": {
        "band_names": [
        "NDVI"
        ],
        "cube_type": "apply_pixel",
        "expr": [
        "(b05-b04)/(b05+b04)"
        ],
        "in_cube": {
        "bands": [
            "B04",
            "B05"
        ],
        "cube_type": "select_bands",
        "in_cube": {
            "chunk_size": [
            16,
            256,
            256
            ],
            "cube_type": "image_collection",
            "file": "/tmp/RtmpacmRAy/file11af57b771e8.sqlite",
            "view": {
            "aggregation": "median",
            "resampling": "bilinear",
            "space": {
                "bottom": -550000.0,
                "left": -6180000.0,
                "nx": 2000,
                "ny": 2000,
                "right": -6080000.0,
                "srs": "EPSG:3857",
                "top": -450000.0
            },
            "time": {
                "dt": "P1Y",
                "t0": "2014",
                "t1": "2018"
            }
            },
            "warp_args": []
        }
        },
        "keep_bands": false
    },
    "reducer_bands": [
        [
        "median",
        "NDVI"
        ]
    ]
    }