Connectivity
==================

- [Introduction to Connectivity Module](#intro)
- [Quick GUIde to Connectivity Assessment](#guide)

***

# Introduction to Connectivity Module<a name="intro"></a>

The *Connectivity* module is used to identify wetted areas that are disconnected from the river mainstem and characterize the degree to which areas are connected. These tools were developed specifically to assess stranding risk for given aquatic species and lifestages.

# Quick GUIde to Connectivity Assessment<a name="guide"></a>

![mtgui](https://github.com/RiverArchitect/Welcome/raw/master/images/gui_start_connect.PNG)

Select a condition and at least one aquatic ambiance (fish species/lifestage as defined in `Fish.xlsx`. Then click the "Run Analysis" button to begin calculations.

Outputs are stored in `Connectivity\Output\reach_id\`. The outputs produced are:

- `Q_disconnect.tif`: a raster showing the highest model discharge for which areas are disconnected from the main river channel. Locations that are not disconnected at any modeled discharge are assigned a value of zero. Thus, this map indicates locations and discharges below which stranding risks may occur.

- `shortest_paths\`: folder containing a raster for each model discharge indicating the number of pixel steps required to escape to the low flow river mainstem and subject to constraints imposed by the travel thresholds for the selected aquatic ambiance.

More to come...

# Defining Travel Thresholds

Whether or not areas are considered to be connected/navigable for a given fish species/lifestage is dependent upon travel thresholds that are defined in the `Fish.xlsx` workbook (see [[ShArC]] for more details). These include a minimum swimming depth and maximum swimming speed.

# References

Travel thresholds in `Fish.xlsx` were provided by the following sources:

[CDFG. (2012). Standard Operating Procedure for Critical Riffle Analysis for Fish Passage in California DFG-IFP-001, updated February 2013. California Department of Fish and Game, DFG-IFP-00(September), 1–25.](https://nrm.dfg.ca.gov/FileHandler.ashx?DocumentID=150377)

[U.S. Forest Service FishXing Data Table](http://www.fsl.orst.edu/geowater/FX3/help/SwimData/swimtable.htm) (various sources)
