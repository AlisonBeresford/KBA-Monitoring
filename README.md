# KBA-Monitoring

# Summary
Contains Google Earth Engine (GEE) JavaScript code for extracting data on fire frequency, forest loss and night-time lights on an annual basis across a set of sites (e.g. Key Biodiversity Areas (KBAs), Protected Areas etc.).

# Site boundaries
Requires site boundaries in kml format to be uploaded to Google Drive as a Google Fusion Table, and imported into GEE as a feature collection.  Digitised boundaries of KBAs are avilable on request from BirdLife International (see http://www.keybiodiversityareas.org).

# Fire frequency (fire_FIRMS.txt)
Fire data are obtained from the Fire Information for Resource Management System (FIRMS; https://explorer.earthengine.google.com/#detail/FIRMS). These data map the location of fires active when the MODIS satellite passed over, at a temporal resolution of one day and a spatial resolution of 1km.  Where fires span more than one 1km pixel, a separate fire event is recorded in each pixel and where fires last more than one day, a separate fire event is recorded each day.  Data are available from 2001 onwards.  Each daily image is re-classified into a binary raster (fire/no fire), from which annual summary maps representing the total numbers of fire events in each year are created. These maps are then intersected with the site boundaries to calculate for each site in each year, the total number of fire events observed.  It is recommended that for making comparisons between sites, the number of fire events for each site in each year should be divided by the area of the site to give a standardised number of fire events per year per unit area (this final step is not included in the GEE code).

# Forest loss (forest_loss_Hansen.txt)
Tree cover and tree cover change data are obtained from the Hansen Global Forest Change dataset (version 1.6; https://explorer.earthengine.google.com/#detail/UMD%2Fhansen%2Fglobal_forest_change_2018_v1_6). These data map tree cover in the year 2000 at a resolution of 1 arc second (c. 30m at the equator), and record loss and gain in each pixel in each subsequent year up to and including 2018.  The code extracts tree cover in the year 2000, selecting all pixels where tree cover was greater than 50% (this value can be modified by the user).  The binary layer created (1 = forest, 0 = non-forest in the year 2000) is then intersected with the site boundaries to extract the area (number of pixels) of each site  containing tree cover in the year 2000.  The “lossyear” band is used to produce a layer per year, from 2001 to 2018 inclusive, indicating, for each tree-covered pixel, whether tree cover was lost in that year.  These annual loss layers are then intersected with the site boundaries to extract the area (number of pixels) of tree cover lost each year in each site.  Together, these values can then be used to calculate the percentage of tree cover lost in each site in each year, relative to the area of tree cover present in 2000 (this final step is not included in the GEE code).

To convert outputs in pixels to areas in square meters, multiply by the scale factor (pixel size).  E.g. if using scale:30, pixel size is 30x30m so area in square metres = no. of pixels x 30 x 30

It should be noted that the output units have been deliberately kept as pixels, rather than area units (e.g. square metres) because of an issue with the way GEE handles the ee.Reducer.sum() command at different scales.

The way that GEE handles scale is described here: https://developers.google.com/earth-engine/scale.  If the scale variable is specified as the native resolution of the data (e.g. using Hansen data at scale 30 (30x30m pixels)), no resampling takes place.  However, if the scale value is different from the native resolution of the data, the first thing GEE does is to resample your input images to the specified output scale.  The issue is that it does this resampling based on the mean (for continuous data) or a sample (for discrete data), even though it’s all wrapped up within a sum function.  This is represented diagramatically in the attached GEE_resample.png file. If each of the lower squares in the diagram represents a 1x1km pixel, then the top square is what you get when you specify a scale of 4000 (4x4km).  In the example given, the true sum is 32, but the resampling done by GEE gives a value of 2.  For this reason, it is reccommended to work either at the native resolution of the data (e.g. using Hansen data at scale 30) or calculate values as number of pixels and convert to square meters or other area units at the end of the analysis.

# Night-time lights (nightlights_DMSP_OLS.txt and nightlights_VIIRS.txt)
Two sources of night-time lights data are used, covering two different time periods.  For the period 1992 to 2013, the Defence Meteorological Program Operational Linescan System (DMSP OLS) Nighttime Lights Time Series (version 4; https://explorer.earthengine.google.com/#detail/NOAA%2FDMSP-OLS%2FNIGHTTIME_LIGHTS) is used, which measures visible and near-infrared emission sources at night at a resolution of 30 arc seconds.  The “stable lights” band is used, which has been processed so that ephemeral events such as fires have been discarded and background noise identified and replaced with values of zero.

For the period 2014 to 2018, the Visible Infrared Imaging Radiometer Suite (VIIRS) Stray Light Corrected Nighttime Day/Night Band Composites (version 1; https://explorer.earthengine.google.com/#detail/NOAA%2FVIIRS%2FDNB%2FMONTHLY_V1%2FVCMSLCFG) is used.  This measures monthly average radiance at a resolution of 15 arc seconds and uses a procedure to correct for stray light.  However, neither background values or ephemeral events such as fires, boats and aurora have been filtered out.

Due to the differences in the methods used to produce the two datasets, it is not possible to make any direct comparisons between the two time periods.  For both datasets, mean images per year are generated and then intersected with site boundaries to give a mean value per year per site for stable lights (1992-2013) and average radiance (2014-2018).

# Outputs
All four scripts create outputs as csv files.  The inclusion of long description and coordinate fields mean these are not handled well by Microsoft Excel, but they are easily read and manipulated using R.
 
# Contact
For any queries, contact alison.beresford@rspb.org.uk.
