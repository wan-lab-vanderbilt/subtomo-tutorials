# Getting Started

In this practical, we will be going over data preprocessing, tomogram reconstruction, and subtomogram averaging using our TOMOMAN/STOPGAP pipeline.
A typical cryo-electron tomography (cryo-ET) workflow is illustrated in figure 1.

## TOMOMAN

TOMOMAN, i.e. TOMOgram MANager, is a MATLAB package for managing the various preprocessing steps for taking raw data to reconstructed tomograms.
TOMOMAN mainly acts as a set of wrapper scripts for external packages, managing the input and outputs of each external module to form a cohesive pipeline.
Preprocessing metadata is collected into the TOMOMAN “tomolist” (default name `tomolist.mat`) while output is written to a plain-text log file (default name: `tomoman.log`).

## STOPGAP


## External Packages

During this practical we will be using a number of software packages.
These include:

- MotionCor2 >= 1.6.4
- AreTomo
- 3dmod
- CTFFIND4
- [Fourier3D](https://github.com/turonova/Fourier3D)
- [NovaCTF](https://github.com/turonova/novaCTF)
- UCSF Chimera (with [Pick Particle](https://www.biochem.mpg.de/7940000/Pick-Particle) and [Place Object](https://www.biochem.mpg.de/7939908/Place-Object) plugins)

## Running TOMOMAN

?

## Setting up the MATLAB Environment

?
