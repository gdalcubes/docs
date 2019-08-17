FAQ
==============================

.. contents:: 

Can I use gdalcubes for dataset XYZ?
-------------------------------------

Most likely, not out of the box! gdalcubes comes with a few predefined image collection
formats for Sentinel-2, Landsat, MODIS, and a few other datasets only. 
You will need to write a custom image collection format (see :ref:`ref_collection_formats`). 
Please share your collection formats via a pull request on the `GitHub collection format respository <https://github.com/appelmar/gdalcubes_formats>`_.


How to contribute?
-----------------------------

Contributions of any kind (whether fixing typos, improving documentation, reporting bugs, asking questions, or implementing new features) are very welcome.
Please use issues and pull requests on GitHub.



How do I cite gdalcubes?
-----------------------------

Please refer to our paper at https://doi.org/10.3390/data4030092. You can use the corresponding BibTeX entry:

.. code-block:: bibtex


    @Article{data4030092,
        AUTHOR = {Appel, Marius and Pebesma, Edzer},
        TITLE = {On-Demand Processing of Data Cubes from Satellite Image Collections with the gdalcubes Library},
        JOURNAL = {Data},
        VOLUME = {4},
        YEAR = {2019},
        NUMBER = {3},
        ARTICLE-NUMBER = {92},
        URL = {https://www.mdpi.com/2306-5729/4/3/92},
        ISSN = {2306-5729},
        DOI = {10.3390/data4030092}
    }




