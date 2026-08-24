JSim
====

JSim is a Java-based simulation system for building quantitative numeric models and analyzing them with respect to experimental reference data. JSim's primary focus is in physiology and biomedicine, however its computational engine is quite general and applicable to a wide range of scientific domains. JSim models may intermix ODEs, PDEs, implicit equations, integrals, summations, discrete events and procedural code as appropriate. JSim's model compiler can automatically insert conversion factors for compatible physical units as well as detect and reject unit unbalanced equations. JSim also imports and exports model archive formats SBML and CellML.   

Please see http://www.physiome.org/jsim/ for more information.

This repository contains the JSim 2.0 source distribution.  Use of this source code is
governed by the terms of the JSim 2.0 license agreement which may be found
in the file license.html within the distribution root directory (/SRC2.0/).

Information on the contents of this distribution and how to build it may
be found in the on-line JSim documentation set:
 https://github.com/NSR-Physiome/JSim/wiki/Building-JSim-from-source
 or:
 http://www.physiome.org/jsim/docs/Devel_Build.html
  
JSim Binaries for MS Windows, Apple OS, and CentOS EL 6/7 can be found at:
 
 http://www.physiome.org/jsim/download/index.html

Modernization and Physiolog collaboration
-----------------------------------------

This fork is being modernized under a scientific-reproducibility constraint: changes
must preserve numerical behavior, model semantics, solver behavior, physical units,
file formats, and historical verification evidence unless a reviewed compatibility
record demonstrates otherwise. See [MODERNIZATION.md](MODERNIZATION.md) for the
verified build status, current blockers, compatibility risks, and milestone plan.

The modernization also supports [Physiolog](https://physiolog.org), an open clinical
physiology textbook, teaching-simulation, and research project. JSim will be used
alongside HumMod, public Physiome models, browser-based teaching models, and other
suitable simulation engines for two connected purposes:

1. Build a generative-mechanism research pipeline from physiological questions and
   teaching models in Physiolog to explicit, reproducible models for *models4PT*.
2. Develop and cross-validate physiology models for teaching in the Physiolog book
   and simulations, as well as for research.

JSim contributes a general equation-based execution environment, physical-unit
checking, parameter estimation and comparison with experimental reference data, and
CellML/SBML interchange. HumMod contributes an integrative whole-body comparison
environment, public Physiome models contribute reusable and cited model structures,
and Physiolog's browser models contribute small, transparent learning experiences.
No engine is treated as ground truth: model and engine versions, assumptions, units,
scenarios, solver settings, outputs, and disagreements must be recorded as part of
the evidence.


