DEVELOPMENT
===========

***

- [**Principles & Requiremments**](#raprin)
- [**Add a module to River Architect**](#addmod)
  * [Use Python templates](#template)
  * [Bind the new module to the GUI](#bind)
- [Modify the Wiki](DevWiki)
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

The *River Architect Wiki* for developers assumes a basic understanding of *Python3*, object-oriented programming, the *markdown* language and basics in *HTML*.


# Add a module to *River Architect* <a name="addmod"></a>
***
Overview
-	[Use Python templates](#template)
-	[Bind the new module to the GUI](#bind)
***

New modules or code modifications should be pushed to the `program` repository [[https://github.com/RiverArchitect/program]] ([read more about using `git`](DevGit)).

*River Architect* has currently five **master** tabs (or modules) and three of them have **slave**-tabs (sub-modules): [Lifespan](LifespanDesign), [Morphology](ModifyTerrain), and [Ecohydraulics](SHArC). The development of a new module or sub-module follows the same standards and varies only in the way how the tab is finally bound in the *River Architect* master GUI. For creating a new (sub) module, do the following:

1. Have a look at *River Architect* and think about the capacities that the new module should have, and how it can be fitted in the existing framework (think twice before adding a new master tab).
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
		+ `self.sub_tab_names = [['Lifespan Design', 'Max Lifespan'], ['Modify Terrain', 'Volume Assessment'], ['Habitat Area (SHArC)', 'Habitat Connectivity'], ['NEW SUB1', 'NEW SUB2', ... and so on]]`
-	`self.tab_dir_names`
		+ No sub tabs: `self.tab_dir_names= ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\'], ['\\SHArC\\', '\\Connectivity\\'], '\\ProjectMaker\\', '\\NewModule\\']` 
		+ With sub tabs located in `RiverArchitect/NewSubModule1/` and  `RiverArchitect/NewSubModule2/`, respectively: `self.tab_dir_names = ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\'], ['\\SHArC\\', '\\Connectivity\\'], '\\ProjectMaker\\', ['\\NewSubModule1\\', '\\NewSubModule2\\']]`
-	`self.tab_list`
		+ No sub tabs: `self.tab_list= [GetStarted.welcome_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ProjectMaker.project_maker_gui.MainGui(self.tab_container), NewModule.new_module_gui.MainGui(self.tab_container)]`
		+ With sub tabs: `self.tab_list= [GetStarted.welcome_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ttk.Notebook(self.tab_container), ProjectMaker.project_maker_gui.MainGui(self.tab_container), ttk.Notebook(self.tab_container)]`
- 	With sub tabs: `self.sub_tab_list = [[LifespanDesign.lifespan_design_gui.FaGui(self.tabs['Lifespan']), MaxLifespan.action_gui.ActionGui(self.tabs['Lifespan'])],  [ModifyTerrain.modify_terrain_gui.MainGui(self.tabs['Morphology']),  VolumeAssessment.volume_gui.MainGui(self.tabs['Morphology'])], [SHArC.sharc_gui.MainGui(self.tabs['Ecohydraulics']), Connectivity.connect_gui.MainGui(self.tabs['Ecohydraulics'])], [NewSubModule1.new_sub1_gui.MainGui(self.tabs['NEW SUB1']), NewSubModule2.new_sub2_gui.MainGui(self.tabs['NEW SUB2'])]]`


### Binding a New Sub-Module to an existing module

The following list indicates variables to be modified when a new sub-module (slave or sub-tab) is added to an existing module. The example assumes the creation of a sub-tab called `RiverCreator` to the `Morphology` module.

-	Verify that the `Morphology` tab is mentioned in  `self.sub_tab_parents = ['Lifespan', 'Morphology', 'Ecohydraulics', 'NEW MODULE']`
-	Add `RIVER CREATOR` to the second nested list in `self.sub_tab_names = [['Lifespan Design', 'Max Lifespan'], ['Modify Terrain', 'Volume Assessment', 'RIVER CREATOR'], ['Habitat Area (SHArC)', 'Habitat Connectivity']]`; the second nested list must be chosen because `Morphology` is the second item in `self.sub_tab_parents`
-	Assuming that the new module is located in `RiverArchitect/RiverCreator`, modify `self.tab_dir_names = ['\\GetStarted\\', ['\\LifespanDesign\\', '\\MaxLifespan\\'], ['\\ModifyTerrain\\', '\\VolumeAssessment\\', '\\RiverCreator\\'], ['\\SHArC\\', '\\Connectivity\\'], '\\ProjectMaker\\', ['\\NewSubModule1\\', '\\NewSubModule2\\']]`
- 	Withe the asumption that the new *River Creator* GUI is located in `RiverArchitect/RiverCreator/rc_gui.py`, modify `self.sub_tab_list = [[LifespanDesign.lifespan_design_gui.FaGui(self.tabs['Lifespan']), MaxLifespan.action_gui.ActionGui(self.tabs['Lifespan'])], [ModifyTerrain.modify_terrain_gui.MainGui(self.tabs['Morphology']), VolumeAssessment.volume_gui.MainGui(self.tabs['Morphology']),  RiverCreator.rc_gui.MainGui(self.tabs['River Creator'])], [SHArC.sharc_gui.MainGui(self.tabs['Ecohydraulics']), Connectivity.connect_gui.MainGui(self.tabs['Ecohydraulics'])]]`


**Please do not forget to [update the wiki](DevWiki) and test new code!**










