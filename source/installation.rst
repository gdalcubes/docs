Installation
==================================================







C++ Library and Tools
---------------------------------
 
gdalcubes can be compiled from sources via `CMake <https://cmake.org>`_. CMake automatically checks for mandatory and optional dependencies and adapts
the build configuration. The following commands install gdalcubes from sources. 

.. code-block:: bash

    git clone https://github.com/appelmar/gdalcubes && cd gdalcubes
    mkdir -p build 
    cd build 
    cmake -DCMAKE_BUILD_TYPE=Release ../ 
    make 
    sudo make install


If any of the required libraries are not available on your system, please use your package manager to install these before, e.g. with
(Ubuntu):

.. code-block:: bash

    sudo apt-get install libgdal-dev libnetcdf-dev libcurl4-openssl-dev libsqlite3-dev

    # additional installs for command line interface 
    sudo apt-get install libboost-program-options-dev libboost-system-dev

    # additional installs for server executable
    sudo apt-get install libcpprest-dev

Please notice we have not yet tried to build
with Microsoft Visual Studio. However, the R package installation from sources includes building 
the C++ library and works on Windows, MacOS and Linux (see `R Package`_).


Docker
###################

The `Dockerfile` at the root of the project is built on a minimal Ubuntu installation, including installation of all dependencies and compiles 
gdalcubes from sources automatically. 

.. code-block:: bash
   :caption: Installing gdalcubes in a Docker container.

    git clone https://github.com/appelmar/gdalcubes && cd gdalcubes 
    docker build -t appelmar/gdalcubes .
    docker run -d -p 11111:1111 appelmar/gdalcubes # runs gdalcubes_server as a deamon 
    docker run appelmar/gdalcubes /bin/bash # run a command line where you can run gdalcubes 





R Package
---------------------------------

The R package can be installed from `CRAN <https://cran.r-project.org/package=gdalcubes>`_, including binary package builds
for MacOS and Windows:

.. code-block:: R

    install.packages("gdalcubes")


Alternatively, installation from sources is easiest with 

.. code-block:: R

    remotes::install_git("https://github.com/appelmar/gdalcubes_R")


Please make sure that the git command line client is available on your system. Otherwise, the above command might not clone the gdalcubes C++ library as a submodule under src/gdalcubes.
The package links to the external libraries `GDAL <https://www.gdal.org>`_, `NetCDF <https://www.unidata.ucar.edu/software/netcdf>`_, `SQLite <https://www.sqlite.org>`_, and `curl <https://curl.haxx.se/libcurl>`_.

Windows
#########

On Windows, you will need `Rtools <https://cran.r-project.org/bin/windows/Rtools>`_. System libraries are automatically downloaded from `rwinlib <https://github.com/rwinlib>`_.

Linux
#########

Please install the system libraries e.g. with the package manager of your Linux distribution. Also make sure that you are using a recent version of GDAL (> 2.3.0). On Ubuntu, the following commands install all libraries.

.. code-block:: bash

    sudo add-apt-repository ppa:ubuntugis/ppa && sudo apt-get update
    sudo apt-get install libgdal-dev libnetcdf-dev libcurl4-openssl-dev libsqlite3-dev libudunits2-dev

MacOS
#########

Use `Homebrew <https://brew.sh>`_ to install system libraries with

.. code-block:: bash

    brew install pkg-config
    brew install gdal
    brew install netcdf
    brew install libgit2
    brew install udunits
    brew install curl
    brew install sqlite













