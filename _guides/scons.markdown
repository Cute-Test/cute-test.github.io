---
layout: guide
title: "SCons Build Support"
tutorial_index: 4
active: guides
---

# What is SCons?
<a name="whatisscons"></a>

[SCons](http://scons.org/) is an open source software build tool which tries to fix the numerous weaknesses of make and its derivatives. For example, the missing automatic dependency extraction, *make*'s complex syntax to describe build properties and cross-platform issues when using shell commands and scripts. *SCons* is a self-contained tool which is independent of existing platform utilities. Because it is based on Python, a SCons user has the full power of a programming language to deal with all build related issues.

> It was long past time for autotools to be replaced, and SCons has won the race to become my build system of choice. Unified builds and extensibility with Python — how can you beat that?

Eric S. Raymond, author of *The Cathedral and the Bazaar*

![](/img/scons-buildconsole.png)


## Use SConsolidator to build your projects in Eclipse
<a name="usesconsolidatortobuildyourprojectsineclipse"></a>

Maintaining a *SCons*-based C/C++ project with Eclipse *CDT* meant that all the intelligence *SCons* puts into your project dependencies had to be re-entered into Eclipse *CDT*'s project settings, so that its indexer and parser would be able to know your code's compile settings and enable many of its features. In addition, *SCons*' intelligence comes at the price of relatively long build start-up times - when it (re-) analyses the project dependencies - which can become annoying when you just fix a simple syntax error.

*SConsolidator* addresses these issues and provides tool integration for SCons in Eclipse for a convenient development experience.

## Main Features
<a name="mainfeatures"></a>

* Conversion of existing C++ CDT managed build projects to SCons projects
* Import of existing SCons projects into Eclipse with wizard support
* Interactive mode to quickly build single C/C++ source files speeding up round-trip times
* A special view for a convenient build target management of all workspace projects
* Graph visualization of build dependencies that helps in debugging SCons build issues

## Works out of the box
<a name="worksoutofthebox"></a>
SConsolidator has been successfully used to import the following SCons-based projects into Eclipse:

* MongoDB
* Blender
* FreeNOS
* Doom 3
* [COAST](http://coast-project.org/)

## Contribute
<a name="contribute"></a>
SConsolidator is available at Github.
Found a bug or have a feature request?

Report bugs and feature requests on [Github](https://github.com/IFS-HSR/SConsolidator).

# Installation
<a name="installation"></a>

To use SConsolidator, you first have to install [SCons](http://www.scons.org/) (version 2.0 is the minimum requirement). SConsolidator requires at least a Eclipse Indigo release of the CDT, otherwise you won't be able to install SConsolidator. Execute the following steps to install it in Eclipse:


1. Choose the menu *Help*, *Install New Software*...
1. Type the URL of the SConsolidator update site into the site field: http://www.sconsolidator.com/update
1. Then, you can either only install SConsolidator's base functionality by choosing the feature *SConsolidator* - *Base* or additionally *SConsolidator* - *Dependency Visualization* for visualizing build target dependencies (the latter needs GEF and Zest to be installed).
1. After the installation, Eclipse will ask to restart itself. Please do this.

# Getting Started
<a name="gettingstarted"></a>

## Add SCons support to an existing project

To add SCons support to an existing C/C++ project, just right click on the project(s) in the project explorer and choose one of the following two submenus of SCons:

* *Use self-provided SCons build* (aka "existing mode") for projects where you provide the SCons build files
* *Use generated SCons build* (aka "managed mode") for projects where SConsolidator should create and update the SCons build files

Notice that you should use managed for simple C/C++ projects **only**. It can be considered as a good starting point if you are not yet familiar with SCons. As soon as your projects grow in size and complexity, you want to use your own provided SCons build files.

For non-C/C++ projects, you only have one menu named *Add SCons support*.

## Building projects with SCons

If you request a build, the output of a SCons build run is shown in a project specific console. Compile errors are shown in red and hyperlinks are provided to directly jump to the location in the editor where the error was found by the compiler. Error markers are created for the compiler errors as we are used from Eclipse.

![](/img/scons-buildconsole.png)

## Target View

SCons target view shows all the SCons projects in the current workspace and allows you to maintain the targets on a project level. To request a build for a specific target, you have to select it in the view and click the build button in the upper right corner. The same can be achieved for a build in the interactive console.

![](/img/scons-targetview.png)

There is a dialog to create a new target for a SCons project. There you can define the name of the target and additional SCons options which will be used when SCons is invoked. Furthermore, you can also specify a target alias which is also shown in the target view.

![](/img/scons-newtarget.png)

## Dependency View

The dependency view visualizes the dependencies of the selected target SCons has emitted. You can choose a target of a SCons project in the workspace with a click on the lookup symbol in the upper right corner. The dependencies are then shown in the graph area.

![](/img/scons-dependencyview.png)

## C/C++ specific support

### Create new SCons based C/C++ Managed Project

To create a new SCons based project, you can use the *New Project* wizard of Eclipse and choose *C/C++ project*. There you can see three new project types for SCons managed projects: *executable*, *shared library* and *static library* project.

![](/img/scons-newprojectdialog.png)

SConsolidator also provides a facility to convert existing managed projects to SCons projects with a context menu on the project level (*SCons* -> *Add SCons support*) and with a wizard as shown in this figure.

![](/img/scons-convertdialog.png)

### Import of Existing SCons based C/C++ Project

SConsolidator helps you to import existing SCons based C/C++ projects with a wizard. You can choose the location where the code of the existing project is located. If the location contains an Eclipse project, the name of the project is deduced and shown in the corresponding input field. In case you want to set further options for the initial SCons run, you have the possibility to put them in the field *SCons options*. SConsolidator will then try to gather all include paths and macros of your project necessary for the CDT indexer to properly work and store them in the project settings.

![](/img/scons-importdialog.png)

### Refresh SConsolidator after altering the SCons build

SConsolidator helps you to keep the build settings of your SCons build consistent with your Eclipse CDT project by (one-way) synchronizing the used include paths and macros. Whenever you changed your SCons build in a way that affects the used include paths or macros, click on the icon with the Switzerland flag in the toolbar or use the project context menu *Refresh C/C++ project from SCons* of SConsolidator. Both actions run your SCons build and collect all the necessary information for Eclipse CDT to work as expected.

### Interactive Console

To build single targets without the need to invoke full builds with every change, we provide an interactive console with SConsolidator where you have all the possibilities known from SCons' interactive mode. We allow building and cleaning of the corresponding target(s) for the source file currently loaded in the active editor and a redo of the last action as can be seen from the buttons. Of course, you can also type SCons commands in the interactive console directly.

![](/img/scons-interactiveconsole.png)