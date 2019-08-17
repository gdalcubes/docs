Distributed Data Cube Processing
==================================


.. warning::

   Features described here are experimental and have not yet been implemented in the R package.


Data cubes can be processed in distributed computing envrionments, where nodes communicate over HTTP.


One coordinator needs to define a *swarm* of workers, each running the gdalcubes server application.
The coordinator then simply distributes chunks to be processed to the available workers.  This basically involves three steps:

1. Send the JSON process graph and required additional files (e.g. image collections) to all worker nodes.
2. Distribute chunks to individual workers, which then start processing.
3. Waits for the workers, download the data and e.g. write to the output netCDF file.  

.. note::

   Since image collections are shared, links to GDAL datasets must be accessible on all worker nodes.

.. warning::

   The current implementation is not robust against failures or unavailability of workers.

.. warning::

   Distributed processing is currently not implemented for streaming operations.


Server API
----------------------------------------------

The worker instances are running the server executable, exposing a a REST-like API for communication.
The following sections describes available endpoints. By default, the `gdalcubes_server` executable will 
listen on `http://0.0.0.0:1111/gdalcubes/api/`.


``GET /version``
#######################

Returns version information as text.



``HEAD /file?name=xyz&size=1234``
#########################################

Checks whether the server instance has a file with name and size equal to the given query parameters. Returns
200 (OK), if the file is available, 409 (Conflict) if a file with the same name has different size, 400 (Bad Request) if size and / or name is missing in the request, or 204 (No Content) if the file does not exist.



``POST /file?name=xyz``
#############################

Uploads the file in the application/octet-stream body as file with name as given in the query parameters.



``POST /cube``
##############################

Uploads a JSON description of a cube and returns a cube ID.




``POST /cube/{cube_id}/{chunk_id}/start``
##############################################

Queue a chunk read for a given cube and given chunk id as given in the path, returns immediately.




``GET /cube/{cube_id}/{chunk_id}/status``
#####################################################################

Ask for the status of a chunk read request. Possible return values are ``notrequested``, ``queued``, ``running``, ``finished``, and ``error``.




``GET /cube/{cube_id}/{chunk_id}/download``
##############################################

Download a chunk with given id as application/octet-stream. The chunk must have been queued before. If the chunk has not yet been
read, it will block until the data becomes available.