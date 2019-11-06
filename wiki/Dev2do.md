The to-do list for developers
===========

To edit this list, please clone the Wiki ([see descriptions](DevGit)) and follow the [Wiki writing instructions](DevWiki).

Minor future tasks (max. one day of work each):

1. The SHArea calculation in [**Project Maker**](ProjectMaker#pmSHArea) does not transfer the formulae from the template workbooks (`ProjectMaker/.templates/Project_vii_TEMPLATE/Geodata/SHArea_evaluation_template_si.xlsx --SHArea_evaluation_template_us.xlsx`). *SHArea* is currently only internally calculated without that the workbook formulae are functionale. Thus, *River Architect* / *Project Maker* do their job, but it is hard to look-up intermediate calculation steps. The problem comes from `openpyxl` and using [`oxl.worksheet.copier`](https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/worksheet/copier.html) will probably fix the issue in `ProjectMaker/s40_compare_sharea.py`.
1. [**Project Maker**](ProjectMaker#pmplants) places plant species in alphabeticaly order when several plant species have the same maximum lifespan at a pixel. However, users should have the option to define the order in which plant species are placed. A drop-down menu or pop-up window call (button?) in the *Project Maker* GUI (`ProjectMaker/project_maker_gui.py`, somewhere between `row=9` and `row=13`) should be implemented to enable the user selection. Moreover, an adaptation will be required in `ProjectMaker/s20_plantings_delineation.py` between lines 96 to 158 (approximately); currently, the automated hierarchical order is enforced by storing pixels, which are already assigned to a plant species, in an `arcpy.Raster()` instance called `occupied_px_ras` (first instantiation is `str()`). `occupied_px_ras` is overwritten (updated) in every loop between lines 115 and 120 (approximately).
1. Change Error and Warning message logging from `logger.info()` to `logger.error()` and `logger.warning()` statements.

Major future tasks for *River Architect* development:

1. Resolve `arcpy` dependencies (move to *QGIS* and [`gdal`](https://gdal.org))
1. Update and test [`River Builder`](RiverBuilder) implementation
1. Couple with 2D numerical models












