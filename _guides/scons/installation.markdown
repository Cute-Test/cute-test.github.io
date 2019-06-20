---
layout: guide
title: "SCons Build Support"
---

# Installation
<a name="installation"></a>

To use SConsolidator, you first have to install [SCons](http://www.scons.org/) (version 2.0 is the minimum requirement). SConsolidator requires at least a Eclipse Indigo release of the CDT, otherwise you won't be able to install SConsolidator. Execute the following steps to install it in Eclipse:


1. Choose the menu *Help*, *Install New Software*...
1. Type the URL of the SConsolidator update site into the site field: http://www.sconsolidator.com/update
1. Then, you can either only install SConsolidator's base functionality by choosing the feature *SConsolidator* - *Base* or additionally *SConsolidator* - *Dependency Visualization* for visualizing build target dependencies (the latter needs GEF and Zest to be installed).
1. After the installation, Eclipse will ask to restart itself. Please do this.

[Getting Started](../getting-started) contains further explanations on how SConsolidator is used.