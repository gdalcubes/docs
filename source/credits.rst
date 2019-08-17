Credits
==================================================

gdalcubes wouldn't exist without other open-source projects.
This document presents a list of used third-party software including their purpose, copyright, and licensing information 
in no particular order. Please notice that some libaries are only used by the command line client or gdalcubes_server, but not 
by the core library.

- `GDAL <https://www.gdal.org/>`_: *A translator library for raster and vector geospatial data formats*
    - Copyright (c) 2000, Frank Warmerdam
    - Copyright (c) The GDAL/OGR project team
    - License:  MIT (https://opensource.org/licenses/MIT)
    - Parts of GDAL are licensed under different terms, see https://github.com/OSGeo/gdal/blob/master/gdal/LICENSE.TXT
    - gdalcubes may statically or dynamically link to the gdal library depending on compilation flags


- `json <https://github.com/nlohmann/json>`_:  *JSON for Modern C++*
     - Copyright (c) 2013-2018 Niels Lohmann
     - License: MIT (https://opensource.org/licenses/MIT) 
     - gdalcubes distributes a **modified** version of the library under `src/external/json.hpp`, modifications are limited to commenting out diagnostic pragmas (see https://github.com/appelmar/gdalcubes/commit/9e936155489a97c5bb211e13e757236762ee1d96)


- `SQLite <https://www.sqlite.org/>`_:  *A self-contained, high-reliability, embedded, full-featured, public-domain, SQL database engine**
     - Copyright:  *public domain*
     - License: *public domain*
     - gdalcubes may statically or dynamically link to the sqlite library depending on compilation flags


- `CURL <https://curl.haxx.se/>`_: *Command line tool and library for transferring data with URLs*
     - Copyright (c) 1996 - 2018, Daniel Stenberg
     - License: curl license(https://curl.haxx.se/docs/copyright.html) 
     - gdalcubes may statically or dynamically link to the libcurl library depending on compilation flags



- `TinyExpr <https://github.com/codeplea/tinyexpr>`_: *A very small recursive descent parser and evaluation engine for math expressions*
    - Copyright (c) 2015-2018 Lewis Van Winkle
    - License:  zlib (https://opensource.org/licenses/Zlib) 
    - gdalcubes distributes a *modified* version of the library under `src/external/tinyexpr`, modifications include the implementation of logical and bit-wise operators and remove ISO C -Wpedantic warnings.



- `netCDF <https://www.unidata.ucar.edu/software/netcdf>`_: *The Unidata network Common Data Form C library*
    - Copyright (c) 1993-2017 University Corporation for Atmospheric Research/Unidata
    - License: MIT-like, see https://www.unidata.ucar.edu/software/netcdf/copyright.html
    - DOI: http://doi.org/10.5065/D6H70CW6
    - gdalcubes may statically or dynamically link to the NetCDF C library depending on compilation flags



- `tiny-process-library <https://gitlab.com/eidheim/tiny-process-library>`_: *A small platform independent library making it simple to create and stop new processes in C++*
    - Copyright (c) 2015-2018 Ole Christian Eidheim
    - License:  MIT (https://opensource.org/licenses/MIT)      
    - gdalcubes_server includes may statically or dynamically link to the cpprestsdk library depending on compilation flags 
    - gdalcubes distributes a **modified** version of the library under `src/external/tiny-process-library`, modifications are limited to replacing _exit() and exit() calls by raise(SIGKILL), in order to comply with R CMD check.



- `Catch2 <https://github.com/catchorg/Catch2>`_: *A modern, C++-native, header-only, test framework for unit-tests, TDD and BDD*
    - Copyright (c) 2010 Two Blue Cubes Ltd
    - License:  Boost Software License 1.0 (https://www.boost.org/LICENSE_1_0.txt)
    - gdalcubes distributes an unmodified version of the library under `src/external/catch.hpp`



- `Boost.Filesystem <https://www.boost.org/doc/libs/1_68_0/libs/filesystem/doc/index.htm>`_
    - Copyright (c) Beman Dawes, 2011
    - License:  Boost Software License 1.0 (https://www.boost.org/LICENSE_1_0.txt)
    - gdalcubes may statically or dynamically link to the Boost.Filesystem library depending on compilation flags         



- `Boost.Program_options <https://www.boost.org/doc/libs/1_68_0/doc/html/program_options.html>`_
    - Copyright (c) 2002-2004 Vladimir Prus
    - License:  Boost Software License 1.0(https://www.boost.org/LICENSE_1_0.txt)       
    - gdalcubes may statically or dynamically link to the Boost.Program_options library depending on compilation flags         



- `Date <https://github.com/HowardHinnant/date>`_: *A date and time library based on the C++11/14/17 <chrono> header*  
    - Copyright (c) 2015, 2016, 2017 Howard Hinnant
    - Copyright (c) 2016 Adrian Colomitchi
    - Copyright (c) 2017 Florian Dang
    - Copyright (c) 2017 Paul Thompson
    - Copyright (c) 2018 Tomasz Kamiński    
    - License: MIT (https://opensource.org/licenses/MIT)       
    - gdalcubes distributes a **modified** version of the library under `src/external/date.h`, modifications are limited to commenting out diagnostic pragmas, and adding a compilation flag for enabling __int128 support (see https://github.com/appelmar/gdalcubes/commit/9e936155489a97c5bb211e13e757236762ee1d96)
 


- `cpprestsdk <(https://github.com/Microsoft/cpprestsdk>`_
    - Copyright (c) Microsoft Corporation
    - License:  MIT (https://opensource.org/licenses/MIT)      
    - gdalcubes_server includes may statically or dynamically link to the cpprestsdk library depending on compilation flags 





Derived work such as R or Python packages may use further external software (see their documentation).  