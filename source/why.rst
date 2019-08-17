Why gdalcubes?
==============================


Collections of satellite imagery are difficult to analyze because of their complex structure:

- spectral bands of one image may have different pixel sizes,
- two or more images may spatially overlap, have different spatial reference systems, and yield irregular time series,
- Images from different data products (or different satellites) do not align in space and time, and use different data formats.

The Geospatial Data Abstraction Library (GDAL) can be used to solve some of these problems by
abstracting from particular file formats and providing high performance image warping (reprojection, rescaling, cropping and reampling), but does
not know about image time series. 


gdalcubes builds on top of GDAL and aims at making the work with large satellite image collections **easier**, **faster**, **more intuitive**, and **more interactive**.




