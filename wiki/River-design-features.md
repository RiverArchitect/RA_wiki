[Feature lifespan and design assessment][3]
======================================

***

- [Quick GUIde to lifespan and design maps][3]
- [Parameter hypotheses](LifespanDesign-parameters)
- **River design and restoration features**
  * [Introduction and predefined features](#featoverview)
  * [Details and parameters of pre-defined features](#feat)
    + [Angular boulders (rocks) and grain mobility](#rocks)
    + [Backwater](#backwtr)
    + [Bioengineering](#bioeng)
    + [Berm Setback / Widen](#berms)
    + [Streamwood and Engineered Log Jams](#elj)
    + [Fine sediment](#finesed)
    + [Grading](#grading)
    + [Vegetation Plantings](#plants)
    + [Sediment replenishment / gravel augmentation](#gravel)
    + [Side cavities](#sidecav)
    + [Side channels / anabranches](#sidechnl)
  * [Instructions for defining new and modifying existing features](#modfeat)
- [Code extension and modification](LifespanDesign-code)

***
# Introduction and Feature Groups<a name="featoverview"></a>

The *River Architect* differentiates between feature layers that actively modify the terrain (terraforming features), vegetation plantings features as well as (soil-) bioengineering features that provide direct aid for habitat enhancement or stabilize terrain modifications, and features that maintain artificially created habitat or support longitudinal connectivity. Feature attributes can be modified in the thresholds workbook (`RiverArchitect/LifespanDesign/.templates/threshold_values.xlsx`), which can also be open from the GUI:

![thresholds](https://github.com/RiverArchitect/Welcome/raw/master/images/threshold_values_illu.png)

Changes in this workbook should limit to cells with `INPUT`-type formatting and only `Feature Names` and `FeatureID`s of vegetation plantings should be modified. Other modifications may cause calculation instabilities or program crashes. The following list provides an overview of default features, where *shortname*s occur in output file names of Rasters, layouts, PDF-maps, and spreadsheets and plantings

-   **Terraforming features** modify the terrain elevation:
    -   Backwater, representative for swale and slackwater creation (*shortname: backwt*)
    -   Berm Setback (Widening, *shortname: widen*)
    -   Grading of terrain (Bar and Floodplain Lowering *shortname: grade*)
    -   Side Cavities (Bank Scalloping or Groins, *shortname: sideca*)
    -   Side Channels, representative for Anabranches, Multithread- or Anastomosed Channels and Flood Runners (*shortname: sidech*)
-   **Plantings features** include up to four plant species. The pre-defined plant species are ([can be modified](#modfeat), except for the fields that are marked for input in the thresholds workbook):
    -   (Fremont) Cottonwood (*Populus Fremontii*, *shortname: cot*)
    -   Box Elder (*Acer Negundo*, *shortname: box*)
    -   White Alder (*Alnus Rhombifolia*, *shortname: whi*)
    -   Willows (*Salix Goodingii / laevigata / lasiolepis / alba*, *shortname: wil*)

-   **Other bioengineering features** have a direct effect on habitat suitability and stabilize terrain modifications (framework features). These features are considered:
    -   Streamwood, engineered log jams (ELJ) and large woody material (LWM) placement including rootstocks (*shortname: elj*)
    -   Grains / Boulders (angular), representative for coarse material placement (*shortname: grains*)
    -   Other soil-bioengineering for terrain (slope) stabilization comprises for instance brush layers and / or fascines (*shortname: bio*)

-   **Connectivity features** enhance the stability of (artificial) river systems that result from the above feature applications:
    -   Sediment Replenishment for restoring sediment continuity (instream, *shortname: gravin*)
    -   Stockpiles of gravel or Gravel Augmentation  for restoring sediment continuity(on banks or floodplain, *shortname: gravou*)
    -   Incorporation of Fine Sediment in soils to increase the survivorship of plantings (*shortname: fines*)   
    -   Flow controls are only indirectly considered in the [*SHArC*][6] module, where discharges drive habitat suitability
    -   Dam removal and the removal of other lateral barriers (e.g., ground sills) are currently not considered in *River Architect*

In addition, the *River Architect* provides the option of limiting restoration feature maps to zones of low habitat suitability (see details in the descriptions of the [*SHArC*][6] module).


# Predefined features and hypotheses<a name="feat"></a>

The standard installation of *River Architect* comes with a set of pre-defined river design features (cf. [Schwindt et al. 2019][schwindt19]). For topographic change, scour and fill rates are considered over a 3-year observation period (2008 to 2014, see [Weber and Pasternack 2017][weber17]). The following hypotheses apply to the pre-defined features.

## Grains, Angular boulders and grain mobility<a name="rocks"></a>

The punctual placing of boulders and comprehensive rock cover is referred to as "angular boulders" for stabilizing banks or erosion-prone surfaces (e.g., [Maynord and Neill 2008][maynord08]). The mobility of the present terrain refers to the present grain size and indicates the necessity of boulder placement on the basis of lifespan maps. The required minimum diameter for boulders or mobile grains results from the spatial evaluation of *D<sub>cr</sub>* on mobile grain design maps, where the following parameters apply:

 **Lifespan Maps**

-   `taux` with mobility threshold of *&tau;<sub>\*,cr</sub>* equal to 0.047.
-   `tcd` - `scour` with a threshold value of 0.3 m (1 ft) multiplied with the number of observation years (if a topographic change raster includes scour / fill observed over multiple years).

**Design Maps**

-   `stable_grains` for design maps (see below formulae), with a frequency threshold of 20.0 years (sample case) and *&tau;<sub>\*,cr</sub>* threshold of `taux` = 0.047.

The minimum required grain sizes are determined in a two-way analysis (i.e., two minimum angular boulders (rocks) size maps are produced based on the highest discharge where hydraulic data is available):

1.  `ds_grains_Dcr` is a derivative of the Gauckler-Manning-Strickler formula using [Manning\'s *n*][manningsn]:\
    *D<sub>cr</sub>* =  *SF* *· u<sup>2</sup> · n<sup>2</sup>* / *\[(s - 1) · h<sup>1/3</sup> · &tau;<sub>\*,cr</sub> \]*

2.  `ds_grains_Dcr` is a derivative of the Chézy formula using the energy slope:\
    *D<sub>cr</sub>* = *SF* *· h · S<sub>e</sub>* / *\[(s - 1) · &tau;<sub>\*,cr</sub> ]*

where:

- *D<sub>cr</sub>* is the minimum required angular boulders (rocks) size (in m or inches)
- *h* is the flow depth (pixel-wise, in m or ft);
- *n* is [Manning\'s *n*][manningsn] can be changed in the GUI (in s/ft<sup>1/3</sup> or s/m<sup>1/3</sup> - an internal conversion factor of k = 1.49 applies in the case of the US customary system)
- *s* is the dimensionless relative grain density (ratio of sediment and water density, equal to 2.68)
- *S<sub>e</sub>* is the energy slope (derived from 's "Slope" function, dimensionless)
- *SF* is a (modifiable) safety factor equal to 1.3 (dimensionless)
- *u* is the flow velocity (pixel-wise, in m/s or fps);
- *&tau;<sub>\*,cr</sub>* is the threshold value of dimensionless bed shear stress for incipient grain motion, equal to 0.047.

The energy slope maps result from computing the theoretic energy height maps as `ras_energy` = `dem.raster` + `h.raster` + `u.raster`<sup>2</sup>/(2 *g*), where *g* denotes gravitational acceleration and the hydraulic rasters correspond to the highest modeled discharge.



## Backwater<a name="backwtr"></a>

The creation of artificial backwaters and swales, or more generally calm water zones, makes sense where the stream power is low and the observed topographic changes are small. The following parameters identify relevant areas for backwater creation:

-   `u` with a threshold of 0.03 m/s (0.1 fps).
-   `mobile_grains` with a frequency threshold of 5 years (in the sample case) 
-   `taux` with a threshold of *&tau;<sub>\*,cr</sub>* threshold of 0.047.
-   `tcd` with `scour` and `fill` thresholds of >= 0.03 m (0.1 ft) multiplied with the number of observation years (if a topographic change raster includes scour / fill observed over multiple years).
-   `mu` using the inclusive method with `mu_good = ["agriplain", "backswamp", "mining pit", "pond", "pool", "slackwater", "swale"].`

## Bioengineering<a name="bioeng"></a>

Areas with a 1.0-year lifespan require bioengineering features that are independent of the depth to the groundwater table because plantings likely will not have sufficient water to survive. Such features typically imply the placement of angular boulders (see [angular boulders](#rocks)).

In the context of river engineering, soil-bioengineering applies living materials (plants) to stabilize terrain and enhance habitat. Alas, dry conditions in arid and semi-arid (Mediterranean) climate zones limits the possibilities of application. The *LifespanDesign* module maps potential bioengineering areas, as a function of:

-   `d2w` the maximum depth to groundwater distance indicates where [vegetation plantings](#plants)-based bioengineering applies.
-   `dem` is used to compute the percent-wise terrain slope `S0`, where modified terrain with slopes of more than a certain percentage is considered to require reinforcement (set `S0` threshold in `RiverArchitect/LifespanDesign/.templates/threshold_values.xlsx`, see [threshold modification](LifespanDesign#input-modify-threshold-values))

The **lifespan maps** of bioengineering features can take three values:

-   `20.0` years (or maximum value as defined in the [input definitions file](Signposts#inpfile), if the terrain slope is greater than defined in the thresholds workbook and the depth to groundwater is lower than defined in the [thresholds workbook](LifespanDesign#input-modify-threshold-values)
-   `1.0` year, if the terrain slope is greater than defined in the thresholds workbook and the depth to groundwater is greater than defined in the thresholds workbook
-   `NoData`, if the terrain slope is lower than defined in the thresholds workbook.

## Berm Setback / Widen<a name="berms"></a>

Berms are artificial lateral confinements that are represented by man-made bars and dikes. Also, levees represent lateral confinement but their flood protection-function should not be deleted, and therefore, levees are not considered for setback action. The code replaces the keyword "Bermsetback" with "Widen" because the removal of lateral confinements represents a river widening.

-   `mu` using the inclusive method with `mu_good = ["bank", " floodplain ", "high floodplain", "island floodplain ", "island high floodplain ", " lateral bar", "levee", "spur dike", "terrace"]`.
-   `det` detrended DEM with a lower limit of 5.2 m (17 ft) and an upper limit of 7.6 m (25 ft).

The detrended DEM (`det`) limits the application of berm setback and widening to economically reasonable extents, where the `det` limits in the code are empiric values and may differ from case to case.

## Streamwood and Engineered Log Jams<a name="elj"></a>

Lifespan maps and design maps are created for streamwood placement and engineered log jams (ELJs), where the following parameters apply:

**Lifespan Maps**

-   `h` with mobility threshold of 1.7 multiplied with the log diameter of 2 ft (0.6 m) (see details in [Lange and Bezzola 2005)][lange05].
-   `Fr` with a threshold of 1 (critical flow conditions).
-   `mu` excluding tributary sections (see below descriptions).

**Design maps**

-   `h` is used to computed the minimum required log diameter to avoid motion for an *n*-years flood (sample case: *n* = 10).

Regarding morphological units, riffle-pool and plane bed morphologies are favorable for streamwood placement, where side channel and tributary systems are not convenient for wood placement. Streamwood inclusive list is defined as\
`mu_good = ["riffle", "riffle transition", "pool", "floodplain", "island floodplain", "lateral bar", "medial bar", "run"]`\
and the exclusive list is defined as `mu_bad = ["tributary channel", "tributary delta"]`.\
For streamwood, the exclusive approach based on `mu_bad` applies (see [parameters](LifespanDesign-parameters)).\
The design maps for the minimum required log diameter *D<sub>w</sub>* results from [Ruiz-Villanueva et al. (2016)][ruiz16b]'s interpolation curve as a function of the flow depth. The module applies on the single-thread formula because it returns larger values for the log diameter than the multi-thread formula when the probability of motion is set to zero: `Dw` = 0.32 / 0.18·`h`. The output map limits to regions where `Dw` is smaller than 7.6 m (300 in).

## Fine sediment<a name="finesed"></a>

Artificially introduced fine sediment facilitates root growth of new plantings but the flow may easily entrain (mobilize) artificially placed fine sediment. Moreover, spontaneous percolation of fine sediment into the voids of the coarser existing sediment may occur. Therefore, plantings-specific parameters apply to the introduction of fine sediment, as well as filter criteria. The analysis considers fine sediment with a maximum grain diameter of 2 mm (0.08 in, i.e., sand). The module uses the following raster criteria:

 **Lifespan Maps**

-   `taux` with a threshold of *&tau;<sub>\*,cr</sub></sub>* = 0.030.
-   `Dcf` is the maximum admissible size of fine sediment including the (D<sub>max, fine</sub> < 2 mm or 0.08 in) that results from [grain mobility](#rocks).
-   `tcd` 
	+ `scour` threshold of [White Alder](#plants) (largest for plantings) of 0.3 m (1.0 ft) multiplied with the number of years of terrain change observation 
	+ `fill` threshold of [Cottonwood](#plants) (highest for plantings) of 0.8 *times the seedling length in m or ft* and multiplied with the number of years of terrain change observation (the sample case uses Cottonwood with a planting depth of 80\%, a cutting length of 7 ft (2.1 m) and a topographic change observation period of 3 years)
-   `d2w` with a lower limit of 0.3 m (1 ft) and an upper limit of 3.0 m (10 ft) corresponding to vegetation [plantings](#plants) requirements.

**Design Maps**

-   `filter` criteria resulting in a design map according to the [USACE (2000)][usace00]:\
    *D<sub>15, fine</sub> > D<sub>15, coarse</sub>* / 20;\
    *D<sub>85, fine</sub> > D<sub>15, coarse</sub>* / 5;\
    *D<sub>max, fine</sub>* must be finer than sand (i.e., *<* 2 mm or 0.08 in), to satisfy its *fine* character.

The topographic change and depth to water table thresholds correspond to the largest values that any plantings type (cf.
[Vegetation Plantings](#plants)) supports because only these areas are of interest for the incorporation of fine sediment in soils.

## Grading<a name="grading"></a>

Grading aims at the reconnection of high floodplains and isolated islands by means of floodplain terracing and bar lowering. Grading is from an interest in areas where potential plantings cannot reach the groundwater table or where even high discharges cannot rework the channel. Low dimensionless shear stress, infrequent grain mobilization or low scour rates indicate relevant sites. The following parameter Rasters and hypotheses apply to lifespan maps for grading measures (no design maps).

-   `taux` with mobility threshold of *&tau;<sub>\*,cr</sub>* equal to 0.047.
-   `tcd` - `scour` with a threshold value of 0.03 m (0.1 ft) multiplied with the number of observation years documented with the `scour` raster, and the `inverse` argument (i.e., areas of interest correspond to regions where the scour threshold is not exceeded).
-   `d2w` with lower and upper limits corresponding to target [plant species](#plants).

Further aspects may be considered in addition to the implemented parameters:

-   Depth to groundwater\
	 	Enable new [Vegetation Plantings](#plants) in Mediterranean climates to achieve depths to groundwater between 2.1 m (7 ft) and 3.0 m (10 ft).
-   Morphological Units\
    *Not applied in the sample case, but can be optionally enabled.*

## Plantings<a name="plants"></a>

The survival analysis of plantings assumes a general cutting length of min. 2.1 m (7 ft), where approximately 80*\%* of the cuttings are planted in the ground and 20*\%* protrude above the ground. The *<a href="LifespanDesign">LifespanDesign</a>* module enables the differentiation between up to four indigenous plant species that are relevant for habitat enhancement. The installation contains threshold parameters for **lifespan maps** for the following plant species native to Northern California (referred to as "sample case"). The depth to the groundwater `d2w` thresholds correspond to naturally observed occurrences. **No design maps** are created because the lifespan maps already contain all relevant information.

-   Box Elder (*Acer Negundo*)
	+  `h` (exclude all submerged regions for more than *Q<sub>sub</sub>*, which persists for more than 85 consecutive days according to [Friedman and Auble 1999][friedman99])
	+  `taux` with threshold value of 0.047 ([Friedman and Auble 1999][friedman99])
	+  `tcd` is *none* because Box Elder is reported to survive burial ([Kui and Stella 2016][kui16a])
	+  `d2w`  with lower and upper thresholds of 0.6 m (2 ft) to 2.0 m (6 ft), respectively.\
	   *Full source ranges: 0.6 m to 4.6 m ([Stella et al. 2003][stella03])*   
      	+  *Note:* the maximum submergence duration supported by Box Elder cuttings is 85 days per year. Therefore, the *LifespanDesign* module uses the discharge that is exceeded during 85 days per year (*Q<sub>sub</sub>*) to limit Box Elder plantings to regions that are not inundated at that discharge 85-days submergence criterion.

-   Cottonwood (*Populus Fremontii*)
	+  `hyd` with `h` >= 0.5 · *stem height* and `u` >= 0.9-1.2 m/s (3.0-4.0 fps) ([Stromberg et al. 1993][stromberg93], [Wilcox and Shafroth 2013][wilcox13], [Bywater-Reyes 2015][bywater15])
	+  `tcd` with 
	   * `scour` >= 0.1 *times root depth* ([Polzin and Rood 2006][polzin06]), or >= 0.2 *times root depth* ([Kui and Stella 2016][kui16a]), or >= 0.5 *times root depth* ([Bywater-Reyes 2015][bywater15])
	   * `fill` >= 0.8 *times seedling length* ([Kui and Stella 2016][kui16a], [Polzin and Rood 2006][polzin06])
	+  `d2w` with a lower threshold of 5 ft (1.5 m) and an upper threshold of 10 ft (3 m)\
	   *Full source ranges: 0.6 m to 4.6 m ([Stella et al. 2003][stella03]), 0.6 m to 2.7 m ([SYRCL 2013][syrcl13]), 0.6 m to 2.8 m above baseflow level for the closely-related black cottonwood (*Populus trichocarpa*, extracted from [Polzin and Rood 2006][polzin06]), 0.23±0.08 m to 1.58±0.14 m for establishment of woody riparian species and up to 2.9 m for existing plants ([Shafroth et al. 1998][shafroth98], [2000][shafroth00]), up to 2.0 m ([Stromberg et al. 1996][stromberg96]), up to 1.0 m, [Stromberg et al. 1991][stromberg91]), up to 4.5 m for adult lifestages ([Busch and Smith 1995][busch95]), up to 3.5 m for adult lifestages ([Lite and Stromberg 2005][lite05]), up to 4-5 m for salicae in general ([Politti et al. 2018][politti18])*
	+  The base case applies sample values of a minimum cutting length of 7 ft (2.1 m), a topographic change observation period length of 3 years and a planting depth of 80\% (0.8) of the cutting length.

-   White Alder (*Alnus Rhombifolia*)
	+  `taux` (threshold of 0.047)
	+  `tcd` - `scour` *>=* 1 ft ([Jablkowski et al. 2017][jablkowski17]) multiplied with the number of years of observation covered by the `scour` raster
	+  `d2w` with a lower threshold of 0.9 m (3 ft) and an upper threshold of 1.5 m (5 ft) ([Stillwater Sciences 2006][stillwater06]) or less than 5.0 m for grown up trees ([Claessens et al. 2010][claessens10])

-   Willows
	+  (Goodding\'s) Black willow (*Salix nigra* including *Salix Gooddingii*)
	   * `h` >= 1.0-1.5 *times the shrub height* ([Stromberg et al. 1993][stromberg93])
	   * `d2w` with a lower threshold of 0.3 m (1.0 ft) and an upper threshold of 1.5 m  (4.9 ft) \
         *Full source ranges: 0.6 m to 2.7 m ([SYRCL 2013][syrcl13]), 0.9 m to 1.5 m ([Stillwater Sciences 2006][stillwater06]), up to 2.0 m ([Shafroth et al. 1998][shafroth98]), 0.21±0.05 m to 1.44±0.22 m for establishment of woody riparian species, up to 2.0 m for existing plants([Stromberg et al. 1996][stromberg96]), up to 3.2 m for adult lifestages, [Stromberg et al. 1996][stromberg96]), up to 4-5 m for salicae in general ([Politti et al. 2018][politti18]), up to 2.6 m for native plants ([Lite and Stromberg 2005][lite05])*
	+  Red willow (*Salix laevigata*)
	   * `d2w` with a lower threshold of 0.3 m (1.0 ft) and an upper threshold of 1.5 m  (4.9 ft) \
	     *Full source ranges: 0.6 m to 2.7 m ([SYRCL 2013][syrcl13]), 0.9 m to 1.5 m ([Stillwater Sciences 2006][stillwater06]), up to 3.0 m ([USACE, YWA and 2017][usace17], [2019][usace19])*
	+  Arroyo willow (*Salix lasiolepis*)
	   * `d2w` with a lower threshold of 0.3 m (1.0 ft) and an upper threshold of 1.5 m  (4.9 ft) \
	     *Full source ranges: 0.6 m to 2.7 m ([SYRCL 2013][syrcl13])*
	+  Willows (*Salix alba*)\
	   *All *Salix alba* parameters are extracted from [Pasquale et al. (2011)][pasquale11], [(2012)][pasquale12], and [(2014)][pasquale14].*
	   * `h` *>=* 0.2 m (0.7 ft)
	   * `taux` with a threshold value of 0.1 (if the root depth is larger than 0.5 m and the stem height is larger than 1.0 m)
	   * `tcd` - `scour` >= 0.1 m (0.2 ft)      
	+  The base case applies sample values of a minimum cutting length of 7 ft (2.1 m), a topographic change observation period length of 3 years and a planting depth of 80\% (0.8) of the cutting length.

Some of the above-listed studies on plant stability refer to sand-bed rivers, where scour depths are likelier to be achieved than in gravel-bed rivers (e.g., [Wilcock 1988][wilcock88], [Bywater-Reyes et al. 2015][bywater15]). Threshold values from studies including data from sand-beds ([Stromberg et al. 1993][stromberg93], [Wilcox and Shafroth 2013][wilcox13], [Kui and Stella 2016][kui16a]) require a safety factor of 2 to 4 regarding scour, as the root-plant surfaces are higher in the finer sand beds. The increased contact surface causes higher stability (i.e., the required drag forces for uprooting in gravel/cobble bed rivers are 2-4 times lower) ([Politti et al. 2018][politti18]).

More information on depth to groundwater `d2w` requirements can be derived from [The Nature Conservancy's website][ncw19] ([Plant Rooting Depth Database](https://groundwaterresourcehub.org/public/uploads/pdfs/Plant_Rooting_Depth_Database_20180419.xlsx)).
Indigenous plant species can be found on regional / country-specific websites listed in the following table (source: [Schwindt et al. 2019][schwindt19], supplemental material).

| Country / Region       | Platform                                                     |
| ---------------------- | ------------------------------------------------------------ |
| Canada (Arctic)        | [Flora of the Canadian Arctic Archipelago][floracanada11]    |
| China                  | [Flora of China][florachina08]                               |
| Croatia                | [Flora Croatica][florafcd18]                                 |
| Cyprus                 | [Flora of Cyprus][floracyprus16]                             |
| Czechia                | [Florabase][florabase18]                                     |
| England                | [Online Atlas of the British and Irish Flora][brc18]         |
| Finland                | [Nature Gate][naturegate18]                                  |
| France                 | [Centre de ressources national sur la flore de France][fcbn18] |
| Germany                | [Flora Web][floraweb18]                                      |
| Great Britain          | [Online Atlas of the British and Irish Flora][brc18]         |
| Greece (Ionic islands) | [Flora Ionica][floraionica18]                                |
| Ireland                | [Online Atlas of the British and Irish Flora][brc18]         |
| Israel                 | [Wildflowers of Israel][israflora18]                         |
| Italy                  | [Acta plantarum (IFPI)][actaplantarum18]                     |
| Malta                  | [Malta Wild Plants][maltaflora14]                            |
| Netherlands            | [NDFF Verspreidingsatlas][ndff18]                            |
| New Zealand            | [New Zealand Department of Conservation (Lange et al.)][delange99] |
| Norway                 | [The Flora of Svalbard][svalbardflora18]                     |
| Pakistan               | [Flora of Pakistan][florapakistan18]                         |
| Portugal               | [Sociedade Portuguesa de Botânica][floraon18]                |
| Russia                 | [Plantarium (Plantarium Project)][florarussia18]             |
| Slovakia               | [Bibdigital (CSIC)][bibdigital18]                            |
| Spain                  | [Flora Iberica][floraiberica18]                              |
| Switzerland            | [Info Flora][infoflora18]                                    |
| Turkey                 | [Plants of Turkey (TAS)][floraturkey18]                      |
| United Kingdom         | [Online Atlas of the British and Irish Flora][brc18]         |
| USA, California        | [Calflora][calflora17]                                       |
| USA, general           | [Flora of Northamerica][floranorthamerica08]                 |
| Worldwide (1)          | [Free and open access to biodiversity data][gbif18]          |
| Worldwide (2)          | [The Plant List][plantlist13]                                |


## Sediment replenishment / gravel augmentation<a name="gravel"></a>

Large dams tend to retain the nearby-totality of the catchment sediment supply. The missing sediment causes channel incision and the morphological depletion of rivers in the long term. Regular artificial gravel injections can antagonize this artificial sediment scarcity (e.g., [Pasternack et al. 2010][pasternack10a]). Other authors ([Gaeuman 2008][gaeuman08] and [Ock et al. 2013][ock13]) distinguish replenishment techniques inside and outside of the main channel. According to this differentiation, two types of gravel augmentation are considered in the *LifespanDesign* module:

1.  Gravel stockpiles on the floodplain and river banks; and

2.  Gravel injections or stockpiles directly in the main channel.

Gravel deposits on floodplains should be erodible by frequent floods (i.e., stockpiles make sense where only larger floods entrain grains). In contrast, gravel injections in the main channel aim at the immediate creation of spawning habitat that should not wash out with the next minor flood event. However, gravel injections with low longevity in the main channel can also serve for an urgent equilibrium of river sediment budget. Therefore, the lifespan maps for gravel replenishment require
two different interpretations inside and outside of the main channel: High lifespans are desirable in the main channel for immediate habitat creation and low lifespans are desirable to equilibrate the sediment budget.

-   In-channel gravel injections

     **Lifespan Maps**

    -   `mobile_grains` analysis with a minimum frequency of 1.0 year and *&tau;<sub>\*,cr</sub>* threshold of 0.047 (see [grain mobility formulae](#rocks)).
    -   `mu` uses the inclusive method with `mu_good = ["chute", "fast glide", "flood runner", "bedrock", "lateral bar", "medial bar", "pool", "riffle", "riffle transition", "run", "slackwater", "slow glide", "swale", " tailings"]`

    **Design Maps**

    -   `stable_grains` for design maps (see grain mobility formulae in the [angular boulders](#rocks) section), with of 1.0 years and *&tau;<sub>\*,cr</sub>*-threshold of 0.047.
-   Floodplain / overbank gravel stockpiles

     **Lifespan Maps**

    -   `mobile_grains` analysis with a minimum frequency of 1.0 year and *&tau;<sub>\*,cr</sub>* threshold of 0.047 (see grain mobility formulae in the [angular boulders](#rocks) section).
    -   `tcd` - `scour` with a threshold value of 0.3 m (1 ft) per year.
    -   `mu` uses the inclusive method with `mu_good = ["agriplain", "backswamp", "bank", "cutbank", "flood runner", "floodplain", "high floodplain", " hillside", "island high floodplain", "island floodplain", "lateral bar", "levee", "medial bar", "mining pit", "point bar", "pond", "spur dike", "tailings ", "terrace"]`

    **Design Maps**

    -   `stable_grains` for design maps (see grain mobility formulae in the [angular boulders](#rocks) section), with of 1.0 years and *&tau;<sub>\*,cr</sub>*-threshold of 0.047.

## Side cavities<a name="sidecav"></a>

From a parametric point of view, side cavities make sense at channel banks to create preservable habitat and/or endorse protection to prevent bank collapses. In the latter case, groin cavities are an adequate protection measure that can additionally improve habitat conditions. The code analyses relevant sites based on the morphological units and important scour rates at banks. It excludes fill zones where artificial side cavities are prone to sedimentation making the measure ecologically inefficient.

-   `tcd`
	+ `fill` threshold value of 0.3 m (1 ft) multiplied with the number of observation years (i.e., if the `fill` raster includes information over multiple years) 
	+ `scour` threshold of 30.5 m (100 ft) marks an arbitrarily chosen value that leads to the exclusion of all fill-prone sites, where side cavities may be easily buried.
-   `mu` using the inclusive method with `mu_good = ["bank", "cutbank", "lateral bar", "spur dike", "tailings"]`.


## Side channels / anabranches<a name="sidechnl"></a>

Any discrete parameters exist for assessing design or lifespan maps for side channels, anabranches, anastomosed or multithread channels. The identification of splays and bank rigidity requires manual and visual proof.\
An initial decision support on the basis of design maps was contemplated by comparing the minimum energy slope *S<sub>e,min</sub>* with the terrain slope*S<sub>0</sub>*. In the 1D-theory, the minimum energy slope results from the
*H*-*h* diagram ([Glenn 2015][glenn15]), based on the assumption that the minimum energy per unit force and pixel *H<sub>min</sub>* corresponds to the Froude number *Fr* = 1 with the critical flow velocity *u<sub>c</sub>* and flow
depth *h<sub>c</sub>*. The pixel unitary discharge results from *q* = *u \* h*, where *u* and *h* are pixel values from the `u` and `h` rasters. Thus, the following set of equations can be used:

  *Fr = 1 <-> 1 = u<sub>c</sub> / (g \* h<sub>c</sub>)<sup>0.5</sup> <-> u<sub>c</sub> = (g \* h<sub>c</sub>)<sup>0.5</sup>*

  *h<sub>c</sub> = (q<sup>2</sup> / g)<sup>1/3</sup>*

  *q = u\*h*

*-> H<sub>min</sub> = h<sub>c</sub> + u<sub>c</sub><sup>2</sup> / (2g) = 1.5\*(q<sup>2</sup> / g)<sup>1/3</sup>*


Thus, the available discharges and related flow velocity `u` and depth `h` rasters could be used for the following calculation (python script sample):

```python
S0 = Slope(dem.raster, "PERCENT_RISE", 1.0))/100
for h.ras in h.rasters and u.ras in u.rasters:
          # compute energetic level
          energy_level[discharge] = dem.raster + 1.5 * Power(Square(h.ras[discharge] * u.ras[discharge]) / g, 1/3)})
          # compute energy slope Se,min
          Se[discharge] = Slope(energy_level[discharge], "PERCENT_RISE", 1.0))/100
          # result = compare Se and S0 (Se / S0)
          Se_S0[discharge] = Se[discharge] / S0)})

```

This sample function uses `arcpy.sa`'s `Slope` function with the arguments `PERCENTRISE` for obtaining percent values instead of degrees and `zFactor` = 1.0 because the x-y-grid units are the same as in z-direction. `g` denotes gravity acceleration (SI metric: 9.81 m/s<sup>2</sup> or U.S. customary: 32.2 ft/s*<sup>2</sup>*).\
However, the underlying 2D numerical model uses the critical flow depth as an iteration criterion for stability, which causes that *S<sub>e,min</sub>* approximately equals *S<sub>0</sub>*. Thus, the *S<sub>e,min</sub>* / *S<sub>0</sub>* ratio is approximately unity and not meaningful. Otherwise, the ratio *S<sub>e,min</sub>* / *S<sub>0</sub>* indicates pixels with excess energy (*S<sub>e,min</sub>* / *S <sub>0</sub> >* 1) allegedly cause erosion. Pixels with energy shortage  (*S<sub>e,min</sub>* / *S <sub>0</sub> <* 1) allegedly result in sediment deposition. Minor topographic change would be expected where the *S<sub>e,min</sub>* / *S <sub>0</sub>*-ratio is close to unity.\
Unless this problem is not solved, the package indicates the adequacy of side channel construction on lifespan maps using the following criteria:

-   `tcd` - `fill`, where the fill rate does not exceed the threshold value defined in the [thresholds workbook](#introduction-and-feature-groups).
-   `taux` the critical dimensionless bed shear stress should be smaller than in the threshold values defined in the [thresholds workbook][3].
-   `sidech` needs to be a manually created *GeoTIFF* (or GRID) raster in the `01_Conditions/CONDITION/` directory. The delineation is typically made in a shape file, which is then converted into a raster file. The delineation criteria are [Van Denderen et al. 2017][vandenderen17]:
    +   Side channel intakes are situated at the outer bank, downstream of outer bends or at the inner bank, inside mild inner bends
    +   A side channel should be longer than the main channel to avoid cutting off the main channel
    +   Structures should be placed in the side channel to control the flow re-partitioning and to avoid flow separation in the main channel.



# Define new and modify existing features<a name="modfeat"></a>
The workbook `RiverArchitect/LifespanDesign/.templates/threshold_values.xlsx` contains pre-defined features and defines feature names as well as features IDs, which can be modified if needed. The workbook can be accessed either by clicking on the [*LifespanDesign*][3] GUI's "Modify survival threshold values" button (or directly from the above directory).\
Modifications of feature IDs and names require careful consideration because the packages apply analysis routines as a function of the features *Python* classes. Changing feature names and parameters and IDs only provides the possibility of renaming features and modifying threshold values, as well as the unit system. The feature IDs are internal abbreviations, which also determine the names of output Rasters, shapefiles, and maps. Editing feature evaluations (e.g., adding
analysis routines) requires changes in the *Python* code as explained on the [*LifespanDesign*][3] pages.\
The workbook enables changing vegetation plantings species in columns `J` to `M`. The following columns are associated with distinct feature layers (cf. definitions in the [feature overview section](#featoverview)) in the workbook:

-   Terraforming features: Columns `"E"`, `"F"`, `"G"`, `"H"`, `"I"`.
-   Plant features: Columns `"J"`, `"K"`, `"L"`, `"M"`.
-   Other Bioengineering features: Columns `"N"`, `"O"`, `"P"`.
-   Connectivity features: Columns `"Q"`, `"R"`, `"S"`.

Detailed instructions for the usage of `threshold_values.xlsx` is provided in the [*LifespanDesign*](LifespanDesign#interface-and-choice-of-features) module, where also more information on threshold values is provided. If the spreadsheet is accidentally deleted or irreparable, incorrect modifications were made, there is an offline backup copy available:\
`RiverArchitect/LifespanDesign/.templates/threshold_values - Copy.xlsx`


[1]: https://github.com/RiverArchitect/RA_wiki/Installation
[2]: https://github.com/RiverArchitect/RA_wiki/Signposts
[3]: https://github.com/RiverArchitect/RA_wiki/LifespanDesign
[4]: https://github.com/RiverArchitect/RA_wiki/MaxLifespan
[5]: https://github.com/RiverArchitect/RA_wiki/ModifyTerrain
[6]: https://github.com/RiverArchitect/RA_wiki/SHArC
[7]: https://github.com/RiverArchitect/RA_wiki/ProjectMaker
[8]: https://github.com/RiverArchitect/RA_wiki/Tools
[9]: https://github.com/RiverArchitect/RA_wiki/FAQ
[10]: https://github.com/RiverArchitect/RA_wiki/Troubleshooting

[busch95]: https://doi.org/10.2307/2937064
[bywater15]: http://dx.doi.org/10.1002/2014WR016641
[carley12]: https://www.sciencedirect.com/science/article/pii/S0169555X12003819
[claessens10]: https://doi.org/10.1093/forestry/cpp038
[friedman99]: http://dx.doi.org/10.1002/(SICI)1099-1646(199909/10)15:5<463::AID-RRR559>3.0.CO;2-Z
[gaeuman08]: http://odp.trrp.net/DataPort/doc.php?id=346
[glenn15]: https://doi.org/10.1201/b18359
[horton01a]: https://doi.org/10.1046/j.1365-3040.2001.00681.x
[horton01b]: https://doi.org/10.1890/1051-0761(2001)0111046:PRTGDV2.0.CO;2
[jablkowski17]: http://www.sciencedirect.com/science/article/pii/S0169555X1730274X
[kui16a]: http://www.sciencedirect.com/science/article/pii/S0378112716300123
[lange05]: https://www.ethz.ch/content/dam/ethz/special-interest/baug/vaw/vaw-dam/documents/das-institut/mitteilungen/2000-2009/188.pdf
[lite05]: https://doi.org/10.1016/j.biocon.2005.01.020
[manningsn]: https://en.wikipedia.org/wiki/Manning_formula#Manning_coefficient_of_roughness
[maynord08]: https://ascelibrary.org/doi/10.1061/9780784408148.apb
[ncw19]: https://groundwaterresourcehub.org/gde-tools/gde-rooting-depths-database-for-gdes
[ock13]: https://www.jstage.jst.go.jp/article/hrl/7/3/7_54/_article
[pasquale11]: https://www.hydrol-earth-syst-sci.net/15/1197/2011/
[pasquale12]: http://www.sciencedirect.com/science/article/pii/S0925857411003697
[pasquale14]: http://dx.doi.org/10.1002/hyp.9993
[pasternack10a]: http://calag.ucanr.edu/archive/?article=ca.v064n02p69
[politti18]: https://www.sciencedirect.com/science/article/pii/S0012825217301186
[polzin06]: https://link.springer.com/article/10.1672/0277-5212(2006)26%5B965:EDSSSA%5D2.0.CO;2
[ruiz16b]: https://www.sciencedirect.com/science/article/pii/S0169555X15002019?via%3Dihub
[schwindt19]: https://www.sciencedirect.com/science/article/pii/S0301479718312751
[shafroth98]: https://doi.org/10.1007/BF03161674
[shafroth00]: https://scholarsarchive.byu.edu/wnan/vol60/iss1/6/
[stella03]: http://www.stillwatersci.com/resources/2003stellaetal.pdf
[stillwater06]: http://www.stillwatersci.com/resources/2006calfed_chfl_restore.pdf
[stromberg91]: https://www.researchgate.net/profile/Brian_Richter/publication/315612144_Flood_Flows_and_Dynamics_of_Sonoran_Riparian_Forests/links/58d547f345851533785bc741/Flood-Flows-and-Dynamics-of-Sonoran-Riparian-Forests.pdf
[stromberg93]: http://www.jstor.org/stable/41712765
[stromberg96]: https://doi.org/10.1002/(SICI)1099-1646(199601)12:1<1::AID-RRR347>3.0.CO;2-D
[syrcl13]: http://yubariver.org/wp-content/uploads/2013/12/Hammon-Bar-Report-2013-SYRCL.pdf
[usace00]: https://www.publications.usace.army.mil/Portals/76/Publications/EngineerManuals/EM_1110-2-1913.pdf
[usace17]: https://www.spk.usace.army.mil/Portals/12/documents/environmental/Yuba_Jan_2018/Appendix%20C%20-%20Engineering.pdf
[usace19]: https://www.spk.usace.army.mil/Portals/12/documents/environmental/Yuba_Jan_2018/Final-Report-EA_Jan2019/YubaEcoStudyFREA-wFONSI.pdf
[vandenderen17]: https://onlinelibrary.wiley.com/doi/full/10.1002/esp.4267
[weber17]: http://www.sciencedirect.com/science/article/pii/S0169555X16309862
[wilcock88]: http://dx.doi.org/10.1029/WR024i007p01127
[wilcox13]: http://dx.doi.org/10.1002/wrcr.20256
[wyrick14]: https://www.sciencedirect.com/science/article/pii/S0169555X14000099
[wyrick16]: https://onlinelibrary.wiley.com/doi/full/10.1002/esp.3854

[floracanada11]: https://nature.ca/aaflora/data/index.htm
[florachina08]: http://flora.huh.harvard.edu/china/mss/alphabetical_families.htm
[florafcd18]: http://hirc.botanic.hr/fcd/
[floracyprus16]: http://www.flora-of-cyprus.eu/endemics
[florabase18]: http://quick.florabase.cz/
[brc18]: https://www.brc.ac.uk/plantatlas/list_plants
[naturegate18]: http://www.luontoportti.com/suomi/en/kasvit
[fcbn18]: http://www.fcbn.fr/donnees-flore/centre-de-ressources-national-sur-la-flore-de-france
[floraweb18]: http://www.floraweb.de/pflanzenarten/downloads.html
[floraionica18]: https://floraionica.univie.ac.at/index.php?site=1
[israflora18]: http://www.wildflowers.co.il/english/
[actaplantarum18]: http://actaplantarum.org/flora/flora.php
[maltaflora14]: http://www.maltawildplants.com/
[ndff18]: https://www.verspreidingsatlas.nl/vaatplanten
[delange99]: https://www.doc.govt.nz/globalassets/documents/conservation/native-plants/chatham-islands-vascular-plants-checklist.pdf
[svalbardflora18]: http://svalbardflora.no/
[florapakistan18]: http://www.tropicos.org/Project/Pakistan
[floraon18]: http://flora-on.pt/
[florarussia18]: http://www.plantarium.ru/
[bibdigital18]: http://bibdigital.rjb.csic.es/ing/Volumenes.php?Libro=6351
[floraiberica18]: http://www.floraiberica.es/PHP/familias_lista.php
[infoflora18]: https://www.infoflora.ch/en/
[floraturkey18]: https://turkiyebitkileri.com/en/
[calflora17]: http://www.calflora.org
[floranorthamerica08]: http://floranorthamerica.org/
[gbif18]: https://www.gbif.org/
[plantlist13]: http://www.theplantlist.org/