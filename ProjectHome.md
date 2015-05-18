To begin with, I ask for everyone's patience, as this is my first "proper" open-source release.

This is IDAOCaml v0.1, an OCaml interpreter that runs inside of the IDA Pro disassembler.  It is first and foremost a **research platform**, meaning that there is strictly speaking no support, and it supports only the minimal amount of functionality needed to build a program analysis platform.  It is distributed under the Qt Public License (QPL), the same license as OCaml itself (except that OCaml has a linking exemption).  Despite the lack of support, I will attempt to provide assistance with no guarantees.  (There is at least one report of successful compilation in the wild).  You can email me at rolf rolles at gmail com.  I ask that if you use this platform for academic research, that you cite IDAOCaml at the project page.

The goal in releasing IDAOCaml is to foster collaboration between academics and industrial researchers:  to draw academics closer into the space inhabited by practitioners and give them a practical outlet for their research, and allow practitioners more direct access to the fruits of academic research.  Getting more industrial researchers interested in OCaml and program analysis, and more academics interested in hands-on reverse engineering, is a hopeful side benefit as well.

I wanted an OCaml interpreter for the inside of IDA, the way there are Python, Ruby, Java, Javascript, etc. interpreters, so that I could use IDA as an interactive development environment while I built a program analysis framework.  IDAOCaml was a success in this regard.  However, one of the goals of the framework was to not rely upon IDA for anything essential, such that whatever tools I developed would work outside of IDA seamlessly.  Therefore, I did not want access to much of IDA's functionality, lest I be tempted to make unportable tools, and that is why there is minimal integration with IDA's SDK.  **If you want additional IDA SDK functionality, you will have to write C code to expose it to the OCaml layer.**  Other people are free to investigate the use of SWIG, etc. to automate wrapping of other SDK functionality if they so choose.

# Functionality #

The plugin currently supports:

  * Displaying arbitrary graphs, including coloring for the vertex text
  * Getting and setting of comments
  * Getting and setting of the screen EA
  * msg() and warning()
  * Getting of bytes, words, and dwords
  * Finding the beginning of functions
  * Dynamic adding and removing of hotkeys (limited support due to what seems to be bugs in IDA 5.6)
  * Unfinished tracing support

Believe it or not, this is all one needs for a decent static binary program analysis platform.  I use get\_byte () to feed into my disassembler, which then feeds into my IR translator, which I can then analyze and display the results using the graph interface, comments and msg().

The release comes with two demos in the /samples directory:

  * A demonstration of how to use the graphing interface
  * A demonstration of how to use the hotkey binding functionality

# Future Functionality #

  * More integration with the GUI
  * More integration with the debugging facilities
  * Investigate compilation to native code

# Usage #

After installation, press Ctrl-F10 to launch the interpreter.  Try

```
IDA.msg "Hello, world!\n";;
```

## Loading Code ##

There are four ways to load code into the environment:

  1. Typing directy into the interpreter window.  The interpreter accepts one command at a time, i.e. `let x = 1;;let y = 2;;` The interpreter will accept this, but only process the binding for x.  `#use` directives (see below) come in handy for issuing multiple statements at once.
  1. `#load` directives to load precompiled modules.  You can load precompiled modules like this: `#directory "c:\\pandemic\\Util";;  #load "LowLevel.cmo";; (* Can also specify full paths *)`  This is nice for functionality that you have codified into something that does not change frequently:  simply compile it and `#load` it.  You do not need to copy the .cmo files into the OCaml lib directory to utilize them in this fashion.  If you want to access the contents of these modules in the toplevel, you'll still need to either open the modules i.e. `open LowLevel` or refer to them by module name i.e. `let x = LowLevel.blah bla in ...`.  Similar to when linking, the modules loaded must be loaded in topological order, otherwise loading will raise errors about missing symbols.  If you want to alter a compiled module that has already been loaded, you'll need to close and re-open the database.
  1. `#use` directives to load non-compiled, textual .ml/mli files.  This is nice for files that are in active development.  You can make changes to the .ml file and `#use` it again, thereby shadowing the old bindings.  However, `#use`ing a file simply loads the contents into the current top-level, and therefore does not account for module boundaries.  I.e. `#use "X.ml";; (* ... *) X.whatever (* ... *)` will not work.
  1. Compiling the .cmo files directly into the plugin, so that they are available at all times (but still need to be `open`ed or referred to by module name).  Place the names of the .cmo files into the EXTRAOCAML variable in the makefile and recompile the plugin with `make clean && make all`.  These files must also be installed into the OCaml `lib/` directory.  This option is most appropriate for parts of the IDAOCaml framework itself, or for things bound against C/C++ code.

## Tips and Tricks ##

Files that you `#use` can have `#load` and `#use` directives inside of them.  When I load my framework, I issue a command like `#use "c:\\pandemic\\framework.ml";;`, where framework.ml has a list of `#load` and `#use` directives like the code block described above under `#load`.  This allows me to load the compiled portion of my framework with one simple command.

For people used to compile/recompile/test cycles, `#load` and `#use` is a different and truly rapid style of development.  It's magical to conceive something, code it, and then watch it happen on the screen seconds later.

Generally, the way I approach development with IDAOCaml is to stare at a problem inside of IDA, formulate a theory on how to subdivide the problem, and attempt to solve the sub-parts.  I code these initial efforts into .ml files in my text editor, and then load them using `#use` directives.  If the efforts prove fruitful, the functionality is eventually refactored and codified into a compilable unit, at which point it can then be `#load`ed instead.

# Dependencies #

  1. ocaml-3.12.0 (source distribution):  http://caml.inria.fr/
  1. cygwin (needed for building OCaml, as per OCaml's README.win32):  http://www.cygwin.com/
  1. The Microsoft Visual C++ toolchain for building OCaml and IDAOCaml
  1. flexdll:  http://alain.frisch.fr/flexdll.html
  1. IDA v5.6 for Windows plus its SDK (commercial) (other versions and platforms untested):  http://hex-rays.com/idapro/

# Building IDAOCaml #

  1. Obtain ocaml-3.12.0 and build it from source under cygwin using the MSVC toolchain.  This process is described in README.win32 that comes with the distribution.  At the time of writing, there is no such prebuilt distribution available from the OCaml maintainers.  Nevertheless, if such a distribution becomes available, it will not be suitable to build the plugin.  The plugin relies upon certain libraries from OCaml that are not included with the binary distributions, and are only available as byproducts of the compilation process.
  1. Edit Config.paths in the directory in which you installed IDAOCaml to reflect your IDA SDK directory as well as the directory into which the OCaml 3.12.0 source is installed.
  1. Build under cygwin (in the same configuration you used to make OCaml) with "make all".
  1. If successful, idaocaml.plw will be created in the directory with the source files.  This plugin can then be copied to your IDA plugins directory.

Currently, I build IDAOCaml on Windows with IDA 5.6 (the native GUI, not the QT-based one).  I have not investigated building under any other configurations, including later or earlier versions of IDA, the QT versions, or upon other platforms.  I will do my best to assist in building the plugin upon other systems, but I suspect I will fail somebody.

Binary releases are currently pointless as one needs to have OCaml 3.12.0 built from source.  If the situation changes, I will re-evaluate this decision.

# General Coding Issues #

There are some naming conflicts between IDA's header files and the OCaml C header files.  This motivates my design of putting all OCaml-related C functionality in files by themselves with "ocaml" in the name, IDA-related C++ functionality in files with "c" in the name, and common/bridge functionality in files with "common" in the name.  I have been a bit sloppy and not followed these conventions pervasively.

To add additional C functionality, add the paths to the relevant .o/.lib files to the EXTRACPP variable in the makefile.  Similarly, to add additional OCaml .cmo files to be bound against the toplevel, add these files to the EXTRAOCAML variable in the makefile.  For reasons that I do not fully understand, bound .cmi/.cmo files must still be copied into the OCaml installation's `lib/` directory.

# Porting to Linux/Mac #

There is a single file that relies upon Windows functionality:  IDAGraphViewer.cpp.  This file mimics the ugraph plugin sample that comes with the IDA SDK prior to 6.x, which also relies upon windows.h for HWND.  Comparing ugraph.cpp from the 5.x and 6.0 SDKs, it looks like a simple matter to port to other platforms.

# General Comments #

The code could be cleaner, and I will most likely refactor it and make it more sensible in a future release.  However, my current priority is that the platform works.

# Known Issues #

Sometimes when attempting to launch the interpreter, IDA will complain about an invalid floating point operation and refuse to load it.  I have not debugged this.  The problem seems to go away if you close IDA and re-open it.

There is a "bug" in OCaml itself that causes an error regarding I/O file descriptors to be issued when the compiler encounters a situation in which it would normally emit a warning (e.g. non-exhaustive pattern match).  The issue is currently open; http://caml.inria.fr/mantis/view.php?id=5271 describes the problem and a solution (a one-line patch plus a recompile of OCaml).

I have experienced issues with linkage to foreign functions.  The issue is currently open:  http://caml.inria.fr/mantis/view.php?id=4166 describes the problem, and I give a workaround herein.  Specifically, the problem manifests itself when the .mli file directly specifies an external function.  For instance, from z3.mli that comes with Microsoft's Z3:

```
external mk_int_sort : context -> sort
	= "camlidl_z3_Z3_mk_int_sort"
```

I've found that by changing the declarations in the .mli to `val` declarations rather than `external` ones and removing the binding string, and instead specifying these linkages in the .ml files, things work.  The example above after transformation would be:

```
val mk_int_sort : context -> sort
```

# Thanks #

#ocaml on FreeNode IRC has been invaluable, and particularly the user **thelema**.  I would not even have known where to begin without thelema's help:

```
<rrolles> can anyone point me to a reference regarding embedding an OCaml REPL inside of some other application?  in particular there's a graphical machine-code disassembler for Windows called IDA, and I would like to embed a custom toplevel inside of it
<thelema> rrolles: look at the embedding ocaml in C section in the reference manual and embed the function Toploop.loop from the toplevel code
```