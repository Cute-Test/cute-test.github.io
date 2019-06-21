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


# Use SConsolidator to build your projects in Eclipse
<a name="usesconsolidatortobuildyourprojectsineclipse"></a>

Maintaining a *SCons*-based C/C++ project with Eclipse *CDT* meant that all the intelligence *SCons* puts into your project dependencies had to be re-entered into Eclipse *CDT*'s project settings, so that its indexer and parser would be able to know your code's compile settings and enable many of its features. In addition, *SCons*' intelligence comes at the price of relatively long build start-up times - when it (re-) analyses the project dependencies - which can become annoying when you just fix a simple syntax error.

*SConsolidator* addresses these issues and provides tool integration for SCons in Eclipse for a convenient development experience.

# Main Features
<a name="mainfeatures"></a>

* Conversion of existing C++ CDT managed build projects to SCons projects
* Import of existing SCons projects into Eclipse with wizard support
* Interactive mode to quickly build single C/C++ source files speeding up round-trip times
* A special view for a convenient build target management of all workspace projects
* Graph visualization of build dependencies that helps in debugging SCons build issues

# Works out of the box
<a name="worksoutofthebox"></a>
SConsolidator has been successfully used to import the following SCons-based projects into Eclipse:

* MongoDB
* Blender
* FreeNOS
* Doom 3
* [COAST](http://coast-project.org/)

# Contribute
<a name="contribute"></a>
SConsolidator is available at Github.
Found a bug or have a feature request?

Report bugs and feature requests on [Github](https://github.com/IFS-HSR/SConsolidator).

# Try it out now!
<a name="tryitoutnow"></a>

Follow the instructions in [Installation](installation) and give it a try!

# Further information
<a name="furtherinformation"></a>

More information on SConsolidator can be found in the [Getting Started](getting-started).