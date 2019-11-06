The to-do list for developers
===========

To edit this list, please clone the Wiki ([see descriptions](DevGit)) and follow the [Wiki writing instructions](DevWiki).

Minor future tasks (max. one day of work each):

- General
	1. Change Error and Warning message logging from `logger.info()` to `logger.error()` and `logger.warning()` statements.

- [Project Maker](ProjectMaker)
	1. The SHArea calculation in [**Project Maker**](ProjectMaker#pmSHArea) does not transfer the formulae from the template workbooks (`ProjectMaker/.templates/Project_vii_TEMPLATE/Geodata/SHArea_evaluation_template_si.xlsx --SHArea_evaluation_template_us.xlsx`). *SHArea* is currently only internally calculated without that the workbook formulae are functionale. Thus, *River Architect* / *Project Maker* do their job, but it is hard to look-up intermediate calculation steps. The problem comes from `openpyxl` and using [`oxl.worksheet.copier`](https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/worksheet/copier.html) will probably fix the issue in `ProjectMaker/s40_compare_sharea.py`.
	1. [**Project Maker**](ProjectMaker#pmplants) places plant species in alphabeticaly order when several plant species have the same maximum lifespan at a pixel. However, users should have the option to define the order in which plant species are placed. A drop-down menu or pop-up window call (button?) in the *Project Maker* GUI (`ProjectMaker/project_maker_gui.py`, somewhere between `row=9` and `row=13`) should be implemented to enable the user selection. Moreover, an adaptation will be required in `ProjectMaker/s20_plantings_delineation.py` between lines 96 to 158 (approximately); currently, the automated hierarchical order is enforced by storing pixels, which are already assigned to a plant species, in an `arcpy.Raster()` instance called `occupied_px_ras` (first instantiation is `str()`). `occupied_px_ras` is overwritten (updated) in every loop between lines 115 and 120 (approximately).
	1. New vegetation plantings are currently superpositioned with existing vegetation. This issue should be fixed in `ProjectMaker/s20_plantings_delineation.py`.
	

	

Major future tasks for *River Architect* development:

1. Add a *Load User Settings* options ([see below](#usrstgs))
1. Resolve `arcpy` dependencies (move to *QGIS* and [`gdal`](https://gdal.org))
1. Update and test [`River Builder`](RiverBuilder) implementation
1. Couple with 2D numerical models




***

# DETAILED PROBLEM DESCRIPTIONS

# User settings  <a name="usrstgs"></a>
***

A user settings load function may use a text file to enable the following user definitions (provided by [sierrajphillips](https://github.com/sierrajphillips)):

[Lifespan Design](LifespanDesign)
- Note if wildcard raster and/or habitat matching was applied
- Record Manning's n that user input
- Note if computation extent was limited to boundary.tif

[Max Lifespan](MaxLifespan)
- Record if lifespan raster extent or an extent raster was used

[SHArC](SHArC)
- Record if calculation boundary shapefile was used
- Record which HSI combination method was used: geometric or product
- Record what SHArea threshold user input (CHSI = ?)
- For Q - Area Analysis - which flow duration curve was used?

[Habitat Connectivity](Connectivity)
- Record what Q-high and Q_low input by user
- Record which interpolation method was selected

[Project Maker](ProjectMaker)
- Record the three user inputs for lifespan (years)
- Record stability drivers set by user (Manning's n and Shields parameter)

***
