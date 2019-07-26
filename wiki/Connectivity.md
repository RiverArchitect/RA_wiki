Habitat Connectivity
====================

***

- [Introduction to Connectivity Module](#intro)
- [Quick GUIde to Connectivity Assessment](#guide)
- [Defining Travel Thresholds](#defining-travel-thresholds)
- [Methodology](#methodology)
  * [Interpolating Hydraulic Rasters](#interpolating-hydraulic-rasters)
  * [Applying Travel Thresholds and Calculating Disconnected Area](#applying-travel-thresholds-and-calculating-disconnected-area)
  * [Determining Q<sub>disconnect<sub>](#determining-qdisconnect)
  * [Escape Route Calculations](#escape-route-calculations)
- [References](#references)

***

# Introduction to Connectivity Module<a name="intro"></a>

The *Connectivity* module is used to identify wetted areas that are disconnected from the river mainstem and characterize the degree to which areas are connected. These tools were developed specifically to assess stranding risk for given aquatic species and lifestages.

# Quick GUIde to Connectivity Assessment<a name="guide"></a>

***

![mtgui](https://github.com/RiverArchitect/Welcome/raw/master/images/gui_start_connect.PNG)

To begin using the Connectivity module, first select a hydraulic [Condition](Signposts#conditions). In order to accurately determine where velocity is a barrier to fish passage, the selected condition must include velocity angle input rasters.

Next, select least one [aquatic ambiance](SHArC#hefish) (fish species/lifestage) from the dropdown menu. The aquatic ambiance contains data specific to the fish species/lifestage. Aquatic ambiance data is used by the Connectivity module to determine if fish are able to traverse wetted areas by accounting for the organism's minimum swimming depth and maximum swimming speed (aquatic ambiance data is also used by [SHArC](SHArC) to determine habitat suitability). These data can be viewed/edited via the drop-down menu: `Select Aquatic Ambiance `  --> `DEFINE FISH SPECIES` (scroll to the "Travel Thresholds" section of the workbook).

Once the desired condition and aquatic ambiance(s) are selected, the module can be run with the "Run Connectivity Analysis" button. For further explanation of the methodology used in the analysis, see [Methodology](Connectivity#Methodology).

Outputs are stored in `Connectivity\Output\Condition_name\`. The outputs produced are:

- `Q_disconnect.tif`: a raster showing the highest model discharge for which areas are disconnected from the main river channel. Locations that are not disconnected at any modeled discharge are assigned a value of zero. Thus, this map indicates locations and discharges below which stranding risks may occur.

- `shortest_paths\`: folder containing a raster for each model discharge indicating the minimum distance/least cost required to escape to the low flow river mainstem and subject to constraints imposed by the travel thresholds for the selected aquatic ambiance.

More to come...

***

# Defining Travel Thresholds

***

Whether or not areas are considered to be connected/navigable for a given fish species/lifestage is dependent upon travel thresholds that are defined in the `Fish.xlsx` workbook (see [SHArC](SHArC) for more details). These include a minimum swimming depth and maximum swimming speed.

***

# Methodology

***

## Interpolating Hydraulic Rasters

The analysis begins by performing an interpolation of the water surface elevation across the extent of the DEM. This step is taken in order to approximate how the water surface extends across floodplain areas, thus identifying the presence of disconnected pools that would not be captured by steady-state hydrodynamic model outputs. The interpolation is performed for each discharge as follows:

- The water depth raster is added to the DEM elevation to produce a water surface elevation (WSE) raster.
- the WSE is interpolated across the DEM extent using Ordinary Kriging interpolation (12 nearest neighbors, Gaussian semivariogram). Only 12 nearest neighbors are used for each interpolated pixel to make the interpolation computationally affordable, and a Gaussian semivariogram model is used by default since it produces the best fit for the hydrodynamic data for the lower Yuba River used during testing and development.
- The DEM is subtracted from the interpolated WSE raster to produce an interpolated depth raster. Only positive values are saved (negative values indicate an estimated depth to groundwater).
- Interpolated velocity and velocity angle rasters are created, where velocity is set to zero in the newly interpolated areas.
- Interpolated rasters are stored in `Connectivity\Condition_name\h_interp`, `Connectivity\Condition_name\u_interp`, `Connectivity\Condition_name\va_interp`.

## Applying Travel Thresholds and Calculating Disconnected Area

Even if there is wetted area connecting two locations, they may not be considered connected in the context of fish passage. This is because low water depths or high velocities may effectively act as "hydraulic barriers", limiting fish mobility. The applied aquatic ambiance contains threshold values for the minimum swimming depth and maximum swimming speed. These thresholds are applied as follows:

- The interpolated depth raster is set to null for values less than the minimum swimming depth.
- The resultant raster is converted into a set of polygon objects.
- Areas of all the polygons are calculated. The polygon with the greatest area is assumed to be the mainstem of the river channel. All other polygons are considered to be disconnected wetted areas.

For now, this calculation of disconnected areas only applies the depth threshold. The velocity threshold is incorporated into escape route calculations, but will also need to be used to update this calculation in the future.

## Determining Q<sub>disconnect
  
The `Q_disconnect` map outputs provide estimates of the discharges at which wetted areas become disconnected from the mainstem of the river channel. Estimating these discharges is done as follows:

- Assign all pixels wetted at the highest discharge a default value of zero.
- Using the polygon sets produced by the previous step, the mainstem polygon is removed for each discharge, Thus producing a set of disconnected area polygons for each discharge.
- Iterating over discharges in ascending order, pixels within disconnected area polygons are assigned a value of the corresponding discharge.
- The resultant raster is saved as the `Q_disconnect` map output.

Note that the default value of zero indicates pixels wetted at the highest discharge but not present within any of the disconnected area polygons. Other values indicate the highest modeled discharge for which the pixel is disconnected from the channel mainstem.

## Escape Route Calculations

The "shortest escape route" output maps (stored in the `shortest_paths\` output directory) show the length/cost of the shortest/least-cost path from each pixel back into the mainstem of the river channel, accounting for both depth and velocity travel thresholds. In this context, the mainstem is defined as the largest continuous portion of interpolated wetted area deeper than the minimum swimming depth at the lowest available discharge. The shortest escape route to the mainstem is found from each starting pixel as follows:

- If the starting pixel is in the mainstem, it assigned a default value of zero. Otherwise, look at all pixels adjacent to the starting pixel.
- Apply depth condition: check that the depth is greater than the minimum swimming depth in both the current pixel and the neighboring pixel.
- Apply velocity condition: calculate the water velocity vector at the current pixel (using velocity magnitude and angle). Check if a "swimming vector" (vector with the magnitude of the maximum swimming speed) can be oriented in such a way that when it is added to the water velocity vector, the resultant vector is oriented in the direction from the current pixel to the neighboring pixel.
- If both the depth and velocity conditions are satisfied, the fish is considered to be able to move from the current pixel to the neighbor.
- Applying this method iteratively to each pixel allows us to determine the shortest path to the mainstem.

This calculation is implemented in a computationally efficient way by first constructing a directed adjacency graph (stored as a dictionary) and then dynamically traversing the graph working outwards from the mainstem to find shortest path lengths.

Currently this method only provides the shortest number of "steps" from pixel to pixel to get to the mainstem. Future developments will include the option to convert these to shortest path lengths or a least-cost path, according to user-selected cost functions.

***

# References

Travel thresholds used to define aquatic ambiances in `Fish.xlsx` were provided by the following sources:

[CDFG. (2012). Standard Operating Procedure for Critical Riffle Analysis for Fish Passage in California DFG-IFP-001, updated February 2013. California Department of Fish and Game, DFG-IFP-00(September), 1–25.](https://nrm.dfg.ca.gov/FileHandler.ashx?DocumentID=150377)

[U.S. Forest Service FishXing Data Table](http://www.fsl.orst.edu/geowater/FX3/help/SwimData/swimtable.htm) (various sources)
