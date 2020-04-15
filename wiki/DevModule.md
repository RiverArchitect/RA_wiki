DEVELOPMENT
===========

***

- [**Principles & Requiremments**](#raprin)
- [**Add a module to River Architect**](#addmod)
  * [Use Python templates](#template)
  * [Bind the new module to the GUI](#bind)
- [Edit the Wiki](DevWiki)
- [Use `git`](DevGit)

***

<a name="raprin"></a>
The development of *River Architect* follows the *Zen of Python*, also known as [PEP20](https://www.python.org/dev/peps/pep-0020/).

```python
	>>> import this
	The Zen of Python, by Tim Peters

	Beautiful is better than ugly.
	Explicit is better than implicit.
	Simple is better than complex.
	Complex is better than complicated.
	Flat is better than nested.
	Sparse is better than dense.
	Readability counts.
	Special cases aren't special enough to break the rules.
	Although practicality beats purity.
	Errors should never pass silently.
	Unless explicitly silenced.
	In the face of ambiguity, refuse the temptation to guess.
	There should be one-- and preferably only one --obvious way to do it.
	Although that way may not be obvious at first unless you're Dutch.
	Now is better than never.
	Although never is often better than *right* now.
	If the implementation is hard to explain, it's a bad idea.
	If the implementation is easy to explain, it may be a good idea.
	Namespaces are one honking great idea -- let's do more of those!
```

The *River Architect Wiki* for developers assumes a basic understanding of *Python3*, object-oriented programming, the *markdown* language and basics in *HTML*. In order to ensure quality, we test and debug new code before pushing it, and we run spell checkers before updating the Wiki (albeit we cannot guarantee that the writing is 100 percent correct, we do our best). 


# Add a module to *River Architect* <a name="addmod"></a>
***
Overview
-	[Use Python templates](#template)
-	[Bind the new module to the GUI](#bind)
- 	[Full folder and file structure](#struc)

***

New modules or code modifications should be pushed to the `program` repository [https://github.com/RiverArchitect/program](https://github.com/RiverArchitect/program) ([read more about using `git`](DevGit)).

*River Architect* has currently five **master** tabs (or modules) and three of them have **slave**-tabs (sub-modules): [Lifespan](LifespanDesign), [Morphology](ModifyTerrain), and [Ecohydraulics](SHArC). The development of a new module or sub-module follows the same standards and varies only in the way how the tab is finally bound in the *River Architect* master GUI. For creating a new (sub) module, do the following:

1. Have a look at *River Architect* ([see folder and file structure](Installation#structure)) and think about the capacities that the new module should have, and how it can be fitted in the existing framework (think twice before adding a new master tab).
1. Clone the template repository: `git clone https://github.com/RiverArchitect/development.git`.
1. Frome the cloned repository, copy the folder `RiverArchitect/development/moduleTEMPLATE/` to `RiverArchitect/moduleTEMPLATE/`.
1. Rename `RiverArchitect/moduleTEMPLATE/` to  `RiverArchitect/NEW_MODULE_NAME/` (obviously, replace `NEW_MODULE_NAME` with the name of the new module).
1. Edit and add the module files `RiverArchitect/NEW_MODULE_NAME/cTEMPLATE.py` and `-- TEMPLATE_gui.py` (see below descriptions of the [module template](#template)).
1. Bind the new module to the *River Architect* master GUI (see below descriptions for [binding a new module to the *River Architect* master GUI](#bind)).


## Working with the module template <a name="template"></a>
***
The *River Architect* [development repository](https://github.com/RiverArchitect/development) provides template file for creating a new module in the `moduleTEMPLATE/` directory:

- 	[`TEMPLATE_gui.py`](#templategui) is a template of the actual GUI that will be shown in the new tab.
-	[`cTEMPLATE.py`](#templateclass) is a template for writing new classes using geospatial analysis with `arcpy.sa`.

Inline comments guide through the scripts and their dependencies. Both scripts load all functions and load most of the required packages from `RiverArchitect/.site_packages/riverpy/fGlobal.py`.

### Using `TEMPLATE_gui.py` <a name="templategui"></a>

The script contains [`tkinter`](https://effbot.org/tkinterbook/) classes to build the tab GUI. The first class is a `PopUpWindow` that may be useful for inquiring complex user input (e.g., calculate a value). A good example of using complex user input within *River Architect* is the [River Builder input file creator](RiverBuilder#cinp). 

```python
class PopUpWindow(object):
    def __init__(self, master):
		... 
```

The most relevant template class here is `class MainGui(sg.RaModuleGui)`, which inherits the tab-slave class `RaModuleGui` from `RiverArchitect/slave_gui.py`. **All tabs must inherit `RiverArchitect/slave_gui.RaModuleGui`**, no matter if this will be a master or a slave tab. As such, all tabs inherit the following class variables (relevant variables only are listed here):

-	`self.condition` is a `STR` of a condition contained in `RiverArchitect/01_Conditions/`
-	`self.features` is a `cDef.FeatureDefinitions()` object resulting from `RiverArchitect/LifespanDesign/.templates/threshold_values.xlsx` ([read more about features](River-design-features))
-	`self.reaches` is a `cDef.ReachDefinitions()` object resulting from `RiverArchitect/ModifyTerrain/.templates/computation_extents.xlsx` ([read more about river reaches](RiverReaches))
-	`self.units` is a `STR` variable, which is either `us` (US customary) or `si` (SI Metric)
-	`self.ww` is an `INT` variable defining the tab window width
-	`self.wh` is an `INT` variable defining the tab window height
-	`self.wx` is an `INT` variable defining the tab window x-position
-	`self.wy` is an `INT` variable defining the tab window y-position
-	`self.xd` is an `INT` variable defining the x-pad (pixel space around `tkinter` objects)
-	`self.yd` is an `INT` variable defining the y-pad (pixel space around `tkinter` objects)

Moreover, the `RaModuleGui` class automatically creates a `"Units"` and a `"Close"` dropdown menu.<br/>

The `MainGui(sg.RaModuleGui)` class header (`__init__()` function) looks like this:

```python
class MainGui(sg.RaModuleGui):
    def __init__(self, from_master):
        sg.RaModuleGui.__init__(self, from_master)
        self.ww = 800  # window width
        self.wh = 840  # window height
        self.title = "TEMPLATE"         # <---------------------- SET MODULE NAME HERE, BUT DO NOT CHANGE CLASS NAME
        self.set_geometry(self.ww, self.wh, self.title)
		# ... tkinter examples
```

We recommend modifying the `__init__()` function of new modules starting with the `self.ww` and `self.wh` variables, which define the tab window width and height in pixels, respectively (try to avoid values larger than 1000 pixels). Most important and **required** edits are:

-	the **file name** (rename `TEMPLATE_gui.py` to something like `new_module_gui.py`), and
-	the window title defined with the `self.title` variable (see above code snippet, where `"TEMPLATE"` should be renamed to something like `"New Module"`).

Below `self.title` variable, the template provides some examples for using `tkinter` objects such as labels, listboxes with scroll bars, buttons, checkbuttons, and drop-down menus. For `tkinter` beginners, we recommend to carefully explore  different `tkinter` objects ([read more](https://effbot.org/tkinterbook/tkinter-index.htm#class-reference)).<br/>

The `MainGui.run_calculation()` function illustrates the implementation of a geospatial calculator class ([see `cTEMPLATE.py`](#templategui)):

```python
    def run_calculation(self):
        """
        Trigger a calculation with cTEMPLATE.TEMPLATE()
        """
        geo_calc_object = cTEMPLATE.TEMPLATE()

        # inquire input raster name
        dir2raster = askopenfilename(initialdir=".", title="Select input GeoTIFF raster",
                                     filetypes=[('GeoTIFF', '*.tif;*.tiff')])
        result = geo_calc_object.use_spatial_analyst_function(dir2raster)
        geo_calc_object.clear_cache()  # remove temporary calculation files

        if not(result == -1):
            showinfo("SUCCESS", "The result raster %s has been created." % result)
        else:
            showinfo("ERROR", "Something went wrong. Check the logfile and River Architect Troubleshooting Wiki.")
```

`geo_calc_object` is a `cTEMPLATE.TEMPLATE()` object that is locally created in the `run_calculation()` function. It has a `use_spatial_analyst_function` that requires an input raster path to apply a (random) *ArcGIS Spatial Analyst* map algebra expression to that input raster. For this purpose, the `MainGui.run_calculation()` function asks the user to define an input raster using `tkinters`'s `askopenfilename()` function (imported from `tkinter.filedialog`). The line `result = geo_calc_object.use_spatial_analyst_function(dir2raster)` passes the selected raster file path (`dir2raster`) to the geospatial calculation function, which returns a string of the output raster if the calculation was successful (otherwise, it returns -1). After the calculation, an info window pops up (`showinfo(WINDOW_NAME, MESSAGE)` imported from `tkinter.messagebox`) and informs about the calculation success.

The `MainGui.set_value()` function illustrates the usage of the above mentioned `PopUpWindow` class (in the same file `TEMPLATE_gui.py`) to modify a value. The usage in the template is somewhat meaningless; a more reasonable value modification can be found in the *Lifespan Design* module where the user can modify the *Manning's n* roughness value ([see applicaton](LifespanDesign#lfinpopt)).

```python
    def set_value(self):
        sub_frame = PopUpWindow(self.master)
        self.master.wait_window(sub_frame.top)
        value = float(sub_frame.value)
        showinfo("INFO", "You entered %s (whatever...)." % str(value))
```

The `MainGui.set_value()` function creates a sub-frame by instantiating a `PopUpWindow` object with `MainGui` as master. While the `PopUpWindow` is open, the main window waits for user input in the `PopUpWindow` (`self.master.wait_window(sub_frame.top)`). After closing the window, the function retrieves the user-defined value with `value = float(sub_frame.value)`. Finally, an info window pops up (`showinfo(WINDOW_NAME, MESSAGE)` imported from `tkinter.messagebox`) and recalls the user-defined value (well, this is trivial here ...).


### Using `cTEMPLATE.py` <a name="templategui"></a>

The `cTEMPLATE.TEMPLATE()` class provides a useful frame for programming geospatial calculations, but none of the template routines is obligatory (in contrast to the [above GUI template](#templategui). For using the template class, make the following adaptations to the code block:

```python
class TEMPLATE:
    def __init__(self, *args, **kwargs):
        """
        ...
        """

        # Create a temporary cache folder for, e.g., geospatial analyses; use self.clear_cache() function to delete
        self.cache = os.path.dirname(__file__) + "\\.cache%s\\" % str(random.randint(10000, 99999))
        chk_dir(self.cache)  # from fGlobal

        # Enable logger
        self.logger = logging.getLogger("logfile")
```

-	rename the class from `TEMPLATE` to something meaningful;
-	modify the input arguments of the `__init__(self, *args, **kwargs)` function.

The `cTEMPLATE.TEMPLATE()` class initiates a `self.logger` variable to write calculation progress to a logfile that is created as `RiverArchitect/logfile.log` when *River Architect* is called from the main GUI. This logefile should be used at every critical step where erroneous user input may yield wrong or no result. For example, if a user selects a raster file without valid data, the code should detect and tell the user about the problem using precise `try`-`except` statements. The `cTEMPLATE.TEMPLATE.use_spatial_analyst_function(input_raster_path)` function provides an example for the implementation of such behavior:
<a name="tryexcept"></a>

```python
	try:
		self.logger.info("  >> loading raster %s ..." % str(input_raster_path))
		in_ras = arcpy.Raster(input_raster_path)
	except:
		# go here if the input_raster_path is invalid and write a USEFUL error message
		self.logger.info("ERROR: Game over ... add this error message to RA_wiki/Troubleshooting")
		return -1	
```

In this exemplary code snippet, the command `self.logger.info("  >> loading raster %s ..." % str(input_raster_path))` writes the raster input path to the console window and to the logfile using `self.logger.info()`. If the next step fails (i.e., `arcpy.Raster()` cannot load the provided `input_raster_path` as raster), the user should be informed. This information is passed recorded in the `except:` statement with `self.logger.info("ERROR: Game over ... add this error message to RA_wiki/Troubleshooting")`, which ends with returning `-1` (exits the function because the raster is necessary for all other steps). It would be better to use more precise exception rules such as `except ValueError:`, but we kept the broad "shotgun" approach with `except:` only because we encountered unexpected exception types using `arcpy`. A better and future way of error message logging in *River Architect* will be to use `self.logger.error()` in the exception statement rather than `self.logger.info()`, which should exclusively be used to log calculation progress. **Important is to add error messages, as well as warning messages, to the [Troubleshooting Wiki](Troubleshooting)** ([read more about extending the Wiki](DevWiki)). Use warning messages when a function can still work even though a variable assignment error occurred such as when an optional, additional raster is missing.

Finally, every class should close with a call function, where only the `TEMPLATE` string should be adapted to the new class name:

```python
    def __call__(self, *args, **kwargs):
        # Object call should return class name, file location and class attributes (dir)
        print("Class Info: <type> = TEMPLATE (%s)" % os.path.dirname(__file__))
        print(dir(self))
```

More features of the template are provided with the inline comments in `cTEMPLATE.py`.

## Binding a new tab (module) to the *River Architect* master GUI <a name="bind"></a>
***

New modules need to be bound to the master GUI, either as master-tab or as slave-tab. Both need to added to the `RiverArchitect/master_gui.py` script in the `RA_gui` class's `__init__()` function.

### Binding a New Module with or without submodules
Assuming a new master tab with the name ***NEW MODULE, optionally with *New Sub Modules***, should to be added, which is located in `RiverArchitect/NewModule/new_module_gui.py` (or submodules in `RiverArchitect/NewSubModule1/new_sub1_gui.py` and  `RiverArchitect/NewSubModule2/new_sub2_gui.py`, respectively), the following variables need to be complemented (**attention: the list order is important** - new modules should only be appended at the end of lists):

-	`self.tab_names = ['Get Started', 'Lifespan', 'Morphology', 'Ecohydraulics', 'Project Maker', 'NEW MODULE']`
-	For adding slave (sub) tabs with names `NEW SUB1` and `NEW SUB2`: 
		+ `self.sub_tab_parents = ['Lifespan', 'Morphology', 'Ecohydraulics', 'NEW MODULE']`
		+ `self.sub_tab_names = [['Lifespan Design', 'Max Lifespan'], ['Modify Terrain', 'Volume Assessment'], ['Habitat Area (SHArC)', 'Stranding Risk'], ['NEW SUB1', 'NEW SUB2', ... and so on]]`
-	`self.tab_dir_names`
		+ No sub tabs: `self.tab_dir_names= ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\'], ['\\SHArC\\', '\\StrandingRisk\\'], '\\ProjectMaker\\', '\\NewModule\\']` 
		+ With sub tabs located in `RiverArchitect/NewSubModule1/` and  `RiverArchitect/NewSubModule2/`, respectively: `self.tab_dir_names = ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\'], ['\\SHArC\\', '\\StrandingRisk\\'], '\\ProjectMaker\\', ['\\NewSubModule1\\', '\\NewSubModule2\\']]`
-	`self.tab_list`
		+ No sub tabs: `self.tab_list= [GetStarted.welcome_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ProjectMaker.project_maker_gui.MainGui(self.tab_container), NewModule.new_module_gui.MainGui(self.tab_container)]`
		+ With sub tabs: `self.tab_list= [GetStarted.welcome_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ProjectMaker.project_maker_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container)]`
- 	With sub tabs: `self.sub_tab_list = [[LifespanDesign.lifespan_design_gui.FaGui(self.tabs['Lifespan']), MaxLifespan.action_gui.ActionGui(self.tabs['Lifespan'])],  [ModifyTerrain.modify_terrain_gui.MainGui(self.tabs['Morphology']),  VolumeAssessment.volume_gui.MainGui(self.tabs['Morphology'])], [SHArC.sharc_gui.MainGui(self.tabs['Ecohydraulics']), StrandingRisk.connect_gui.MainGui(self.tabs['Ecohydraulics'])], [NewSubModule1.new_sub1_gui.MainGui(self.tabs['NEW SUB1']), NewSubModule2.new_sub2_gui.MainGui(self.tabs['NEW SUB2'])]]`


### Binding a New Sub-Module to an existing module

The following list indicates variables to be modified when a new sub-module (slave or sub-tab) is added to an existing module. The example assumes the creation of a sub-tab called `RiverCreator` to the `Morphology` module.

-	Verify that the `Morphology` tab is mentioned in  `self.sub_tab_parents = ['Lifespan', 'Morphology', 'Ecohydraulics', 'NEW MODULE']`
-	Add `RIVER CREATOR` to the second nested list in `self.sub_tab_names = [['Lifespan Design', 'Max Lifespan'], ['Modify Terrain', 'Volume Assessment', 'RIVER CREATOR'], ['Habitat Area (SHArC)', 'Stranding Risk']]`; the second nested list must be chosen because `Morphology` is the second item in `self.sub_tab_parents`
-	Assuming that the new module is located in `RiverArchitect/RiverCreator`, modify `self.tab_dir_names = ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\', '\\RiverCreator\\'], ['\\SHArC\\', '\\StrandingRisk\\'], '\\ProjectMaker\\', ['\\NewSubModule1\\', '\\NewSubModule2\\']]`
- 	Withe the asumption that the new *River Creator* GUI is located in `RiverArchitect/RiverCreator/rc_gui.py`, modify `self.sub_tab_list = [[LifespanDesign.lifespan_design_gui.FaGui(self.tabs['Lifespan']), MaxLifespan.action_gui.ActionGui(self.tabs['Lifespan'])], [ModifyTerrain.modify_terrain_gui.MainGui(self.tabs['Morphology']), VolumeAssessment.volume_gui.MainGui(self.tabs['Morphology']),  RiverCreator.rc_gui.MainGui(self.tabs['River Creator'])], [SHArC.sharc_gui.MainGui(self.tabs['Ecohydraulics']), StrandingRisk.connect_gui.MainGui(self.tabs['Ecohydraulics'])]]`


**Please do not forget to [update the wiki](DevWiki) and test new code!**


# Full list of folders and files <a name="struc"></a>

***

This is the full list of files and folders in the `RiverArchitect/program` repository. Please update after adding or deleting files. The file list can be generated (in Windows) with *PowerShell*: Enter `Get-ChildItem -Path D:\path-to-local-copy-of-RiverArchitect\program -Recurse` and copy-paste the comand line output.

```

RiverArchitect\program
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:38 AM                .github
d-----        7/26/2019  11:39 AM                .site_packages
d-----        7/26/2019  11:39 AM                00_Flows
d-----        7/26/2019  11:39 AM                01_Conditions
d-----        7/26/2019  11:39 AM                02_Maps
d-----       11/22/2019  10:17 AM                StrandingRisk
d-----        7/26/2019  11:39 AM                docs
d-----       11/19/2019  12:40 PM                GetStarted
d-----        7/26/2019  11:39 AM                LifespanDesign
d-----        7/26/2019  11:39 AM                MaxLifespan
d-----        10/2/2019   9:57 AM                ModifyTerrain
d-----        11/5/2019   1:57 PM                ProjectMaker
d-----        10/2/2019   9:57 AM                SHArC
d-----       11/25/2019   1:04 PM                Tools
d-----        7/26/2019  11:39 AM                VolumeAssessment
-a----       11/13/2019   8:47 AM             22 .gitignore
-a----         9/4/2019   5:04 PM           1540 LICENSE
-a----       11/22/2019  10:17 AM           6458 master_gui.py
-a----        7/26/2019  11:39 AM           5856 README.md
-a----        11/4/2019  12:33 PM          11594 slave_gui.py
-a----        7/26/2019  11:39 AM            619 Start_River_Architect.bat


RiverArchitect\program\.github
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:38 AM                ISSUE_TEMPLATE


RiverArchitect\program\.github\ISSUE_TEMPLATE
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:38 AM            834 bug_report.md
-a----        7/26/2019  11:38 AM            126 custom.md
-a----        7/26/2019  11:38 AM            595 feature_request.md


RiverArchitect\program\.site_packages
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                openpyxl
d-----       11/19/2019  12:40 PM                riverpy
d-----        7/26/2019  11:39 AM                templates
-a----        7/26/2019  11:38 AM             23 info.md


RiverArchitect\program\.site_packages\openpyxl
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:38 AM                doc
d-----        7/26/2019  11:39 AM                openpyxl
d-----        7/26/2019  11:39 AM                openpyxl 2.5.2 documentation_files
-a----        7/26/2019  11:38 AM            227 .coveragerc
-a----        7/26/2019  11:38 AM            122 .flow
-a----        7/26/2019  11:38 AM            160 .hgeol
-a----        7/26/2019  11:38 AM            274 .hgignore
-a----        7/26/2019  11:38 AM           4348 .hgtags
-a----        7/26/2019  11:38 AM            173 .hg_archival.txt
-a----        7/26/2019  11:38 AM           1405 AUTHORS.rst
-a----        7/26/2019  11:38 AM            175 bitbucket-pipelines.yml
-a----        7/26/2019  11:38 AM           1131 LICENCE.rst
-a----        7/26/2019  11:38 AM            206 MANIFEST.in
-a----        7/26/2019  11:38 AM          69488 openpyxl 2.5.2 documentation.html
-a----        7/26/2019  11:39 AM             67 openpyxl.cfg
-a----        7/26/2019  11:39 AM           2090 openpyxl.py
-a----        7/26/2019  11:39 AM            499 pytest.ini
-a----        7/26/2019  11:38 AM           1046 README.rst
-a----        7/26/2019  11:39 AM             57 requirements.txt
-a----        7/26/2019  11:39 AM            486 shippable.yml
-a----        7/26/2019  11:39 AM           2077 tox.ini


RiverArchitect\program\.site_packages\openpyxl\doc
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:38 AM                charts
d-----        7/26/2019  11:38 AM                _static
-a----        7/26/2019  11:38 AM          45206 changes.rst
-a----        7/26/2019  11:38 AM           2450 comments.rst
-a----        7/26/2019  11:38 AM           9791 conf.py
-a----        7/26/2019  11:38 AM            926 defined_names.rst
-a----        7/26/2019  11:38 AM           5765 development.rst
-a----        7/26/2019  11:38 AM            335 example.py
-a----        7/26/2019  11:38 AM          25958 filters.png
-a----        7/26/2019  11:38 AM            579 filters.py
-a----        7/26/2019  11:38 AM            607 filters.rst
-a----        7/26/2019  11:38 AM           7327 formatting.rst
-a----        7/26/2019  11:38 AM           1757 format_merged_cells.py
-a----        7/26/2019  11:38 AM           3812 formula.rst
-a----        7/26/2019  11:38 AM           6170 index.rst
-a----        7/26/2019  11:38 AM           5505 logo.png
-a----        7/26/2019  11:38 AM           4269 make.bat
-a----        7/26/2019  11:38 AM           4602 Makefile
-a----        7/26/2019  11:38 AM           3810 optimized.rst
-a----        7/26/2019  11:38 AM           2425 pandas.rst
-a----        7/26/2019  11:38 AM           1691 print_settings.rst
-a----        7/26/2019  11:38 AM           2642 protection.rst
-a----        7/26/2019  11:38 AM           7561 styles.rst
-a----        7/26/2019  11:38 AM          20324 table.png
-a----        7/26/2019  11:38 AM            773 table.py
-a----        7/26/2019  11:38 AM           8508 tutorial.rst
-a----        7/26/2019  11:38 AM           4085 usage.rst
-a----        7/26/2019  11:38 AM           2661 validation.rst
-a----        7/26/2019  11:38 AM           2880 windows-development.rst
-a----        7/26/2019  11:38 AM           1792 worksheet_properties.rst
-a----        7/26/2019  11:38 AM            700 worksheet_tables.rst


RiverArchitect\program\.site_packages\openpyxl\doc\charts
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:38 AM          37749 area.png
-a----        7/26/2019  11:38 AM            691 area.py
-a----        7/26/2019  11:38 AM            642 area.rst
-a----        7/26/2019  11:38 AM          39610 area3D.png
-a----        7/26/2019  11:38 AM            718 area3d.py
-a----        7/26/2019  11:38 AM         104913 bar.png
-a----        7/26/2019  11:38 AM           1279 bar.py
-a----        7/26/2019  11:38 AM            851 bar.rst
-a----        7/26/2019  11:38 AM          28272 bar3D.png
-a----        7/26/2019  11:38 AM            573 bar3d.py
-a----        7/26/2019  11:38 AM          59465 bubble.png
-a----        7/26/2019  11:38 AM           1181 bubble.py
-a----        7/26/2019  11:38 AM            343 bubble.rst
-a----        7/26/2019  11:38 AM          41321 chartsheet.png
-a----        7/26/2019  11:38 AM            477 chartsheet.py
-a----        7/26/2019  11:38 AM            240 chartsheet.rst
-a----        7/26/2019  11:38 AM          79037 chart_layout.png
-a----        7/26/2019  11:38 AM           1600 chart_layout.py
-a----        7/26/2019  11:38 AM           1574 chart_layout.rst
-a----        7/26/2019  11:38 AM          11790 chart_layout_default.png
-a----        7/26/2019  11:38 AM          94413 doughnut.png
-a----        7/26/2019  11:38 AM           1308 doughnut.py
-a----        7/26/2019  11:38 AM            281 doughnut.rst
-a----        7/26/2019  11:38 AM          27943 gauge.png
-a----        7/26/2019  11:38 AM           1436 gauge.py
-a----        7/26/2019  11:38 AM            616 gauge.rst
-a----        7/26/2019  11:38 AM           1600 introduction.rst
-a----        7/26/2019  11:38 AM           1888 limits_and_scaling.rst
-a----        7/26/2019  11:38 AM          57320 limits_and_scaling_log.png
-a----        7/26/2019  11:38 AM           1562 limits_and_scaling_log.py
-a----        7/26/2019  11:38 AM          22311 limits_and_scaling_minmax.png
-a----        7/26/2019  11:38 AM            855 limits_and_scaling_minmax.py
-a----        7/26/2019  11:38 AM          75148 limits_and_scaling_orientation.png
-a----        7/26/2019  11:38 AM           1492 limits_and_scaling_orientation.py
-a----        7/26/2019  11:38 AM         125459 line.png
-a----        7/26/2019  11:38 AM           1961 line.py
-a----        7/26/2019  11:38 AM            680 line.rst
-a----        7/26/2019  11:38 AM          43079 line3D.png
-a----        7/26/2019  11:38 AM            790 line3D.py
-a----        7/26/2019  11:38 AM          29891 pattern.png
-a----        7/26/2019  11:38 AM            957 pattern.py
-a----        7/26/2019  11:38 AM            295 pattern.rst
-a----        7/26/2019  11:38 AM          38189 pie.png
-a----        7/26/2019  11:38 AM           1461 pie.py
-a----        7/26/2019  11:38 AM           1064 pie.rst
-a----        7/26/2019  11:38 AM          40590 pie3D.png
-a----        7/26/2019  11:38 AM            561 pie3D.py
-a----        7/26/2019  11:38 AM          60325 projected-pie.png
-a----        7/26/2019  11:38 AM          67116 radar.png
-a----        7/26/2019  11:38 AM           1000 radar.py
-a----        7/26/2019  11:38 AM            584 radar.rst
-a----        7/26/2019  11:38 AM          38199 scatter.png
-a----        7/26/2019  11:38 AM            742 scatter.py
-a----        7/26/2019  11:38 AM            651 scatter.rst
-a----        7/26/2019  11:38 AM          21534 secondary.png
-a----        7/26/2019  11:38 AM            893 secondary.py
-a----        7/26/2019  11:38 AM            363 secondary.rst
-a----        7/26/2019  11:38 AM          92652 stock.png
-a----        7/26/2019  11:38 AM           2494 stock.py
-a----        7/26/2019  11:38 AM           1605 stock.rst
-a----        7/26/2019  11:38 AM         143344 surface.png
-a----        7/26/2019  11:38 AM           1241 surface.py
-a----        7/26/2019  11:38 AM            531 surface.rst


RiverArchitect\program\.site_packages\openpyxl\doc\_static
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:38 AM              0 .placeholder


RiverArchitect\program\.site_packages\openpyxl\openpyxl
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                cell
d-----        7/26/2019  11:39 AM                chart
d-----        7/26/2019  11:39 AM                chartsheet
d-----        7/26/2019  11:39 AM                comments
d-----        7/26/2019  11:39 AM                compat
d-----        7/26/2019  11:39 AM                descriptors
d-----        7/26/2019  11:39 AM                develop
d-----        7/26/2019  11:39 AM                drawing
d-----        7/26/2019  11:39 AM                formatting
d-----        7/26/2019  11:39 AM                formula
d-----        7/26/2019  11:39 AM                packaging
d-----        7/26/2019  11:39 AM                pivot
d-----        7/26/2019  11:39 AM                reader
d-----        7/26/2019  11:39 AM                styles
d-----        7/26/2019  11:39 AM                tests
d-----        7/26/2019  11:39 AM                utils
d-----        7/26/2019  11:39 AM                workbook
d-----        7/26/2019  11:39 AM                worksheet
d-----        7/26/2019  11:39 AM                writer
d-----        7/26/2019  11:39 AM                xml
-a----        7/26/2019  11:39 AM            270 .constants.json
-a----        7/26/2019  11:39 AM           1720 conftest.py
-a----        7/26/2019  11:39 AM            880 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\cell
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM          11401 cell.py
-a----        7/26/2019  11:39 AM          12732 cell.pyc
-a----        7/26/2019  11:39 AM            992 interface.py
-a----        7/26/2019  11:39 AM           3984 read_only.py
-a----        7/26/2019  11:39 AM           6970 read_only.pyc
-a----        7/26/2019  11:39 AM           4454 text.py
-a----        7/26/2019  11:39 AM           5609 text.pyc
-a----        7/26/2019  11:39 AM            149 __init__.py
-a----        7/26/2019  11:39 AM            369 __init__.pyc


RiverArchitect\program\.site_packages\openpyxl\openpyxl\cell\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          10938 test_cell.py
-a----        7/26/2019  11:39 AM           4704 test_read_only.py
-a----        7/26/2019  11:39 AM           4512 test_text.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\chart
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           2964 area_chart.py
-a----        7/26/2019  11:39 AM           3765 area_chart.pyc
-a----        7/26/2019  11:39 AM          12691 axis.py
-a----        7/26/2019  11:39 AM          11963 axis.pyc
-a----        7/26/2019  11:39 AM           4214 bar_chart.py
-a----        7/26/2019  11:39 AM           4738 bar_chart.pyc
-a----        7/26/2019  11:39 AM           2060 bubble_chart.py
-a----        7/26/2019  11:39 AM           2496 bubble_chart.pyc
-a----        7/26/2019  11:39 AM           7902 chartspace.py
-a----        7/26/2019  11:39 AM           8688 chartspace.pyc
-a----        7/26/2019  11:39 AM           4401 data_source.py
-a----        7/26/2019  11:39 AM           6885 data_source.pyc
-a----        7/26/2019  11:39 AM            848 descriptors.py
-a----        7/26/2019  11:39 AM           1725 descriptors.pyc
-a----        7/26/2019  11:39 AM           1796 error_bar.py
-a----        7/26/2019  11:39 AM           2180 error_bar.pyc
-a----        7/26/2019  11:39 AM           4186 label.py
-a----        7/26/2019  11:39 AM           4176 label.pyc
-a----        7/26/2019  11:39 AM           2000 layout.py
-a----        7/26/2019  11:39 AM           2492 layout.pyc
-a----        7/26/2019  11:39 AM           1987 legend.py
-a----        7/26/2019  11:39 AM           2625 legend.pyc
-a----        7/26/2019  11:39 AM           4102 line_chart.py
-a----        7/26/2019  11:39 AM           4420 line_chart.pyc
-a----        7/26/2019  11:39 AM           2639 marker.py
-a----        7/26/2019  11:39 AM           3050 marker.pyc
-a----        7/26/2019  11:39 AM           1227 packaging.rst
-a----        7/26/2019  11:39 AM           1195 picture.py
-a----        7/26/2019  11:39 AM           1510 picture.pyc
-a----        7/26/2019  11:39 AM           4869 pie_chart.py
-a----        7/26/2019  11:39 AM           6065 pie_chart.pyc
-a----        7/26/2019  11:39 AM           5925 plotarea.py
-a----        7/26/2019  11:39 AM           6519 plotarea.pyc
-a----        7/26/2019  11:39 AM           1493 print_settings.py
-a----        7/26/2019  11:39 AM           2323 print_settings.pyc
-a----        7/26/2019  11:39 AM           1501 radar_chart.py
-a----        7/26/2019  11:39 AM           2067 radar_chart.pyc
-a----        7/26/2019  11:39 AM           1634 reader.py
-a----        7/26/2019  11:39 AM           2380 reader.pyc
-a----        7/26/2019  11:39 AM           3329 reference.py
-a----        7/26/2019  11:39 AM           5158 reference.pyc
-a----        7/26/2019  11:39 AM           1598 scatter_chart.py
-a----        7/26/2019  11:39 AM           2134 scatter_chart.pyc
-a----        7/26/2019  11:39 AM           5895 series.py
-a----        7/26/2019  11:39 AM           6169 series.pyc
-a----        7/26/2019  11:39 AM           1407 series_factory.py
-a----        7/26/2019  11:39 AM           1603 series_factory.pyc
-a----        7/26/2019  11:39 AM           2852 shapes.py
-a----        7/26/2019  11:39 AM           3000 shapes.pyc
-a----        7/26/2019  11:39 AM           1659 stock_chart.py
-a----        7/26/2019  11:39 AM           2080 stock_chart.pyc
-a----        7/26/2019  11:39 AM           2994 surface_chart.py
-a----        7/26/2019  11:39 AM           4566 surface_chart.pyc
-a----        7/26/2019  11:39 AM           1521 text.py
-a----        7/26/2019  11:39 AM           2230 text.pyc
-a----        7/26/2019  11:39 AM           1909 title.py
-a----        7/26/2019  11:39 AM           2887 title.pyc
-a----        7/26/2019  11:39 AM           3017 trendline.py
-a----        7/26/2019  11:39 AM           3313 trendline.pyc
-a----        7/26/2019  11:39 AM            936 updown_bars.py
-a----        7/26/2019  11:39 AM           1457 updown_bars.pyc
-a----        7/26/2019  11:39 AM           3075 _3d.py
-a----        7/26/2019  11:39 AM           3544 _3d.pyc
-a----        7/26/2019  11:39 AM           5111 _chart.py
-a----        7/26/2019  11:39 AM           7200 _chart.pyc
-a----        7/26/2019  11:39 AM            603 __init__.py
-a----        7/26/2019  11:39 AM           1095 __init__.pyc


RiverArchitect\program\.site_packages\openpyxl\openpyxl\chart\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            290 conftest.py
-a----        7/26/2019  11:39 AM           4661 test_area_chart.py
-a----        7/26/2019  11:39 AM           9278 test_axis.py
-a----        7/26/2019  11:39 AM           5335 test_bar_chart.py
-a----        7/26/2019  11:39 AM            956 test_bubble_chart.py
-a----        7/26/2019  11:39 AM           3591 test_chart.py
-a----        7/26/2019  11:39 AM           6792 test_chartspace.py
-a----        7/26/2019  11:39 AM           3503 test_data_source.py
-a----        7/26/2019  11:39 AM           1075 test_error_bar.py
-a----        7/26/2019  11:39 AM           1888 test_label.py
-a----        7/26/2019  11:39 AM           2083 test_layout.py
-a----        7/26/2019  11:39 AM           1496 test_legend.py
-a----        7/26/2019  11:39 AM           2016 test_line_chart.py
-a----        7/26/2019  11:39 AM           2066 test_marker.py
-a----        7/26/2019  11:39 AM            812 test_picture.py
-a----        7/26/2019  11:39 AM           4489 test_pie_chart.py
-a----        7/26/2019  11:39 AM           4233 test_plotarea.py
-a----        7/26/2019  11:39 AM           1506 test_print.py
-a----        7/26/2019  11:39 AM           1135 test_radar_chart.py
-a----        7/26/2019  11:39 AM           1088 test_reader.py
-a----        7/26/2019  11:39 AM           2994 test_reference.py
-a----        7/26/2019  11:39 AM            961 test_scatter_chart.py
-a----        7/26/2019  11:39 AM           8891 test_series.py
-a----        7/26/2019  11:39 AM           3765 test_series_factory.py
-a----        7/26/2019  11:39 AM           1389 test_shapes.py
-a----        7/26/2019  11:39 AM           2334 test_stock_chart.py
-a----        7/26/2019  11:39 AM           3887 test_surface_chart.py
-a----        7/26/2019  11:39 AM            918 test_text.py
-a----        7/26/2019  11:39 AM           1759 test_title.py
-a----        7/26/2019  11:39 AM           1552 test_trendline.py
-a----        7/26/2019  11:39 AM            878 test_updown_bars.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\chart\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          35393 chart1.xml
-a----        7/26/2019  11:39 AM           3958 plotarea.xml
-a----        7/26/2019  11:39 AM          37202 sample.xlsx
-a----        7/26/2019  11:39 AM           4502 scatterchart_plot_area.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\chartsheet
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           4081 chartsheet.py
-a----        7/26/2019  11:39 AM           1730 custom.py
-a----        7/26/2019  11:39 AM            718 properties.py
-a----        7/26/2019  11:39 AM           1304 protection.py
-a----        7/26/2019  11:39 AM           1626 publish.py
-a----        7/26/2019  11:39 AM           2770 relation.py
-a----        7/26/2019  11:39 AM           1380 views.py
-a----        7/26/2019  11:39 AM            110 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\chartsheet\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           3159 test_chartsheet.py
-a----        7/26/2019  11:39 AM           4671 test_custom.py
-a----        7/26/2019  11:39 AM           1238 test_properties.py
-a----        7/26/2019  11:39 AM           1950 test_protection.py
-a----        7/26/2019  11:39 AM           3523 test_publish.py
-a----        7/26/2019  11:39 AM           1916 test_relation.py
-a----        7/26/2019  11:39 AM           1775 test_views.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\comments
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM            466 author.py
-a----        7/26/2019  11:39 AM           1513 comments.py
-a----        7/26/2019  11:39 AM           6335 comment_sheet.py
-a----        7/26/2019  11:39 AM           3950 shape_writer.py
-a----        7/26/2019  11:39 AM            106 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\comments\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            341 conftest.py
-a----        7/26/2019  11:39 AM           1060 test_author.py
-a----        7/26/2019  11:39 AM           1226 test_comment.py
-a----        7/26/2019  11:39 AM           1267 test_comment_reader.py
-a----        7/26/2019  11:39 AM           2877 test_comment_sheet.py
-a----        7/26/2019  11:39 AM           3727 test_shape_writer.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\comments\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          13202 comments.xlsx
-a----        7/26/2019  11:39 AM            776 comments1.xml
-a----        7/26/2019  11:39 AM           1894 comments2.xml
-a----        7/26/2019  11:39 AM           2319 commentsDrawing1.vml
-a----        7/26/2019  11:39 AM            587 comments_out.xml
-a----        7/26/2019  11:39 AM           3920 control+comments.vml
-a----        7/26/2019  11:39 AM            342 google_docs_comments.xml
-a----        7/26/2019  11:39 AM            743 size+comments.vml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\compat
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM            194 abc.py
-a----        7/26/2019  11:39 AM            579 accumulate.py
-a----        7/26/2019  11:39 AM            487 numbers.py
-a----        7/26/2019  11:39 AM           1122 singleton.py
-a----        7/26/2019  11:39 AM           1139 strings.py
-a----        7/26/2019  11:39 AM           2040 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\compat\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           2277 test_compat.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\descriptors
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           7212 base.py
-a----        7/26/2019  11:39 AM           2543 excel.py
-a----        7/26/2019  11:39 AM            340 namespace.py
-a----        7/26/2019  11:39 AM           2651 nested.py
-a----        7/26/2019  11:39 AM           3363 sequence.py
-a----        7/26/2019  11:39 AM           6907 serialisable.py
-a----        7/26/2019  11:39 AM            824 slots.py
-a----        7/26/2019  11:39 AM           1855 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\descriptors\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           7602 test_base.py
-a----        7/26/2019  11:39 AM           5168 test_excel.py
-a----        7/26/2019  11:39 AM            504 test_namespace.py
-a----        7/26/2019  11:39 AM           7593 test_nested.py
-a----        7/26/2019  11:39 AM           8306 test_sequence.py
-a----        7/26/2019  11:39 AM           4326 test_serialisable.py
-a----        7/26/2019  11:39 AM             74 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\develop
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           9857 classify.py
-a----        7/26/2019  11:39 AM           1289 stub.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\develop\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM           4258 test_classify.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\develop\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           1528 defined_name.xsd


RiverArchitect\program\.site_packages\openpyxl\openpyxl\drawing
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM          15377 colors.py
-a----        7/26/2019  11:39 AM           2824 drawing.py
-a----        7/26/2019  11:39 AM           9507 effect.py
-a----        7/26/2019  11:39 AM          11940 fill.py
-a----        7/26/2019  11:39 AM          16846 graphic.py
-a----        7/26/2019  11:39 AM           1970 image.py
-a----        7/26/2019  11:39 AM           4365 line.py
-a----        7/26/2019  11:39 AM          11326 shape.py
-a----        7/26/2019  11:39 AM          16534 shapes.py
-a----        7/26/2019  11:39 AM           9888 spreadsheet_drawing.py
-a----        7/26/2019  11:39 AM          22480 text.py
-a----        7/26/2019  11:39 AM            105 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\drawing\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            243 conftest.py
-a----        7/26/2019  11:39 AM           4593 test_color.py
-a----        7/26/2019  11:39 AM            316 test_descriptors.py
-a----        7/26/2019  11:39 AM           2760 test_drawing.py
-a----        7/26/2019  11:39 AM           1113 test_effect.py
-a----        7/26/2019  11:39 AM           6149 test_fill.py
-a----        7/26/2019  11:39 AM          15806 test_graphic.py
-a----        7/26/2019  11:39 AM           1035 test_image.py
-a----        7/26/2019  11:39 AM           3513 test_line.py
-a----        7/26/2019  11:39 AM           7351 test_shape.py
-a----        7/26/2019  11:39 AM           3256 test_shapes.py
-a----        7/26/2019  11:39 AM          10487 test_spreadsheet_drawing.py
-a----        7/26/2019  11:39 AM           3109 test_text.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\drawing\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            493 plain.png
-a----        7/26/2019  11:39 AM            877 spreadsheet_drawing_with_chart.xml
-a----        7/26/2019  11:39 AM           1681 text_box_drawing.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\formatting
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           2784 formatting.py
-a----        7/26/2019  11:39 AM           9421 rule.py
-a----        7/26/2019  11:39 AM             98 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\formatting\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            484 conftest.py
-a----        7/26/2019  11:39 AM          10683 test_formatting.py
-a----        7/26/2019  11:39 AM          11363 test_rule.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\formatting\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          50266 conditional-formatting.xlsx
-a----        7/26/2019  11:39 AM           7682 worksheet.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\formula
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           1868 excel_tokenizer_parser.txt
-a----        7/26/2019  11:39 AM          14442 tokenizer.py
-a----        7/26/2019  11:39 AM           6711 translate.py
-a----        7/26/2019  11:39 AM            108 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\formula\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          19663 test_tokenizer.py
-a----        7/26/2019  11:39 AM           8346 test_translate.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\packaging
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           4159 core.py
-a----        7/26/2019  11:39 AM           4744 extended.py
-a----        7/26/2019  11:39 AM            959 interface.py
-a----        7/26/2019  11:39 AM           5627 manifest.py
-a----        7/26/2019  11:39 AM           4115 relationship.py
-a----        7/26/2019  11:39 AM           3915 workbook.py
-a----        7/26/2019  11:39 AM             90 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\packaging\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            243 conftest.py
-a----        7/26/2019  11:39 AM           2858 test_core.py
-a----        7/26/2019  11:39 AM           2292 test_extended.py
-a----        7/26/2019  11:39 AM            275 test_interface.py
-a----        7/26/2019  11:39 AM          10007 test_manifest.py
-a----        7/26/2019  11:39 AM           4245 test_relationship.py
-a----        7/26/2019  11:39 AM           4833 test_workbook.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\packaging\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          10004 bug137.xlsx
-a----        7/26/2019  11:39 AM           1100 core.xml
-a----        7/26/2019  11:39 AM           5181 hyperlink.xlsx
-a----        7/26/2019  11:39 AM           1570 manifest.xml
-a----        7/26/2019  11:39 AM          14504 pivot.xlsx
-a----        7/26/2019  11:39 AM           8464 print_settings.xlsx
-a----        7/26/2019  11:39 AM           8352 sample.xlsm
-a----        7/26/2019  11:39 AM            423 workbook_1904.xml
-a----        7/26/2019  11:39 AM            577 workbook_links.xml
-a----        7/26/2019  11:39 AM            641 workbook_missing_id.xml
-a----        7/26/2019  11:39 AM          21086 workbook_security.xlsx


RiverArchitect\program\.site_packages\openpyxl\openpyxl\pivot
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM          30377 cache.py
-a----        7/26/2019  11:39 AM           6984 fields.py
-a----        7/26/2019  11:39 AM           2687 record.py
-a----        7/26/2019  11:39 AM          37642 table.py
-a----        7/26/2019  11:39 AM             74 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\pivot\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            341 conftest.py
-a----        7/26/2019  11:39 AM           5423 test_cache.py
-a----        7/26/2019  11:39 AM           4079 test_fields.py
-a----        7/26/2019  11:39 AM           2730 test_record.py
-a----        7/26/2019  11:39 AM          12660 test_table.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\pivot\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           2284 pivotCacheDefinition.xml
-a----        7/26/2019  11:39 AM           2069 pivotCacheRecords.xml
-a----        7/26/2019  11:39 AM           2725 pivotTable.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\reader
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           9704 excel.py
-a----        7/26/2019  11:39 AM            723 strings.py
-a----        7/26/2019  11:39 AM          13006 worksheet.py
-a----        7/26/2019  11:39 AM             35 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\reader\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            279 conftest.py
-a----        7/26/2019  11:39 AM           4860 test_excel.py
-a----        7/26/2019  11:39 AM            850 test_strings.py
-a----        7/26/2019  11:39 AM           2530 test_style.py
-a----        7/26/2019  11:39 AM          20784 test_worksheet.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\reader\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          10004 bug137.xlsx
-a----        7/26/2019  11:39 AM           9233 complex-styles-worksheet.xml
-a----        7/26/2019  11:39 AM          46700 complex-styles.xlsx
-a----        7/26/2019  11:39 AM          14649 contains_chartsheets.xlsx
-a----        7/26/2019  11:39 AM           9476 empty_with_no_properties.xlsx
-a----        7/26/2019  11:39 AM           5130 extended_conditional_formatting_sheet.xml
-a----        7/26/2019  11:39 AM           1245 frozen_view_worksheet.xml
-a----        7/26/2019  11:39 AM           1245 hidden_rows_cols.xml
-a----        7/26/2019  11:39 AM            936 hidden_sheets.xml
-a----        7/26/2019  11:39 AM          73734 jasper_sheet.xml
-a----        7/26/2019  11:39 AM          12452 legacy_drawing.xlsm
-a----        7/26/2019  11:39 AM          15231 legacy_drawing_worksheet.xml
-a----        7/26/2019  11:39 AM              0 null_file.xlsx
-a----        7/26/2019  11:39 AM           2038 protected_sheet.xml
-a----        7/26/2019  11:39 AM            908 shared-strings-rich.xml
-a----        7/26/2019  11:39 AM            175 sharedStrings-emptystring.xml
-a----        7/26/2019  11:39 AM            190 sharedStrings.xml
-a----        7/26/2019  11:39 AM            730 sharedStrings2.xml
-a----        7/26/2019  11:39 AM           2706 Table1-XmlFromAccess.xml
-a----        7/26/2019  11:39 AM           3517 worksheet_formulae.xml
-a----        7/26/2019  11:39 AM           1099 worksheet_without_coordinates.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\styles
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           2524 alignment.py
-a----        7/26/2019  11:39 AM           3557 borders.py
-a----        7/26/2019  11:39 AM          31221 builtins.py
-a----        7/26/2019  11:39 AM           5343 cell_style.py
-a----        7/26/2019  11:39 AM           5145 colors.py
-a----        7/26/2019  11:39 AM           2288 differential.py
-a----        7/26/2019  11:39 AM           6475 fills.py
-a----        7/26/2019  11:39 AM           3564 fonts.py
-a----        7/26/2019  11:39 AM           7276 named_styles.py
-a----        7/26/2019  11:39 AM           4701 numbers.py
-a----        7/26/2019  11:39 AM            433 protection.py
-a----        7/26/2019  11:39 AM           1495 proxy.py
-a----        7/26/2019  11:39 AM           4241 styleable.py
-a----        7/26/2019  11:39 AM           7254 stylesheet.py
-a----        7/26/2019  11:39 AM           2764 table.py
-a----        7/26/2019  11:39 AM            402 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\styles\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            484 conftest.py
-a----        7/26/2019  11:39 AM            671 test_alignments.py
-a----        7/26/2019  11:39 AM           2161 test_borders.py
-a----        7/26/2019  11:39 AM           7431 test_cell_style.py
-a----        7/26/2019  11:39 AM           1993 test_colors.py
-a----        7/26/2019  11:39 AM           1428 test_differential.py
-a----        7/26/2019  11:39 AM           9306 test_fills.py
-a----        7/26/2019  11:39 AM           1981 test_fonts.py
-a----        7/26/2019  11:39 AM           7388 test_named_style.py
-a----        7/26/2019  11:39 AM           4067 test_number_style.py
-a----        7/26/2019  11:39 AM            385 test_protection.py
-a----        7/26/2019  11:39 AM           1213 test_proxy.py
-a----        7/26/2019  11:39 AM           2918 test_styleable.py
-a----        7/26/2019  11:39 AM          10789 test_stylesheet.py
-a----        7/26/2019  11:39 AM           2255 test_table.py
-a----        7/26/2019  11:39 AM             47 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\styles\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           1320 alignment_styles.xml
-a----        7/26/2019  11:39 AM           2594 builtins_as_custom_number_formats.xml
-a----        7/26/2019  11:39 AM           9677 complex-styles.xml
-a----        7/26/2019  11:39 AM          59885 dxf_style.xml
-a----        7/26/2019  11:39 AM           1693 empty-workbook-styles.xml
-a----        7/26/2019  11:39 AM           1506 none_value_styles.xml
-a----        7/26/2019  11:39 AM           1084 no_number_format.xml
-a----        7/26/2019  11:39 AM          90953 rgb_colors.xml
-a----        7/26/2019  11:39 AM           1737 simple-styles.xml
-a----        7/26/2019  11:39 AM           3831 styles_number_formats.xml
-a----        7/26/2019  11:39 AM           2293 worksheet_unprotected_style.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
d-----        7/26/2019  11:39 AM                long
d-----        7/26/2019  11:39 AM                schemas
-a----        7/26/2019  11:39 AM            592 conftest.py
-a----        7/26/2019  11:39 AM            605 helper.py
-a----        7/26/2019  11:39 AM            511 notes_on_namespaces.rst
-a----        7/26/2019  11:39 AM           1623 schema.py
-a----        7/26/2019  11:39 AM           2038 test_backend.py
-a----        7/26/2019  11:39 AM          13109 test_iter.py
-a----        7/26/2019  11:39 AM           1452 test_read.py
-a----        7/26/2019  11:39 AM           4507 test_vba.py
-a----        7/26/2019  11:39 AM            152 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                genuine
d-----        7/26/2019  11:39 AM                reader
d-----        7/26/2019  11:39 AM                writer
-a----        7/26/2019  11:39 AM           3072 Thumbs.db


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\data\genuine
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           7511 empty-with-styles.xlsx
-a----        7/26/2019  11:39 AM           7842 empty.xlsx
-a----        7/26/2019  11:39 AM           8107 empty_libre.xlsx
-a----        7/26/2019  11:39 AM          10300 empty_no_dimensions.xlsx
-a----        7/26/2019  11:39 AM          30133 guess_types.xlsx
-a----        7/26/2019  11:39 AM           4534 libreoffice_nrt.xlsx
-a----        7/26/2019  11:39 AM          11395 sample.xlsx


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\data\reader
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM         410964 bigfoot.xlsx
-a----        7/26/2019  11:39 AM           1091 bug328_hyperlinks.xml
-a----        7/26/2019  11:39 AM           1276 bug393-worksheet.xml
-a----        7/26/2019  11:39 AM           1382 empty_rows.xml
-a----        7/26/2019  11:39 AM           1681 merged-ranges.xml
-a----        7/26/2019  11:39 AM           6399 nonstandard_workbook_name.xlsx
-a----        7/26/2019  11:39 AM            126 null_archive.xlsx
-a----        7/26/2019  11:39 AM           6075 sheet2.xml
-a----        7/26/2019  11:39 AM           6073 sheet2_invalid_dimension.xml
-a----        7/26/2019  11:39 AM           6046 sheet2_no_dimension.xml
-a----        7/26/2019  11:39 AM           5656 sheet2_no_span.xml
-a----        7/26/2019  11:39 AM          22554 vba+comments.xlsm
-a----        7/26/2019  11:39 AM           6302 vba-comments-saved.xlsm
-a----        7/26/2019  11:39 AM          52938 vba-test.xlsm


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\data\writer
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           2000 Content_types_vba.xml
-a----        7/26/2019  11:39 AM            208 font.xml
-a----        7/26/2019  11:39 AM           1106 styles.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\long
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            555 dump_writer_performance.py
-a----        7/26/2019  11:39 AM            516 long_dump_thread.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\tests\schemas
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          76483 dml-chart.xsd
-a----        7/26/2019  11:39 AM           6956 dml-chartDrawing.xsd
-a----        7/26/2019  11:39 AM            743 dml-compatibility.xsd
-a----        7/26/2019  11:39 AM          52820 dml-diagram.xsd
-a----        7/26/2019  11:39 AM            624 dml-lockedCanvas.xsd
-a----        7/26/2019  11:39 AM         150710 dml-main.xsd
-a----        7/26/2019  11:39 AM           1231 dml-picture.xsd
-a----        7/26/2019  11:39 AM           8862 dml-spreadsheetDrawing.xsd
-a----        7/26/2019  11:39 AM           8973 dml-wordprocessingDrawing.xsd
-a----        7/26/2019  11:39 AM           2565 opc-coreProperties.xsd
-a----        7/26/2019  11:39 AM           1377 opc-relationships.xsd
-a----        7/26/2019  11:39 AM          85246 pml.xsd
-a----        7/26/2019  11:39 AM           1269 shared-additionalCharacteristics.xsd
-a----        7/26/2019  11:39 AM           7328 shared-bibliography.xsd
-a----        7/26/2019  11:39 AM           6141 shared-commonSimpleTypes.xsd
-a----        7/26/2019  11:39 AM           1248 shared-customXmlDataProperties.xsd
-a----        7/26/2019  11:39 AM            880 shared-customXmlSchemaProperties.xsd
-a----        7/26/2019  11:39 AM           2608 shared-documentPropertiesCustom.xsd
-a----        7/26/2019  11:39 AM           3507 shared-documentPropertiesExtended.xsd
-a----        7/26/2019  11:39 AM           7507 shared-documentPropertiesVariantTypes.xsd
-a----        7/26/2019  11:39 AM          23870 shared-math.xsd
-a----        7/26/2019  11:39 AM           1367 shared-relationshipReference.xsd
-a----        7/26/2019  11:39 AM         246625 sml.xsd
-a----        7/26/2019  11:39 AM          26088 vml-main.xsd
-a----        7/26/2019  11:39 AM          25279 vml-officeDrawing.xsd
-a----        7/26/2019  11:39 AM            535 vml-presentationDrawing.xsd
-a----        7/26/2019  11:39 AM           5712 vml-spreadsheetDrawing.xsd
-a----        7/26/2019  11:39 AM           4010 vml-wordprocessingDrawing.xsd
-a----        7/26/2019  11:39 AM         174907 wml.xsd


RiverArchitect\program\.site_packages\openpyxl\openpyxl\utils
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                jdcal
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM            798 bound_dictionary.py
-a----        7/26/2019  11:39 AM           6343 cell.py
-a----        7/26/2019  11:39 AM           2417 dataframe.py
-a----        7/26/2019  11:39 AM           4351 datetime.py
-a----        7/26/2019  11:39 AM            829 escape.py
-a----        7/26/2019  11:39 AM            928 exceptions.py
-a----        7/26/2019  11:39 AM           3733 formulas.py
-a----        7/26/2019  11:39 AM           1257 indexed_list.py
-a----        7/26/2019  11:39 AM            869 protection.py
-a----        7/26/2019  11:39 AM           2629 units.py
-a----        7/26/2019  11:39 AM            391 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\utils\jdcal
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                jdcal.egg-info
-a----        7/26/2019  11:39 AM          12556 jdcal.py
-a----        7/26/2019  11:39 AM          13557 jdcal.pyc
-a----        7/26/2019  11:39 AM           1291 LICENSE.txt
-a----        7/26/2019  11:39 AM             57 MANIFEST.in
-a----        7/26/2019  11:39 AM           5679 PKG-INFO
-a----        7/26/2019  11:39 AM           3921 README.rst
-a----        7/26/2019  11:39 AM             59 setup.cfg
-a----        7/26/2019  11:39 AM            772 setup.py
-a----        7/26/2019  11:39 AM           4118 test_jdcal.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\utils\jdcal\jdcal.egg-info
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM              1 dependency_links.txt
-a----        7/26/2019  11:39 AM           5679 PKG-INFO
-a----        7/26/2019  11:39 AM            182 SOURCES.txt
-a----        7/26/2019  11:39 AM              6 top_level.txt


RiverArchitect\program\.site_packages\openpyxl\openpyxl\utils\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            876 test_bound_dictionary.py
-a----        7/26/2019  11:39 AM           5482 test_cell.py
-a----        7/26/2019  11:39 AM           2222 test_dataframe.py
-a----        7/26/2019  11:39 AM           5464 test_datetime.py
-a----        7/26/2019  11:39 AM            310 test_escape.py
-a----        7/26/2019  11:39 AM           1039 test_indexed_list.py
-a----        7/26/2019  11:39 AM            197 test_protection.py
-a----        7/26/2019  11:39 AM           6675 test_units.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                external_link
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           4173 child.py
-a----        7/26/2019  11:39 AM           7213 defined_name.py
-a----        7/26/2019  11:39 AM            387 external_reference.py
-a----        7/26/2019  11:39 AM            842 function_group.py
-a----        7/26/2019  11:39 AM           6455 parser.py
-a----        7/26/2019  11:39 AM           5300 properties.py
-a----        7/26/2019  11:39 AM           6064 protection.py
-a----        7/26/2019  11:39 AM           1220 smart_tags.py
-a----        7/26/2019  11:39 AM           5253 views.py
-a----        7/26/2019  11:39 AM           2681 web.py
-a----        7/26/2019  11:39 AM          12165 workbook.py
-a----        7/26/2019  11:39 AM            107 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook\external_link
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           4638 external.py
-a----        7/26/2019  11:39 AM           7490 external.pyc
-a----        7/26/2019  11:39 AM            110 __init__.py
-a----        7/26/2019  11:39 AM            311 __init__.pyc


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook\external_link\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            279 conftest.py
-a----        7/26/2019  11:39 AM           4553 test_external.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook\external_link\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          10044 book1.xlsx
-a----        7/26/2019  11:39 AM           8578 book2.xlsx
-a----        7/26/2019  11:39 AM           1779 externalLink1.xml
-a----        7/26/2019  11:39 AM            290 externalLink1.xml.rels
-a----        7/26/2019  11:39 AM           8742 merge_range.xlsx
-a----        7/26/2019  11:39 AM            376 OLELink.xml
-a----        7/26/2019  11:39 AM            885 workbook.xml
-a----        7/26/2019  11:39 AM           1124 workbook.xml.rels
-a----        7/26/2019  11:39 AM           1070 workbook_external_range.xml
-a----        7/26/2019  11:39 AM            999 workbook_namedrange.xml
-a----        7/26/2019  11:39 AM           1584 [Content_Types].xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            279 conftest.py
-a----        7/26/2019  11:39 AM           2918 test_child.py
-a----        7/26/2019  11:39 AM          12101 test_defined_name.py
-a----        7/26/2019  11:39 AM           1116 test_external_reference.py
-a----        7/26/2019  11:39 AM           1602 test_function_group.py
-a----        7/26/2019  11:39 AM           1257 test_parser.py
-a----        7/26/2019  11:39 AM            900 test_pivot.py
-a----        7/26/2019  11:39 AM           2041 test_properties.py
-a----        7/26/2019  11:39 AM           2585 test_protection.py
-a----        7/26/2019  11:39 AM           2042 test_smart_tags.py
-a----        7/26/2019  11:39 AM           2399 test_views.py
-a----        7/26/2019  11:39 AM           2328 test_web.py
-a----        7/26/2019  11:39 AM           5902 test_workbook.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\workbook\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            393 broken_print_titles.xml
-a----        7/26/2019  11:39 AM            428 workbook.xml
-a----        7/26/2019  11:39 AM            715 workbook_russian_code_name.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\worksheet
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM          12921 cell_range.py
-a----        7/26/2019  11:39 AM           2202 copier.py
-a----        7/26/2019  11:39 AM           6201 datavalidation.py
-a----        7/26/2019  11:39 AM           8449 dimensions.py
-a----        7/26/2019  11:39 AM            314 drawing.py
-a----        7/26/2019  11:39 AM          10999 filters.py
-a----        7/26/2019  11:39 AM           8049 header_footer.py
-a----        7/26/2019  11:39 AM           1459 hyperlink.py
-a----        7/26/2019  11:39 AM            807 merge.py
-a----        7/26/2019  11:39 AM           4864 page.py
-a----        7/26/2019  11:39 AM           1664 pagebreak.py
-a----        7/26/2019  11:39 AM           3126 properties.py
-a----        7/26/2019  11:39 AM           3876 protection.py
-a----        7/26/2019  11:39 AM           7638 read_only.py
-a----        7/26/2019  11:39 AM            387 related.py
-a----        7/26/2019  11:39 AM          11228 table.py
-a----        7/26/2019  11:39 AM           4671 views.py
-a----        7/26/2019  11:39 AM          29163 worksheet.py
-a----        7/26/2019  11:39 AM           7983 write_only.py
-a----        7/26/2019  11:39 AM            108 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\worksheet\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            290 conftest.py
-a----        7/26/2019  11:39 AM           8062 test_cell_range.py
-a----        7/26/2019  11:39 AM           5702 test_datavalidation.py
-a----        7/26/2019  11:39 AM           5742 test_dimensions.py
-a----        7/26/2019  11:39 AM           9927 test_filters.py
-a----        7/26/2019  11:39 AM           5854 test_header.py
-a----        7/26/2019  11:39 AM           1929 test_hyperlink.py
-a----        7/26/2019  11:39 AM           3513 test_page.py
-a----        7/26/2019  11:39 AM           1296 test_pagebreak.py
-a----        7/26/2019  11:39 AM           1663 test_properties.py
-a----        7/26/2019  11:39 AM           2272 test_protection.py
-a----        7/26/2019  11:39 AM            640 test_read_only.py
-a----        7/26/2019  11:39 AM            494 test_related.py
-a----        7/26/2019  11:39 AM           6239 test_table.py
-a----        7/26/2019  11:39 AM           2556 test_views.py
-a----        7/26/2019  11:39 AM          19758 test_worksheet.py
-a----        7/26/2019  11:39 AM           3719 test_worksheet_copy.py
-a----        7/26/2019  11:39 AM          12580 test_write_only.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\worksheet\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           4916 copy_test.xlsx
-a----        7/26/2019  11:39 AM            194 sheetPr2.xml
-a----        7/26/2019  11:39 AM           1284 sheet_inline_strings.xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\writer
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           4560 etree_worksheet.py
-a----        7/26/2019  11:39 AM           9836 excel.py
-a----        7/26/2019  11:39 AM            857 strings.py
-a----        7/26/2019  11:39 AM          10402 theme.py
-a----        7/26/2019  11:39 AM           6092 workbook.py
-a----        7/26/2019  11:39 AM           5742 worksheet.py
-a----        7/26/2019  11:39 AM             74 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\writer\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                data
-a----        7/26/2019  11:39 AM            243 conftest.py
-a----        7/26/2019  11:39 AM           4815 test_excel.py
-a----        7/26/2019  11:39 AM            915 test_strings.py
-a----        7/26/2019  11:39 AM           2039 test_template.py
-a----        7/26/2019  11:39 AM            396 test_theme.py
-a----        7/26/2019  11:39 AM           8798 test_workbook.py
-a----        7/26/2019  11:39 AM          17565 test_worksheet.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\writer\tests\data
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           8446 empty.xlsm
-a----        7/26/2019  11:39 AM          11395 empty.xlsx
-a----        7/26/2019  11:39 AM           8450 empty.xltm
-a----        7/26/2019  11:39 AM           8431 empty.xltx
-a----        7/26/2019  11:39 AM            493 plain.png
-a----        7/26/2019  11:39 AM            212 sharedStrings.xml
-a----        7/26/2019  11:39 AM          10140 theme1.xml
-a----        7/26/2019  11:39 AM          22554 vba+comments.xlsm
-a----        7/26/2019  11:39 AM            606 workbook.xml
-a----        7/26/2019  11:39 AM            533 workbook.xml.rels
-a----        7/26/2019  11:39 AM            709 workbook_auto_filter.xml
-a----        7/26/2019  11:39 AM            649 workbook_protection.xml
-a----        7/26/2019  11:39 AM            660 workbook_vba.xml.rels
-a----        7/26/2019  11:39 AM           1594 [Content_Types].xml


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
d-----        7/26/2019  11:39 AM                xmlfile
-a----        7/26/2019  11:39 AM           4305 constants.py
-a----        7/26/2019  11:39 AM           2333 functions.py
-a----        7/26/2019  11:39 AM            750 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           1288 test_functions.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml\xmlfile
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                et_xmlfile
d-----        7/26/2019  11:39 AM                et_xmlfile.egg-info
-a----        7/26/2019  11:39 AM              0 MANIFEST.in
-a----        7/26/2019  11:39 AM           1898 PKG-INFO
-a----        7/26/2019  11:39 AM            977 README.rst
-a----        7/26/2019  11:39 AM             59 _xmlfile.cfg
-a----        7/26/2019  11:39 AM           2062 _xmlfile.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml\xmlfile\et_xmlfile
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                tests
-a----        7/26/2019  11:39 AM           3204 xmlfile.py
-a----        7/26/2019  11:39 AM           4502 xmlfile.pyc
-a----        7/26/2019  11:39 AM            266 __init__.py
-a----        7/26/2019  11:39 AM            535 __init__.pyc


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml\xmlfile\et_xmlfile\tests
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           8732 common_imports.py
-a----        7/26/2019  11:39 AM            605 helper.py
-a----        7/26/2019  11:39 AM          11376 test_incremental_xmlfile.py
-a----        7/26/2019  11:39 AM              0 __init__.py


RiverArchitect\program\.site_packages\openpyxl\openpyxl\xml\xmlfile\et_xmlfile.egg-info
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM              1 dependency_links.txt
-a----        7/26/2019  11:39 AM           1898 PKG-INFO
-a----        7/26/2019  11:39 AM            348 SOURCES.txt
-a----        7/26/2019  11:39 AM             11 top_level.txt


RiverArchitect\program\.site_packages\openpyxl\openpyxl 2.5.2 documentation_files
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          35943 analytics.js
-a----        7/26/2019  11:39 AM           6686 doctools.js
-a----        7/26/2019  11:39 AM          84240 jquery-2.js
-a----        7/26/2019  11:39 AM           8013 jquery-migrate-1.js
-a----        7/26/2019  11:39 AM           5505 logo.png
-a----        7/26/2019  11:39 AM          15414 modernizr.js
-a----        7/26/2019  11:39 AM            390 readthedocs-data.js
-a----        7/26/2019  11:39 AM           3555 readthedocs-doc-embed.css
-a----        7/26/2019  11:39 AM          36087 readthedocs-doc-embed.js
-a----        7/26/2019  11:39 AM            833 readthedocs-dynamic-include.js
-a----        7/26/2019  11:39 AM         112264 sphinx_rtd_theme.css
-a----        7/26/2019  11:39 AM          16254 underscore.js


RiverArchitect\program\.site_packages\riverpy
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           7131 cDefinitions.py
-a----        7/26/2019  11:39 AM           4141 cFeatures.py
-a----        7/26/2019  11:39 AM           5840 cFish.py
-a----       11/19/2019  12:40 PM          19563 cFlows.py
-a----        7/26/2019  11:39 AM          12334 cInputOutput.py
-a----        7/26/2019  11:39 AM           2550 cInterpolator.py
-a----        7/26/2019  11:39 AM           2470 cLogger.py
-a----        7/26/2019  11:39 AM          10583 cMakeTable.py
-a----       11/18/2019   2:11 PM          23510 cMapper.py
-a----         8/6/2019   8:03 AM           2899 config.py
-a----        7/26/2019  11:39 AM           9290 cReachManager.py
-a----        7/26/2019  11:39 AM           4245 cThresholdDirector.py
-a----        9/17/2019  10:03 AM          21388 fGlobal.py
-a----        7/26/2019  11:39 AM            120 __init__.py


RiverArchitect\program\.site_packages\templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          70862 code_icon.ico
-a----       11/19/2019  11:35 AM            134 credits.txt
-a----        7/26/2019  11:39 AM           6172 empty.xlsx
-a----        7/26/2019  11:39 AM          34208 Fish.xlsx
-a----        7/26/2019  11:39 AM          40751 morphological_units.xlsx
-a----        7/26/2019  11:39 AM            164 oups.txt


RiverArchitect\program\00_Flows
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                InputFlowSeries
d-----       11/21/2019   6:10 PM                templates
-a----        7/26/2019  11:39 AM             38 info.md


RiverArchitect\program\00_Flows\InputFlowSeries
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM         252121 flow_series_example_data.xlsx


RiverArchitect\program\00_Flows\templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/21/2019   6:10 PM         188246 disc_freq_template.xlsx
-a----        7/26/2019  11:39 AM         205163 flow_duration_template.xlsx
-a----        7/26/2019  11:39 AM          15163 flow_return_period_template.xlsx


RiverArchitect\program\01_Conditions
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                none
-a----        7/26/2019  11:39 AM       13258925 20XX_sample.zip
-a----        7/26/2019  11:39 AM            208 README.md


RiverArchitect\program\01_Conditions\none
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            111 info.txt
-a----        7/26/2019  11:39 AM             60 __init__.py


RiverArchitect\program\02_Maps
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                templates


RiverArchitect\program\02_Maps\templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                Index
d-----        7/26/2019  11:39 AM                legend
d-----        7/26/2019  11:39 AM                rasters
d-----        7/29/2019   5:48 PM                river_template.gdb
d-----        7/29/2019   5:48 PM                scratch.gdb
d-----        7/26/2019  11:39 AM                symbology
-a----        7/26/2019  11:39 AM         475672 river_template.aprx
-a----        7/26/2019  11:39 AM           4096 river_template.tbx


RiverArchitect\program\02_Maps\templates\Index
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                river_map_template
d-----        7/26/2019  11:39 AM                river_template
d-----        7/26/2019  11:39 AM                Thumbnail


RiverArchitect\program\02_Maps\templates\Index\river_map_template
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             75 log.txt
-a----        7/26/2019  11:39 AM             20 segments.gen
-a----        7/26/2019  11:39 AM            200 segments_2
-a----        7/26/2019  11:39 AM           1509 _0.cfs


RiverArchitect\program\02_Maps\templates\Index\river_template
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             71 log.txt
-a----        7/26/2019  11:39 AM             20 segments.gen
-a----        7/26/2019  11:39 AM           2580 segments_12
-a----        7/26/2019  11:39 AM           2075 segments_z
-a----        7/26/2019  11:39 AM            977 _10.cfs
-a----        7/26/2019  11:39 AM           6183 _m.cfs
-a----        7/26/2019  11:39 AM            673 _n.cfs
-a----        7/26/2019  11:39 AM              9 _n_1.del
-a----        7/26/2019  11:39 AM            807 _o.cfs
-a----        7/26/2019  11:39 AM            688 _p.cfs
-a----        7/26/2019  11:39 AM              9 _p_1.del
-a----        7/26/2019  11:39 AM            837 _q.cfs
-a----        7/26/2019  11:39 AM            688 _r.cfs
-a----        7/26/2019  11:39 AM              9 _r_1.del
-a----        7/26/2019  11:39 AM            805 _s.cfs
-a----        7/26/2019  11:39 AM            688 _t.cfs
-a----        7/26/2019  11:39 AM              9 _t_1.del
-a----        7/26/2019  11:39 AM            849 _u.cfs
-a----        7/26/2019  11:39 AM            789 _v.cfs
-a----        7/26/2019  11:39 AM            805 _w.cfs
-a----        7/26/2019  11:39 AM            804 _x.cfs
-a----        7/26/2019  11:39 AM            737 _y.cfs
-a----        7/26/2019  11:39 AM              9 _y_1.del
-a----        7/26/2019  11:39 AM            973 _z.cfs
-a----        7/26/2019  11:39 AM              9 _z_1.del


RiverArchitect\program\02_Maps\templates\Index\Thumbnail
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM              0 indx
-a----        7/26/2019  11:39 AM           4208 layers.jpg
-a----        7/26/2019  11:39 AM           5833 layers12.jpg
-a----        7/26/2019  11:39 AM           5611 layers2.jpg
-a----        7/26/2019  11:39 AM           6006 layers22.jpg
-a----        7/26/2019  11:39 AM           5851 layers32.jpg
-a----        7/26/2019  11:39 AM           4823 layers5.jpg
-a----        7/26/2019  11:39 AM           5789 layers7.jpg
-a----        7/26/2019  11:39 AM           4346 map.jpg


RiverArchitect\program\02_Maps\templates\legend
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM        1120768 legend.ServerStyle
-a----        7/26/2019  11:39 AM        1622016 legend.style


RiverArchitect\program\02_Maps\templates\rasters
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                ds_bio_s0
d-----        7/26/2019  11:39 AM                ds_elj_dwc
d-----        7/26/2019  11:39 AM                ds_finese_d15
d-----        7/26/2019  11:39 AM                ds_finese_d85
d-----        7/26/2019  11:39 AM                ds_sidec
d-----        7/26/2019  11:39 AM                ds_sidech110
d-----        7/26/2019  11:39 AM                ds_sym_d
d-----        7/26/2019  11:39 AM                ds_widen
d-----        7/26/2019  11:39 AM                lf_sym
-a----        7/29/2019   5:48 PM           1400 ds_bio_s0.aux.xml
-a----        7/29/2019   5:48 PM           1383 ds_elj_dwc.aux.xml
-a----        7/29/2019   5:48 PM           1413 ds_finese_d15.aux.xml
-a----        7/29/2019   5:48 PM           1387 ds_finese_d85.aux.xml
-a----        7/29/2019   5:48 PM           1487 ds_sidec.aux.xml
-a----        7/29/2019   5:48 PM           1407 ds_sidech110.aux.xml
-a----        7/29/2019   5:48 PM           2335 ds_sym_D.aux.xml
-a----        7/29/2019   5:48 PM            504 ds_widen.aux.xml
-a----        7/26/2019  11:39 AM        1274552 layout_lf.xml
-a----        7/29/2019   5:48 PM           2316 lf_sym.aux.xml


RiverArchitect\program\02_Maps\templates\rasters\ds_bio_s0
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          98486 w001001.adf
-a----        7/26/2019  11:39 AM            428 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_elj_dwc
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            170 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          49286 w001001.adf
-a----        7/26/2019  11:39 AM            236 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_finese_d15
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            170 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          98486 w001001.adf
-a----        7/26/2019  11:39 AM            428 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_finese_d85
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            170 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          49286 w001001.adf
-a----        7/26/2019  11:39 AM            236 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_sidec
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/29/2019   5:48 PM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM              8 vat.adf
-a----        7/26/2019  11:39 AM           1662 w001001.adf
-a----        7/26/2019  11:39 AM            428 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_sidech110
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          49286 w001001.adf
-a----        7/26/2019  11:39 AM            236 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_sym_d
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/29/2019   5:48 PM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          32886 w001001.adf
-a----        7/26/2019  11:39 AM            172 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\ds_widen
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/26/2019  11:39 AM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM              8 vat.adf
-a----        7/26/2019  11:39 AM          98486 w001001.adf
-a----        7/26/2019  11:39 AM            428 w001001x.adf


RiverArchitect\program\02_Maps\templates\rasters\lf_sym
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             32 dblbnd.adf
-a----        7/26/2019  11:39 AM            308 hdr.adf
-a----        7/29/2019   5:48 PM            358 prj.adf
-a----        7/26/2019  11:39 AM             32 sta.adf
-a----        7/26/2019  11:39 AM          98486 w001001.adf
-a----        7/26/2019  11:39 AM            428 w001001x.adf


RiverArchitect\program\02_Maps\templates\river_template.gdb
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            110 a00000001.gdbindexes
-a----        7/26/2019  11:39 AM            302 a00000001.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000001.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000001.TablesByName.atx
-a----        7/26/2019  11:39 AM           2055 a00000002.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000002.gdbtablx
-a----        7/26/2019  11:39 AM             42 a00000003.gdbindexes
-a----        7/26/2019  11:39 AM            525 a00000003.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000003.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000004.CatItemsByPhysicalName.atx
-a----        7/26/2019  11:39 AM           4118 a00000004.CatItemsByType.atx
-a----        7/26/2019  11:39 AM           4118 a00000004.FDO_UUID.atx
-a----        7/26/2019  11:39 AM            310 a00000004.gdbindexes
-a----        7/26/2019  11:39 AM           1712 a00000004.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000004.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000004.spx
-a----        7/26/2019  11:39 AM          12310 a00000005.CatItemTypesByName.atx
-a----        7/26/2019  11:39 AM           4118 a00000005.CatItemTypesByParentTypeID.atx
-a----        7/26/2019  11:39 AM           4118 a00000005.CatItemTypesByUUID.atx
-a----        7/26/2019  11:39 AM            296 a00000005.gdbindexes
-a----        7/26/2019  11:39 AM           2074 a00000005.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000005.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByDestinationID.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByOriginID.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByType.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.FDO_UUID.atx
-a----        7/26/2019  11:39 AM            318 a00000006.gdbindexes
-a----        7/26/2019  11:39 AM            190 a00000006.gdbtable
-a----        7/26/2019  11:39 AM             32 a00000006.gdbtablx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByBackwardLabel.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByDestItemTypeID.atx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByForwardLabel.atx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByName.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByOriginItemTypeID.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByUUID.atx
-a----        7/26/2019  11:39 AM            602 a00000007.gdbindexes
-a----        7/26/2019  11:39 AM           3479 a00000007.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000007.gdbtablx
-a----        7/26/2019  11:39 AM              4 gdb
-a----        7/26/2019  11:39 AM            400 timestamps


RiverArchitect\program\02_Maps\templates\scratch.gdb
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            110 a00000001.gdbindexes
-a----        7/26/2019  11:39 AM            302 a00000001.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000001.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000001.TablesByName.atx
-a----        7/26/2019  11:39 AM           2055 a00000002.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000002.gdbtablx
-a----        7/26/2019  11:39 AM             42 a00000003.gdbindexes
-a----        7/26/2019  11:39 AM            525 a00000003.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000003.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000004.CatItemsByPhysicalName.atx
-a----        7/26/2019  11:39 AM           4118 a00000004.CatItemsByType.atx
-a----        7/26/2019  11:39 AM           4118 a00000004.FDO_UUID.atx
-a----        7/26/2019  11:39 AM            310 a00000004.gdbindexes
-a----        7/26/2019  11:39 AM           1712 a00000004.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000004.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000004.spx
-a----        7/26/2019  11:39 AM          12310 a00000005.CatItemTypesByName.atx
-a----        7/26/2019  11:39 AM           4118 a00000005.CatItemTypesByParentTypeID.atx
-a----        7/26/2019  11:39 AM           4118 a00000005.CatItemTypesByUUID.atx
-a----        7/26/2019  11:39 AM            296 a00000005.gdbindexes
-a----        7/26/2019  11:39 AM           2074 a00000005.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000005.gdbtablx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByDestinationID.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByOriginID.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.CatRelsByType.atx
-a----        7/26/2019  11:39 AM           4118 a00000006.FDO_UUID.atx
-a----        7/26/2019  11:39 AM            318 a00000006.gdbindexes
-a----        7/26/2019  11:39 AM            190 a00000006.gdbtable
-a----        7/26/2019  11:39 AM             32 a00000006.gdbtablx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByBackwardLabel.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByDestItemTypeID.atx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByForwardLabel.atx
-a----        7/26/2019  11:39 AM          12310 a00000007.CatRelTypesByName.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByOriginItemTypeID.atx
-a----        7/26/2019  11:39 AM           4118 a00000007.CatRelTypesByUUID.atx
-a----        7/26/2019  11:39 AM            602 a00000007.gdbindexes
-a----        7/26/2019  11:39 AM           3479 a00000007.gdbtable
-a----        7/26/2019  11:39 AM           5152 a00000007.gdbtablx
-a----        7/26/2019  11:39 AM              4 gdb
-a----        7/26/2019  11:39 AM            400 timestamps


RiverArchitect\program\02_Maps\templates\symbology
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           7666 LifespanRasterSymbology.lyrx


RiverArchitect\program\StrandingRisk
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        8/20/2019  12:22 PM                .templates
-a----       11/21/2019   6:10 PM          28031 cConnectivityAnalysis.py
-a----       11/22/2019  10:17 AM          16505 cGraph.py
-a----       11/19/2019  12:40 PM          12408 connect_gui.py
-a----       11/19/2019  12:40 PM           4199 cRatingCurves.py
-a----        7/26/2019  11:39 AM             84 __init__.py


RiverArchitect\program\StrandingRisk\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        8/20/2019  12:22 PM         191729 disconnected_area_template.xlsx


RiverArchitect\program\docs
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM           3358 CODE_OF_CONDUCT.md
-a----        7/26/2019  11:39 AM           7504 CONTRIBUTING.md
-a----        7/26/2019  11:39 AM            802 PULL_REQUEST_TEMPLATE.md


RiverArchitect\program\GetStarted
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
-a----       11/19/2019  12:40 PM          18968 cConditionCreator.py
-a----        7/26/2019  11:39 AM           7321 cDetrendedDEM.py
-a----        7/26/2019  11:39 AM           4387 cMakeInp.py
-a----        7/26/2019  11:39 AM           8722 cMorphUnits.py
-a----        8/20/2019  12:22 PM          19556 cWaterLevel.py
-a----        7/26/2019  11:39 AM           3542 fSubCondition.py
-a----       10/30/2019   4:27 PM           4851 popup_align_rasters.py
-a----        11/4/2019  10:43 AM          10762 popup_analyze_q.py
-a----       10/30/2019   4:27 PM          18197 popup_create_c.py
-a----        9/17/2019  10:19 AM           7493 popup_create_c_sub.py
-a----        7/26/2019  11:39 AM           4329 popup_make_inp.py
-a----        9/17/2019  10:19 AM          13449 popup_populate_c.py
-a----       10/30/2019   4:27 PM           6226 welcome_gui.py
-a----        7/26/2019  11:39 AM             80 __init__.py


RiverArchitect\program\GetStarted\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM         143253 welcome.gif


RiverArchitect\program\LifespanDesign
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/25/2019   6:43 PM                .templates
d-----        7/26/2019  11:39 AM                Output
-a----       10/24/2019   2:47 PM          53673 cLifespanDesignAnalysis.py
-a----       10/29/2019   6:17 PM          10816 cParameters.py
-a----        7/26/2019  11:39 AM           5776 cReadInpLifespan.py
-a----       10/23/2019   3:48 PM          16584 feature_analysis.py
-a----       11/25/2019   4:10 PM          21688 lifespan_design_gui.py
-a----        7/26/2019  11:39 AM            113 __init__.py


RiverArchitect\program\LifespanDesign\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            703 mapping.inp
-a----        7/26/2019  11:39 AM            773 mapping_full.inp
-a----        7/26/2019  11:39 AM          11323 prepare_map_coordinates.xlsx
-a----       11/25/2019   4:10 PM          23190 threshold_values.xlsx


RiverArchitect\program\LifespanDesign\Output
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                Rasters


RiverArchitect\program\LifespanDesign\Output\Rasters
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             58 __init__.py


RiverArchitect\program\MaxLifespan
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
d-----        7/26/2019  11:39 AM                Output
-a----       11/25/2019   4:11 PM          14301 action_gui.py
-a----        7/26/2019  11:39 AM           2934 action_planner.py
-a----        7/26/2019  10:50 AM          11193 cActionAssessment.py
-a----        7/26/2019  11:39 AM           3558 cFeatureActions.py
-a----        7/26/2019  11:39 AM            102 __init__.py


RiverArchitect\program\MaxLifespan\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                rasters
-a----        7/26/2019  11:39 AM            704 mapping.inp
-a----        7/26/2019  11:39 AM          11323 prepare_map_coordinates.xlsx


RiverArchitect\program\MaxLifespan\.templates\rasters
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             91 zeros.tfw
-a----        7/26/2019  11:39 AM           9445 zeros.tif
-a----        7/26/2019  11:39 AM            364 zeros.tif.aux.xml


RiverArchitect\program\MaxLifespan\Output
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                Rasters


RiverArchitect\program\MaxLifespan\Output\Rasters
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\ModifyTerrain
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
d-----        7/26/2019  11:39 AM                Output
d-----        7/26/2019  11:39 AM                RiverBuilder
-a----        7/26/2019  10:39 AM          12435 cModifyTerrain.py
-a----        7/26/2019  11:39 AM           1941 cRiverBuilder.py
-a----        7/26/2019  11:39 AM          12125 cRiverBuilderConstruct.py
-a----         8/6/2019   8:03 AM          13861 modify_terrain_gui.py
-a----        7/26/2019  11:39 AM           3777 riverbuilder_input_example.txt
-a----        9/17/2019  10:27 AM          15650 sub_gui_rb.py
-a----        7/26/2019  11:39 AM             87 __init__.py


RiverArchitect\program\ModifyTerrain\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM         382805 chnl_xs_parameters.gif
-a----        7/26/2019  11:39 AM          10681 computation_extents.xlsx
-a----        7/26/2019  11:39 AM        1024000 determine_extents.mxd
-a----        7/26/2019  11:39 AM            593 determine_extents.xml
-a----        7/26/2019  11:39 AM            704 mapping.inp
-a----        7/26/2019  11:39 AM          11323 prepare_map_coordinates.xlsx


RiverArchitect\program\ModifyTerrain\Output
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                Grade
d-----        7/26/2019  11:39 AM                RiverBuilder
d-----        7/26/2019  11:39 AM                Widen


RiverArchitect\program\ModifyTerrain\Output\Grade
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\ModifyTerrain\Output\RiverBuilder
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\ModifyTerrain\Output\Widen
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\ModifyTerrain\RiverBuilder
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                messages
-a----        7/26/2019  11:39 AM          45428 riverbuilder.r
-a----        7/26/2019  11:39 AM           3117 RiverBuilderRenderer.py


RiverArchitect\program\ModifyTerrain\RiverBuilder\messages
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM            537 chnl_xs_pts.txt
-a----        7/26/2019  11:39 AM            391 d50.txt
-a----        7/26/2019  11:39 AM            730 datum.txt
-a----        7/26/2019  11:39 AM             81 dy.txt
-a----        7/26/2019  11:39 AM            292 floodplain_outer_height.txt
-a----        7/26/2019  11:39 AM            270 h_bf.txt
-a----        7/26/2019  11:39 AM            381 length.txt
-a----        7/26/2019  11:39 AM            935 sub_curvature.txt
-a----        7/26/2019  11:39 AM            285 sub_floodplain_l.txt
-a----        7/26/2019  11:39 AM            150 sub_floodplain_r.txt
-a----        7/26/2019  11:39 AM            568 sub_meander.txt
-a----        7/26/2019  11:39 AM            149 sub_thalweg.txt
-a----        7/26/2019  11:39 AM            667 sub_w_bf.txt
-a----        7/26/2019  11:39 AM            204 s_valley.txt
-a----        7/26/2019  11:39 AM            572 taux.txt
-a----        7/26/2019  11:39 AM            300 terrace_outer_height.txt
-a----        7/26/2019  11:39 AM           1391 user_functions.txt
-a----        7/26/2019  11:39 AM            150 w_bf.txt
-a----        7/26/2019  11:39 AM            431 w_bf_min.txt
-a----        7/26/2019  11:39 AM            343 w_boundary.txt
-a----        7/26/2019  11:39 AM            831 w_floodplain.txt
-a----        7/26/2019  11:39 AM            419 w_terrace.txt
-a----        7/26/2019  11:39 AM            585 xs_shape.txt
-a----        7/26/2019  11:39 AM            553 x_res.txt


RiverArchitect\program\ProjectMaker
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/29/2019   5:48 PM                .templates
-a----        10/2/2019   9:57 AM          10092 cSHArC.py
-a----        7/26/2019  11:39 AM           4245 fFunctions.py
-a----       11/25/2019   4:13 PM          34810 project_maker_gui.py
-a----       11/19/2019   9:32 AM          11784 s20_plantings_delineation.py
-a----       11/19/2019   9:43 AM           8778 s21_plantings_stabilization.py
-a----       10/24/2019   2:19 PM          11928 s30_terrain_stabilization.py
-a----       11/12/2019   2:21 PM           8488 s40_compare_sharea.py
-a----        7/26/2019  11:39 AM            217 __init__.py


RiverArchitect\program\ProjectMaker\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/25/2019   4:08 PM                Project_vii_TEMPLATE
-a----        7/26/2019  11:39 AM              5 area_dummy.cpg
-a----        7/26/2019  11:39 AM            118 area_dummy.dbf
-a----        7/26/2019  11:39 AM            132 area_dummy.sbn
-a----        7/26/2019  11:39 AM            116 area_dummy.sbx
-a----        7/26/2019  11:39 AM            220 area_dummy.shp
-a----        7/26/2019  11:39 AM            108 area_dummy.shx


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/19/2019  10:27 AM                Geodata
d-----        7/26/2019  11:39 AM                Index
d-----       10/30/2019   5:57 PM                ProjectMaps.gdb
d-----       11/19/2019   9:41 AM                Quantities
-a----        11/7/2019   5:51 PM         203109 ProjectMaps.aprx
-a----        4/26/2019   9:43 AM           4096 ProjectMaps.tbx
-a----       11/25/2019   4:07 PM          35894 Project_assessment_vii.xlsx


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Geodata
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/19/2019   9:12 AM                Rasters
d-----       11/19/2019   9:12 AM                Shapefiles
-a----        7/26/2019  11:39 AM          20814 SHArea_evaluation_template_si.xlsx
-a----        7/26/2019  11:39 AM          20797 SHArea_evaluation_template_us.xlsx


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Geodata\Rasters
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/19/2019   9:12 AM             10 __init__.py


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Geodata\Shapefiles
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/19/2019   9:12 AM             10 __init__.py


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Index
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        11/5/2019   1:57 PM                ProjectMaps
d-----        7/26/2019  11:39 AM                Thumbnail


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Index\ProjectMaps
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/26/2019  10:18 AM             20 segments.gen
-a----        4/26/2019  10:18 AM           1067 segments_9
-a----        4/26/2019   9:43 AM           1122 _1.cfs
-a----        4/26/2019   9:43 AM            633 _2.cfs
-a----        4/26/2019  10:17 AM            669 _3.cfs
-a----        4/26/2019  10:18 AM              9 _3_1.del
-a----        4/26/2019  10:17 AM            812 _4.cfs
-a----        4/26/2019  10:18 AM            708 _5.cfs
-a----        4/26/2019  10:18 AM              9 _5_1.del
-a----        4/26/2019  10:18 AM            806 _6.cfs


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Index\Thumbnail
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/26/2019  10:17 AM              0 indx
-a----        4/26/2019  10:18 AM           1827 layers.jpg


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\ProjectMaps.gdb
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/26/2019   9:43 AM            110 a00000001.gdbindexes
-a----       10/25/2019   8:54 AM            338 a00000001.gdbtable
-a----       10/25/2019   8:54 AM           5152 a00000001.gdbtablx
-a----       10/25/2019   8:54 AM           4118 a00000001.TablesByName.atx
-a----        4/26/2019   9:43 AM           2055 a00000002.gdbtable
-a----        4/26/2019   9:43 AM           5152 a00000002.gdbtablx
-a----        4/26/2019   9:43 AM             42 a00000003.gdbindexes
-a----        4/26/2019   9:43 AM            525 a00000003.gdbtable
-a----        4/26/2019   9:43 AM           5152 a00000003.gdbtablx
-a----       10/25/2019   8:54 AM           4118 a00000004.CatItemsByPhysicalName.atx
-a----       10/25/2019   8:54 AM           4118 a00000004.CatItemsByType.atx
-a----       10/25/2019   8:54 AM           4118 a00000004.FDO_UUID.atx
-a----       10/25/2019   8:54 AM           8536 a00000004.freelist
-a----        4/26/2019   9:43 AM            310 a00000004.gdbindexes
-a----       10/25/2019   8:54 AM          24787 a00000004.gdbtable
-a----       10/25/2019   8:54 AM           5152 a00000004.gdbtablx
-a----       10/25/2019   8:54 AM           4118 a00000004.spx
-a----        4/26/2019   9:43 AM          12310 a00000005.CatItemTypesByName.atx
-a----        4/26/2019   9:43 AM           4118 a00000005.CatItemTypesByParentTypeID.atx
-a----        4/26/2019   9:43 AM           4118 a00000005.CatItemTypesByUUID.atx
-a----        4/26/2019   9:43 AM            296 a00000005.gdbindexes
-a----        4/26/2019   9:43 AM           2074 a00000005.gdbtable
-a----        4/26/2019   9:43 AM           5152 a00000005.gdbtablx
-a----       10/25/2019   8:54 AM           4118 a00000006.CatRelsByDestinationID.atx
-a----       10/25/2019   8:54 AM           4118 a00000006.CatRelsByOriginID.atx
-a----       10/25/2019   8:54 AM           4118 a00000006.CatRelsByType.atx
-a----       10/25/2019   8:54 AM           4118 a00000006.FDO_UUID.atx
-a----        4/26/2019   9:43 AM            318 a00000006.gdbindexes
-a----       10/25/2019   8:54 AM            263 a00000006.gdbtable
-a----       10/25/2019   8:54 AM           5152 a00000006.gdbtablx
-a----        4/26/2019   9:43 AM          12310 a00000007.CatRelTypesByBackwardLabel.atx
-a----        4/26/2019   9:43 AM           4118 a00000007.CatRelTypesByDestItemTypeID.atx
-a----        4/26/2019   9:43 AM          12310 a00000007.CatRelTypesByForwardLabel.atx
-a----        4/26/2019   9:43 AM          12310 a00000007.CatRelTypesByName.atx
-a----        4/26/2019   9:43 AM           4118 a00000007.CatRelTypesByOriginItemTypeID.atx
-a----        4/26/2019   9:43 AM           4118 a00000007.CatRelTypesByUUID.atx
-a----        4/26/2019   9:43 AM            602 a00000007.gdbindexes
-a----        4/26/2019   9:43 AM           3479 a00000007.gdbtable
-a----        4/26/2019   9:43 AM           5152 a00000007.gdbtablx
-a----       10/25/2019   8:54 AM             66 a00000009.gdbindexes
-a----       10/25/2019   8:54 AM            168 a00000009.gdbtable
-a----       10/25/2019   8:54 AM           5152 a00000009.gdbtablx
-a----        4/26/2019   9:43 AM              4 gdb
-a----       10/25/2019   8:54 AM            400 timestamps


RiverArchitect\program\ProjectMaker\.templates\Project_vii_TEMPLATE\Quantities
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/19/2019   9:12 AM             10 __init__.py


RiverArchitect\program\SHArC
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
d-----        7/26/2019  11:39 AM                CHSI
d-----        7/26/2019  11:39 AM                HSI
d-----        7/26/2019  11:39 AM                SHArea
-a----       11/21/2019   6:14 PM          38809 cHSI.py
-a----       10/22/2019   2:31 PM          32273 sharc_gui.py
-a----        7/26/2019  11:12 AM          17490 sub_gui_covhsi.py
-a----       10/30/2019   5:10 PM           9689 sub_gui_hhsi.py
-a----        7/26/2019  11:39 AM            122 __init__.py


RiverArchitect\program\SHArC\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM         242416 CONDITION_QvsA_template_si.xlsx
-a----        7/26/2019  11:39 AM         242653 CONDITION_QvsA_template_us.xlsx
-a----        7/26/2019  11:39 AM          17940 CONDITION_sharea_template_si.xlsx
-a----        7/26/2019  11:39 AM          17923 CONDITION_sharea_template_us.xlsx
-a----        7/26/2019  11:39 AM           6172 empty.xlsx


RiverArchitect\program\SHArC\CHSI
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\SHArC\HSI
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\SHArC\SHArea
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py


RiverArchitect\program\Tools
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
-a----       11/25/2019   1:05 PM           6547 cHydraulic.py
-a----        7/26/2019  11:39 AM           8790 cInputOutput.py
-a----       11/25/2019   1:05 PM           6126 cPoolRiffle.py
-a----       11/25/2019   1:05 PM           8129 fTools.py
-a----       11/25/2019   1:05 PM           3112 make_annual_flow_duration.py
-a----       11/25/2019   1:05 PM           1687 make_annual_peak.py
-a----        7/26/2019  11:39 AM           1554 morphology_designer.py
-a----       10/30/2019  12:49 PM           1218 rename_files.py
-a----        7/26/2019  11:39 AM            137 run_make_annual_flow_duration.bat
-a----        7/26/2019  11:39 AM            132 run_make_d2w.bat
-a----        7/26/2019  11:39 AM            124 run_make_det.bat
-a----        7/26/2019  11:39 AM            124 run_make_det_gen.bat
-a----        7/26/2019  11:39 AM            126 run_make_mu.bat
-a----        7/26/2019  11:39 AM            147 run_morphology_designer.bat


RiverArchitect\program\Tools\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          11618 annual_peak_template.xlsx
-a----        7/26/2019  11:39 AM          20867 flow_duration_template.xlsx
-a----        7/26/2019  11:39 AM          11763 input.xlsx
-a----        7/26/2019  11:39 AM          13784 output.xlsx


RiverArchitect\program\VolumeAssessment
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/26/2019  11:39 AM                .templates
d-----        7/26/2019  11:39 AM                Output
-a----       11/21/2019   6:07 PM          10217 cVolumeAssessment.py
-a----        7/26/2019  11:35 AM           7013 volume_gui.py
-a----        7/26/2019  11:39 AM             79 __init__.py


RiverArchitect\program\VolumeAssessment\.templates
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM          10125 volumes_template.xlsx


RiverArchitect\program\VolumeAssessment\Output
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/26/2019  11:39 AM             62 __init__.py

```






