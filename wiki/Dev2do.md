The to-do list for developers
===========

To edit this list, please clone the Wiki ([see descriptions](DevGit)) and follow the [Wiki writing instructions](DevWiki).

Minor future tasks:

1. The SHArea calculation in [Project Maker](ProjectMaker#pmSHArea) does not transfer the formulae from the template workbooks (`ProjectMaker/.templates/Project_vii_TEMPLATE/Geodata/SHArea_evaluation_template_si.xlsx --SHArea_evaluation_template_us.xlsx`). *SHArea* is currently only internally calculated without that the workbook formulae are functionale. Thus, *River Architect* / *Project Maker* do their job, but it is hard to look-up intermediate calculation steps. The problem comes from `openpyxl` and using [`oxl.worksheet.copier`](https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/worksheet/copier.html) will probably fix the issue in `ProjectMaker/s40_compare_sharea.py`.

Major future tasks for *River Architect* development:

1. Resolve `arcpy` dependencies (move to *QGIS* and [`gdal`](https://gdal.org))
1. Update and test [`River Builder`](RiverBuilder) implementation
1. Couple with 2D numerical models
1. Change Error and Warning message logging from `logger.info()` to `logger.error()` and `logger.warning()` statements.











