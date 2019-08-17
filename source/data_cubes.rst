Data Cubes
==================================================

gdalcubes implements *regular dense raster data cubes*, making the following assumptions (see [ApPe19]_):


1. Spatial dimensions refer to a single spatial reference system (SRS);
2. Cells of a data cube have a constant spatial size (with regard to the cube’s SRS);
3. The spatial reference is defined by a simple offset and the cell size per axis, i.e., the cube axes are aligned with the SRS axes;
4. Cells of a data cube have a constant temporal duration, defined by an integer number and a date or time unit (years, months, days, hours, minutes, or seconds);
5. The temporal reference is defined by a simple start date/time and the temporal duration of cells;
6. For every combination of dimensions, a cell has a single, scalar (real) attribute value.


Data cubes are always four-dimensional (bands, time, y, x).



Data Cube View
--------------------------------------------------

To create data cubes, users first define virtual data cubes as a *data cube view*, without connecting it to specific datasets. The data cube view contains information on the geometry of the cube, including

- the spatial reference system (SRS),
- the spatiotemporal extent (left, right, bottom, top, start date/time, end date/time), and
- the spatiotemporal size of pixels (spatial size, temporal duration).

The resolution of dimensions can be specified either by the number of pixels (nx, ny, nt), or by the size of pixels (dx, dy, dt). 
If the size of the extent in one dimension does not align with the corresponding pixel size, the extent will be enlarged accordingly. 
For example, assuming ``left = 1``, ``right = 10``, and ``dx = 2``, pixels would start at ``x = [1,3,5,7,9]`` such that the last pixel would have size 1. In this case, the extent would be replaced by ``left=0.5``, ``right=10.5``, such that pixels start at ``x = [0.5, 2.5, 4.5, 6.5, 8.5]`` and all pixels have size 2.

Please notice that the cube view does not include any information on the band dimension, i.e., the cube view is independent of particular data products. 

Handling Date/time
####################################################

The temporal duration of pixels (dt) includes a temporal unit (or *granularity*). Following ISO8601 durations, it is defined as a string starting with "P", following by an integer number and a unit (one of "Y", "M", "D", "TH", "TM", "TS"). In contrast to ISO8601, it is however **not** possible to mix several units, e.g. ``"P1M10DT2H"`` (one month, 10 days, and 2 hours). 

If dt uses a larger unit than t0 and t1, the temporal extent is enlarged. For instance, defining ``t0 = "2019-03-05"``, ``t1 = "2019-06-05"``, and monthly temporal resolution ``dt = "P1M"``, will lead to the extent ``t0 = "2019-03"`` and ``t1 = "2019-06"``, such that the data cube covers the full March and June of 2019.



Data Cube Data Model
--------------------------------------------------

To manage large data cubes under limited main memory, a data cube is divided into smaller chunks, where the  size of chunks (how many pixels in space and time) can be user-defined. A chunk contains values for all bands and stores values as a simple four-dimensional double array.  


.. TODO: add data cube image

.. TODO: add data cube class description (public methods)

.. TODO: parallelization?



Creating Data Cubes from Image Collections
--------------------------------------------------

Combining a *data cube view* with an *image collection* yields a regular raster data cube with band data from the image collection and geometry from the data cube view. 

To read values of data cube chunk from an image collection, the following algorithm is used [ApPe19]_:

.. _cube_creation_alg:

1. Allocate and initialize an in-memory chunk buffer for the resulting chunk data (a four-dimensional bands, t, y, x array);

2. Find all images of the collection that intersect with the spatiotemporal extent of the chunk;

3. For all images found:

    3.1. Crop, reproject, and resample according to the spatiotemporal extent of the chunk and the data cube view and store the result as an in-memory three-dimensional (bands, y, x) array;

    3.2. Copy the result to the chunk buffer at the correct temporal slice. If the chunk buffer already contains values at the target position, update a pixel-wise aggregator (e.g., mean, median, min., max.) to combine pixel values from multiple images which are written to the same cell in the data cube.

4. Finalize the pixel-wise aggregator if needed (e.g., divide pixel values by n for mean aggregation).






Aggregation and Resampling
##################################################

To build regular data cubes from image collections, individual images are reprojected, rescaled, cropped, and resampled with regard to the data cube view first. Afterwards, pixel values of several images that are located within the same data cube cell (i.e. from different days in the same month) are combined by an aggregation function. Methods for both, spatial *resampling* and temporal *aggregation* can be provided in the data cube view. gdalcubes supports all `resampling methods implemented in GDAL <https://gdal.org/programs/gdalwarp.html#cmdoption-gdalwarp-r>`__. Supported aggregation methods include ``"first"``, ``"min"``, ``"max"``, ``"median"``, ``"mean"``, and ``"last"``.  





Proxy Objects, Lazy Evaluation
--------------------------------------------------

Creating data cubes in the C++ library or the R package is a cheap operation and does not 
read any pixel values nor run any expensive operations. Instead, it returns a *proxy* object, knowing the 
shape of the data cube. This allows for *lazy evaluation* and some optimizaitons when operations
on data cubes are chained (see :doc:`operations`).


Image Masks
--------------------------------------------------

When data products include mask bands, these can be used to filter pixels
of images contributing the data cube cells. Masks are applied on images, not on cubes (during step 3.1 in  :ref:`above <cube_creation_alg>`). As such, masked values simply will not contribute to the pixel-wise aggregation.

In gdalcubes, masks can be defined by band name and a set of mask values (or a value range). If a pixel's mask band value is within the set of values, this pixel is masked, otherwise not. Optionally, the masked can be inverted, and a bitwise AND can be applied to extract specific bits of a bit mask (e.g. for MODIS quality flag bands).  





.. rubric:: References

.. [ApPe19] Appel, M., & Pebesma, E. (2019). On-Demand Processing of Data Cubes from Satellite Image Collections with the gdalcubes Library. Data, 4(3), 92.