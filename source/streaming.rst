Streaming
==============================

Chunks of a data cube can be *streamed* to external processes. For example
this allows to run user-defined R or Python scripts on chunks of data cubes in parallel.

There are currently three operations supporting chunk streaming:


1. ``stream_apply_pixel`` streams chunks of a source data cube to an external processs and expects the same number of pixels as a result, but any number of bands.   
2. ``stream_reduce_time`` combines chunks of the same spatial part but different spans of time first, such that one resulting chunk contains complete time series of the cube and afterwards calls an external process on chunks. The external process is expected to produce a single time slice (i.e., nt = 1), the same number of pixels as the source data cube in space, and any number of bands.
3. ``stream_chunk`` is a less user-friendly but more general operation that tries to derive the dimensionality of the result cube automatically, by calling the process on dummy data once. 

Notice that the number of result bands must be identical for all input chunks. The R package 
includes an easy to use implementation that allows passing R functions to ``apply_pixel`` and ``reduce_time``.


Binary Serialization Format
-----------------------------

To stream data to and from the external process, a custom binary serialization format is used.  
The format includes data cube values of one chunk and includes dimension values, band names, and the spatial reference system.

.. warning::

   The binary serialization format will be replaced with CF compliant netCDF-4 files in future versions.  


The format contains

1. the size of the chunk as 4 * 4 bytes in total (nbands:int32, nt:int32, ny:int32, nx:int32),

2. band names, where each band starts with the number of characters as int32 followed by actual characters,

3. dimension values in the order time, y, x as doubles, summing to (nt + ny + nx) * 8 bytes in total,

4. the spatial reference system as a string (number of characters as int32, followed by actual characters), and

5. actual values of the chunk as doubles (summing to nbands * nt * ny * nx * 8 bytes).



