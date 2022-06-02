# Watershed_Delineation

# Importing Libraries

import numpy as np  
import matplotlib.pyplot as plt  
import matplotlib.colors as colors  
import seaborn as sns  
import geopandas  
from pysheds.grid import Grid  
import mplleaflet  
%matplotlib inline  
  
  
# Opening Raster File
  
grid = Grid.from_raster('ned10m33101c8.tif')  
dem = grid.read_raster('ned10m33101c8.tif')  


# Plotting Raster Image

fig, ax = plt.subplots(figsize=(10,8))  
fig.patch.set_alpha(0)  
  
plt.imshow(dem, extent=grid.extent, cmap='terrain', zorder=1)  
plt.colorbar(label='Elevation (m)')  
plt.grid(zorder=0)  
plt.title('Digital elevation map', size=14)  
plt.xlabel('Longitude')  
plt.ylabel('Latitude')  
plt.tight_layout()  
  
<img src="Images/img/1.png">

### Conditioning DEM


# Filling pits in DEM
pit_filled_dem = grid.fill_pits(dem)  
  
# Filling depressions in DEM
flooded_dem = grid.fill_depressions(pit_filled_dem)  
    
# Resolving flats in DEM
inflated_dem = grid.resolve_flats(flooded_dem)  
  
### Determining D8 flow directions from DEM
# Specify directional mapping
dirmap = (64, 128, 1, 2, 4, 8, 16, 32)  
    
# Computing flow directions
fdir = grid.flowdir(inflated_dem, dirmap=dirmap)  

# Plotting Flow

fig = plt.figure(figsize=(8,6))  
fig.patch.set_alpha(0)  
  
plt.imshow(fdir, extent=grid.extent, cmap='viridis', zorder=2)  
boundaries = ([0] + sorted(list(dirmap)))  
plt.colorbar(boundaries= boundaries,  
             values=sorted(dirmap))  
plt.xlabel('Longitude')  
plt.ylabel('Latitude')  
plt.title('Flow direction grid', size=14)  
plt.grid(zorder=-1)  
plt.tight_layout()  

<img src="Images/img/2.png">

### Calculating flow accumulation
acc = grid.accumulation(fdir, dirmap=dirmap)  

# Plotting flow accumulation
fig, ax = plt.subplots(figsize=(8,6))  
fig.patch.set_alpha(0)  
plt.grid('on', zorder=0)  
im = ax.imshow(acc, extent=grid.extent, zorder=2,
               cmap='cubehelix',  
               norm=colors.LogNorm(1, acc.max()),
               interpolation='bilinear')  
plt.colorbar(im, ax=ax, label='Upstream Cells')  
plt.title('Flow Accumulation', size=14)  
plt.xlabel('Longitude')  
plt.ylabel('Latitude')  
plt.tight_layout()  

<img src="Images/img/3.png">

# Delineating a catchment

# Specifying pour point
x, y = 225000, 3.689  

# Snapping pour point to high accumulation cell
x_snap, y_snap = grid.snap_to_mask(acc > 1000, (x, y))  

# Delineating the catchment
catch = grid.catchment(x=x_snap, y=y_snap, fdir=fdir, dirmap=dirmap, 
                       xytype='coordinate')  

# Cropping and plotting the catchment

# Clipping the bounding box to the catchment
grid.clip_to(catch)  
clipped_catch = grid.view(catch)  

# Plotting the catchment
fig, ax = plt.subplots(figsize=(12,10))  
fig.patch.set_alpha(0)  

plt.grid('on', zorder=0)  
im = ax.imshow(np.where(clipped_catch, clipped_catch, np.nan), extent=grid.extent,
               zorder=1, cmap='Greys_r')  
plt.xlabel('Longitude')  
plt.ylabel('Latitude')  
plt.xlim(230900, 232000)  
plt.ylim(3.6825*1e6, 3.6835*1e6)  
plt.title('Delineated Catchment', size=10)  

<img src="Images/img/4.png">

# Extracting river network
branches = grid.extract_river_network(fdir, acc > 50, dirmap=dirmap)  


# Plotting with river network

sns.set_palette('husl')  
fig, ax = plt.subplots(figsize=(8.5,6.5))  

plt.xlim(grid.bbox[0], grid.bbox[2])  
plt.ylim(grid.bbox[1], grid.bbox[3])  
ax.set_aspect('equal')  

for branch in branches['features']:  
    line = np.asarray(branch['geometry']['coordinates'])  
    plt.plot(line[:, 0], line[:, 1])  
    
_ = plt.title('D8 channels', size=14)  
_ = plt.xlabel('Longitude')  
_ = plt.ylabel('Latitude')  
_ = plt.xlim(230900, 232000)  
_ = plt.ylim(3.6825*1e6, 3.6835*1e6)  

<img src="Images/img/5.png">

# Calculating distance to outlet from each cell

dist = grid.distance_to_outlet(x=x_snap, y=y_snap, fdir=fdir, dirmap=dirmap,
                               xytype='coordinate')  

# Plotting the distances to the outlet

fig, ax = plt.subplots(figsize=(8,6))  
fig.patch.set_alpha(0)  
plt.grid('on', zorder=0)  
im = ax.imshow(dist, extent=grid.extent, zorder=2, cmap='cubehelix_r')  
plt.colorbar(im, ax=ax, label='Distance to outlet (cells)')  
plt.xlabel('Longitude')  
plt.ylabel('Latitude')  
plt.xlim(230900, 232000)  
plt.ylim(3.6825*1e6, 3.6835*1e6)  
plt.title('Flow Distance', size=14)  

<img src="Images/img/6.png">

# Data Collection Update

# Research Objective and Plan:
* Pick 1 watershed, pick 1 HUC, use 10M DEM and delineate a watershed + Delineate the watershed 
using 1m Lidar data  
* Compare & see the difference between them.   
* Get soil and land use data. Create a decision tool to determine the risk of flooding at each point of 
watershed.  
* Create a map to compare among the watersheds and do risk analysis.  
* Create a dashboard that will layout the soil, land use and risk of flooding data.  
* Use python packages like voila, bokeh; create a python server and deploy the application.  

# Work Flow:
Step 1:
Instruction: Pull out all the watersheds along the coast. Get the HUC 8 & HUC 10 data. Go to 
https://www.twdb.texas.gov/mapping/gisdata.asp and select the 'HUC 8 Shapefile'.
Completed: Downloaded watershed shapefiles for Texas state for HUC 8, 10 & 12 codes.
Step 2:
Instruction: Download Texas County shapefile. Go to - https://gistxdot.opendata.arcgis.com/datasets/TXDOT::texas-county-boundaries-line/about
Completed: Downloaded Texas County shapefile
Step 3:
Instruction: Download Texas National Elevation Dataset. Go to -
https://datagateway.nrcs.usda.gov/GDGorder.aspx
Completed: Downloaded 10M DEM data for the coastal counties of Texas (Aransas, Bee, Brookes, Jim 
Wells, Kenedy, Kleberg, Nueces, Refugio, San Patricio). For 1M Lidar Data, a space amount to 170 GB 
approx. is required (To be downloaded).
For Entire Texas State (Not Downloaded Yet):
LiDAR Elevation Dataset - Bare Earth DEM - 1 Meter, 3470 maps. Space Required - 2016411.659 MB
LiDAR Elevation Dataset - Bare Earth DEM - 2 Meter, 319 maps. Space Required - 31106.479 MB
National Elevation Dataset 10 Meter, 4604 maps. Space Required - 33519.342 MB
Sources Studied:
* Study of Drainage Basin - https://en.wikipedia.org/wiki/Drainage_basin
* Study of HUC Codes - https://digitalatlas.cose.isu.edu/hydr/huc/huctxt.htm
* Study of watershed delinieation - https://www.wvca.us/envirothon/pdf/Watershed_Delineation_2.pdf
* Spatial Analysis dropbox module - https://www.dropbox.com/home/module2-spatial%20analysis
* USGS Elev Data - https://www.usgs.gov/faqs/what-types-elevation-datasets-are-available-whatformats-do-they-come-and-where-can-i-download
* Download Elevation Dataset - https://www.youtube.com/watch?v=VlKkAYoNoRE
* National Map Downloader - https://apps.nationalmap.gov/downloader/#/
* Difference between lidar & digital elevation model - https://www.usgs.gov/faqs/what-differencebetween-lidar-data-and-digital-elevation-model-dem
* Where to download lidar data - https://www.usgs.gov/faqs/what-lidar-data-and-where-can-idownload-it?qt-news_science_products=0#qt-news_science_products
** USDA FAQ - https://datagateway.nrcs.usda.gov/GDGOrder_FAQ.html#cos
