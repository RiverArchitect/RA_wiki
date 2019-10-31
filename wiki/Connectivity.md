Habitat Connectivity
====================

***

- [Introduction to Connectivity Module](#intro)
- [Quick GUIde to Connectivity Assessment](#guide)
- [Defining Travel Thresholds](#defining-travel-thresholds)
- [Methodology](#methodology)
  * [Interpolating Hydraulic Rasters](#interpolating-hydraulic-rasters)
  * [Escape Route Calculations](#escape-route-calculations)
  * [Calculating Disconnected Habitat Area](#calculating-disconnected-area)
  * [Determining Q<sub>disconnect</sub>](#determining-qdisconnect)
- [References](#references)

***

# Introduction to Connectivity Module<a name="intro"></a>

The *Connectivity* module can be used to identify the following for a given flow reduction scenario: 
- wetted areas that become disconnected from the river mainstem in the event of a flow reduction
- the discharges at which specific areas disconnect
- stranding risk associated with each disconnected area
- the degree to which wetted areas are connected before a complete disconnection occurs.

These tools were developed to assess stranding risk for any given aquatic species and lifestage.

# Quick GUIde to Connectivity Assessment<a name="guide"></a>

***

![mtgui](https://github.com/RiverArchitect/Media/raw/master/images/gui_start_connect.PNG)

To begin using the Connectivity module, first select a hydraulic [Condition](Signposts#conditions). In order to accurately determine where velocity is a barrier to fish passage, the selected condition must include velocity angle input rasters. In order to estimate stranding risks, cHSI rasters must have already been calculated for the input Condition using the [SHArC](SHArC) module.

Next, select at least one [Physical Habitat](SHArC#hefish) (fish species/lifestage) from the dropdown menu. The Physical Habitat contains data specific to the fish species/lifestage. Physical Habitat data is used by the Connectivity module to determine if fish are able to traverse wetted areas by accounting for the organism's minimum swimming depth and maximum swimming speed (Physical Habitat data are also used by [SHArC](SHArC) to determine habitat suitability). These data can be viewed/edited via the drop-down menu: `Select Physical Habitat `  --> `DEFINE FISH SPECIES` (scroll to the "Travel Thresholds" section of the workbook).

Once the desired condition and Physical Habitat(s) are selected, choose model discharges Q<sub>high</sub> and Q<sub>low</sub>. This defines the range of discharges over which to apply the connectivity analysis, simulating the changes in habitat connectivity for a flow reduction from Q<sub>high</sub> to Q<sub>low</sub>.

Lastly, select an interpolation method. See [Interpolating Hydraulic Rasters](#interpolating-hydraulic-rasters) for more information.

For further explanation of the methodology used in the analysis, see [Methodology](Connectivity#Methodology).

Outputs are stored in `Connectivity\Output\Condition_name\`. The outputs produced are:

- interpolated rasters (`h_interp`, `u_interp`, `va_interp`): interpolated depth, velocity (magnitude), and velocity angle rasters. See [Interpolating Hydraulic Rasters](Connectivity#interpolating-hydraulic-rasters) for more information.'

Outputs specific to the applied flow reduction are stored in the subdirectory `Connectivity\Output\Condition_name\flow_red_Qhigh_Qlow`. These outputs include:

- `shortest_paths\`: directory containing a raster for each model discharge in the range Q<sub>low</sub>-Q<sub>high</sub>, indicating the minimum distance/least cost required to escape to the river mainstem at Q<sub>low</sub>, subject to constraints imposed by the travel thresholds for the selected Physical Habitat. See [Escape Route Calculations](Connectivity#escape-route-calculations) for more.

- `disc_areas\`: directory containing a shapefile for each model discharge indicating wetted areas which are effectively disconnected at that discharge. See [Calculating Disconnected Habitat Area](Connectivity#calculating-disconnected-area) for more.

- `disconnected_area.xlsx`: a spreadsheet containing plotted data of discharge vs disconnected area.

- `disconnected_habitat.tif`: a raster showing all wetted areas which become disconnected in the applied flow reduction scenario, weighted by the combined habitat suitability index (cHSI) at Q<sub>high</sub> (before the flow reduction occurs). See [Calculating Disconnected Habitat Area](Connectivity#calculating-disconnected-area) for more.

- `Q_disconnect.tif`: a raster showing the highest model discharge for which areas are disconnected from the mainstem at Q<sub>low</sub>. Locations that are not disconnected at any modeled discharge are assigned a value of zero. Thus, this map indicates locations and discharges below which stranding risks may occur. See [Determining Q<sub>disconnect</sub>](Connectivity#determining-qdisconnect) for more.

***

# Defining Travel Thresholds

***

Whether or not areas are considered to be connected/navigable for a given fish species/lifestage is dependent upon travel thresholds that are defined in the `Fish.xlsx` workbook (see [SHArC](SHArC) for more details). These include a minimum swimming depth and maximum swimming speed. To view/modify the species/lifestage specific travel thresholds, use the drop-down menu: "Select Physical Habitat" --> "DEFINE FISH SPECIES".

***

# Methodology

***

## Interpolating Hydraulic Rasters

The analysis begins by performing an interpolation of the water surface elevation across the extent of the DEM. This step is taken in order to approximate how the water surface extends across floodplain areas, thus identifying the presence of disconnected pools that would not be captured by steady-state hydrodynamic model outputs. The interpolation is performed for each discharge as follows:

- The water depth raster is added to the DEM elevation to produce a water surface elevation (WSE) raster.
- the WSE is interpolated across the DEM extent using an Inverse Distance Weighted (IDW) interpolation scheme on the 12 nearest neighbors.
- The DEM is subtracted from the interpolated WSE raster to produce an interpolated depth raster. Only positive values are saved (negative values indicate an estimated depth to groundwater).
- Interpolated velocity and velocity angle rasters are created, where velocity is set to zero in the newly interpolated areas.
- Interpolated rasters are stored in `Connectivity\Condition_name\h_interp`, `Connectivity\Condition_name\u_interp`, `Connectivity\Condition_name\va_interp`. All interpolated rasters are saved with a corresponding .info.txt file which records the interpolation method and input rasters used in its creation.

The available interpolation methods are:

- `IDW`: Inverse distance weighted interpolation. Each null cell in the low flow depth raster is assigned a weighted average of its 12 nearest neighbors. Each weight is inversely proportional to the second power of the distance between the interpolated cell and its neighbor. Thus, the closest neighbors to an interpolated cell receive the largest weights. This method was found to have the best balance of computational speed and accuracy when tested via cross-validation.
- `Kriging`: Ordinary Kriging interpolation. This method also uses a weighted average of the 12 nearest neighbors, but weights are calculated using a semi-variogram, which describes the variance of the input data set as a function of the distance between points. A spherical functional form is also assumed for the fitted semi-variogram. Kriging is the most accurate interpolation method if certain assumptions are met regarding normality and stationarity of error terms. However, it is also the most computationally expensive method. This method was found to have comparable accuracy compared to IDW, but with significantly higher computational costs.
- `Nearest Neighbor`: The simplest method, which sets the water surface elevation of each null cell to that of the nearest neighboring cell.

## Escape Route Calculations

The "shortest escape route" output maps (stored in the `shortest_paths\` output directory) show the length/cost of the shortest/least-cost path from each pixel back into the mainstem of the river channel at Q<sub>low</sub>, accounting for both depth and velocity travel thresholds. In this context, the mainstem is defined as the largest continuous portion of interpolated wetted area deeper than the minimum swimming depth at Q<sub>low</sub>. Conceptually, the shortest escape route to the mainstem is found from each starting pixel as follows:

- If the starting pixel is already in the mainstem, it assigned a default value of zero. Otherwise, look at all wetted pixels adjacent to the starting pixel.
- Apply depth and velocity travel criteria (see below).
- If both the depth and velocity travel criteria are satisfied, the fish is considered to be able to move from the current pixel to the neighbor. An associated cost of traveling to the neighboring cell can also be computed.
- Apply this method iteratively to each reachable neighbor. The path which reaches a cell in the mainstem with the smallest total cost is the least-cost path to the mainstem.

***

### Applying Depth and Velocity Travel Criteria

![connect_vel](https://github.com/RiverArchitect/Media/raw/master/images/connect_vel_condition.png)

The depth threshold criteria is applied simply by checking if the current cell and neighboring cell are greater than the minimum swimming depth. If so, the depth threshold criteria is satisfied. Because velocity is a vector quantity, the direction of travel must be considered when applying the velocity threshold criteria.

From the center of each cell, the area is divided into 8 octants corresponding to the 8 neighboring cells, with each octant centered on the direction to its neighboring cell and spanning 45°. If it is possible to add the water velocity vector (𝑣<sub>𝑤</sub>) to a vector with the magnitude of the maximum swimming speed (𝑣<sub>𝑓</sub>) to yield a vector (𝑉) falling within an octant, then the velocity threshold criteria is satisfied for travel to the corresponding neighboring cell. A specific example is shown in this diagram where the resultant velocity vector 𝑉 points into the upper quadrant, thus the velocity threshold criteria is satisfied for travel to the upper neighbor. Octants are colored red/green based on whether the criteria is/is not satisfied for travel in that direction.

***

Least cost path calculations are implemented in a computationally efficient way by constructing a weighted, directed adjacency graph (stored as a dictionary) and then dynamically traversing the graph working outwards from the mainstem to find shortest path lengths using Dijkstra's algorithm (see explanation below):

![connect_graph](https://github.com/RiverArchitect/Media/raw/master/images/connect_graph.png)

Applying the depth and velocity thresholds at each cell yields a set of neighboring cells for which travel is possible. Each cell which can reach other cells or be reached is represented as a vertex of the graph. Reachability of neighboring vertices is represented by edges connecting the vertices, with an arrow symbol indicating the direction of possible travel (edges without arrows indicate travel is possible in both directions). For each edge, an associated cost of traveling along the edge is calculated, thus yielding a weighted digraph to represent possible fish travel and the associated costs. The least-cost path leading from each vertex to any of the vertices in the target area (corresponding to river mainstem at a lower discharge) is then computed and the path length/cost is stored as a value in the output raster at the location of the starting vertex. Here the target vertices are shown in gold and the least-cost path from point A is shown in green (where the cost function applied is Euclidean distance).

***

## Calculating Disconnected Habitat Area

Even if there is wetted area connecting two locations, they may not be considered connected in the context of fish passage. This is because low water depths or high velocities may effectively act as "hydraulic barriers", limiting fish mobility. The applied Physical Habitat contains threshold values for the minimum swimming depth and maximum swimming speed. After escape routes are calculated for a given discharge, the resultant path length raster shows which areas are able to reach the river mainstem for the given hydraulic conditions and biological limitations. Wetted areas in the corresponding interpolated depth raster for which a least-cost path cannot be calculated are considered to be disconnected areas. These disconnected areas are then cropped within the wetted area at Q<sub>high</sub> (not interpolated), as floodplains outside the Q<sub>high</sub> wetted area are assumed to not be active for the simulated flow reduction. Resultant disconnected areas are saved to a shapefile for each discharge in the `disc_areas` output directory.

Once disconnected areas have been calculated, they are weighted by the combined habitat suitability index (cHSI) at Q<sub>high</sub> for the target Physical Habitat. The resultant raster is stored as `disconnected_habitat` in the output directory. Since cHSI serves as a proxy for fish density in habitat-limited environments, the cHSI at Q<sub>high</sub> in areas that become disconnected by the flow reduction serves as a spatially-distributed stranding risk metric.

## Determining Q<sub>disconnect</sub>
  
The `Q_disconnect` map outputs provide estimates of the discharges at which wetted areas become disconnected from the mainstem of the river channel. Estimating these discharges is done as follows:

- Assign all pixels wetted at the highest discharge a default value of zero.
- Iterating over discharges in ascending order, pixels within disconnected area polygons are assigned a value of the corresponding discharge.
- The resultant raster is saved as the `Q_disconnect.tif` map output.

Note that the default value of zero indicates pixels wetted at the highest discharge but not present within any of the disconnected area polygons. Other values indicate the highest modeled discharge for which the pixel is disconnected from the channel mainstem.

***

# References

Travel thresholds used to define Physical Habitats in `Fish.xlsx` were provided by the following sources:

[CDFG. (2012). Standard Operating Procedure for Critical Riffle Analysis for Fish Passage in California DFG-IFP-001, updated February 2013. California Department of Fish and Game, DFG-IFP-00(September), 1–25.](https://nrm.dfg.ca.gov/FileHandler.ashx?DocumentID=150377)

[U.S. Forest Service FishXing Data Table](http://www.fsl.orst.edu/geowater/FX3/help/SwimData/swimtable.htm) (various sources)
