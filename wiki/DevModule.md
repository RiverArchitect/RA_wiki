DEVELOPMENT
===========

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


# Add a module to *River Architect* <a name="addmod"></a>

*River Architect* has currently five **master** tabs (or modules) and three of them have **slave**-tabs (sub-modules): [Lifespan](LifespanDesign), [Morphology](ModifyTerrain), and [Ecohydraulics](SHArC). The development of a new module or sub-module follows the same standards and varies only in the way how the tab is finally bound in the *River Architect* master GUI. For creating a new (sub) module, do the following:

1. Have a look at *River Architect* and think about the capacities that the new module should have, and how it can be fitted in the existing framework (think twice before adding a new master tab).
1. Clone the template respository: `git clone https://github.com/RiverArchitect/development.git`.
1. Frome the cloned repository, copy the folder `RiverArchitect/development/moduleTEMPLATE/` to `RiverArchitect/moduleTEMPLATE/`.
1. Rename `RiverArchitect/moduleTEMPLATE/` to  `RiverArchitect/NEW_MODULE_NAME/` (obviously, replace `NEW_MODULE_NAME` with the name of the new module).
1. Edit and add the module files `RiverArchitect/NEW_MODULE_NAME/cTEMPLATE.py` and `-- TEMPLATE_gui.py` (see below descriptions of the [module template](#template)).
1. Bind the new module to the *River Architect* master GUI (see below descriptions for [binding a new module to the *River Architect* master GUI](#bind)).


## Working with the module template <a name="template"></a>
***
The *River Architect* [development repository](https://github.com/RiverArchitect/development) provides template file for creating a new module in the `moduleTEMPLATE/` directory:

- 	[`TEMPLATE_gui.py`](#templategui) is a template of the actual GUI that will be shown in the new tab.
-	[`cTEMPLATE.py`](#templateclass) is a template for writing new classes using geospatial analysis with `arcpy.sa`.

Inline comments guide trhough the scripts and their dependencies. Both scripts load all functions and load most of the required packages from `RiverArchitect/.site_packages/riverpy/fGlobal.py`.

### Using `TEMPLATE_gui.py` <a name="templategui"></a>

The script contains [`tkinter`](https://effbot.org/tkinterbook/) classes to build the tab GUI. The first class is a `PopUpWindow` that may be useful for inquiring complex user input (e.g., calculate a value). A good example for using complex user input within *River Architect* is the [River Builder input file creator](RiverBuilder#cinp). 

```python
class PopUpWindow(object):
    def __init__(self, master):
		... 
```

The most relevant template class here is `class MainGui(sg.RaModuleGui)`, which inherits the tab-slave class `RaModuleGui` from `RiverArchitect/slave_gui.py`. **All tabs must inherit `RiverArchitect/slave_gui.RaModuleGui`**, no matter if this will be a master or a slave tab. As such, all tabs inherit the following class variables (relevant variables only are listed here):

-	`self.condition` is a `STR` of a condition contained in `RiverArchitect/01_Conditions/`
-	`self.features` is a `cDef.FeatureDefinitions()` object resulting from `RiverArchitect/LifespanDesign/.templates/threshold_values.xlsx` ([read more about features](River-design-features))
-	`self.reaches` is a `cDef.ReachDefinitions()` object resulting from `RiverArchitect/ModifyTerrain/.templates/computation_extents.xlsx` ([read more about river reaches](RiverReaches))
-	`self.units` is a `STR` variable, which is either `us` (US costumary) or `si` (SI Metric)
-	`self.ww` is an `INT` variable defining the tab window width
-	`self.wh` is an `INT` variable defining the tab window height
-	`self.wx` is an `INT` variable defining the tab window x-position
-	`self.wy` is an `INT` variable defining the tab window y-position
-	`self.xd` is an `INT` variable defining the x-pad (pixel space around `tkinter` objects)
-	`self.yd` is an `INT` variable defining the y-pad (pixel space around `tkinter` objects)

Moreover, the `RaModuleGui` class automatically creates a `"Units"` and a `"Close"` dropdown menu.





### Using `cTEMPLATE.py` <a name="templategui"></a>


## Binding a new tab (module) to the *River Architect* master GUI <a name="bind"></a>
***

