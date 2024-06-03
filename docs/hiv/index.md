# HIV Dataset Tutorial

In this tutorial, we will work on the HIV immature capsid virus-like particles (VLPs) dataset found in [EMPIAR-10164](https://www.ebi.ac.uk/empiar/EMPIAR-10164/). The goals for this tutorial include:

- Setting up a TOMOMAN project directory
- Running TOMOMAN jobs locally
- Performing preprocessing using TOMOMAN
- Automated tilt series alignment in AreTomo
- 3D-CTF corrected tomogram reconstruction with NovaCTF
- Manually picking spherical particles in Chimera using the Pick Particle tool
- Generating oversampled initial particle positions on spherical surfaces
- Setting up a STOPGAP project directory
- Running STOPGAP jobs locally
- Extracting subtomograms using STOPGAP
- Subtomogram alignment and averaging in STOPGAP
- Visualizing particle positions in Chimera using the Place Objects tool
- Generating a *de novo* structure from a single VLP at 8x binning
- Expanding the dataset to all VLPs in the 8x-binned tomogram
- Rescaling the motivelists for processing 4x-binned data

Be sure to set up your environment properly before beginning.

## External Packages

During this tutorial we will be using a number of other software packages.
These include:

- MotionCor2 ([publication](https://doi.org/10.1038/nmeth.4193))
- AreTomo ([publication](https://doi.org/10.1016/j.yjsbx.2022.100068))
- IMOD ([publication](https://doi.org/10.1006/jsbi.1996.0013))
- CTFFIND4 ([publication](https://doi.org/10.1016/j.jsb.2015.08.008))
- Fourier3D ([code](https://github.com/turonova/Fourier3D))
- NovaCTF ([publication](https://doi.org/10.1016/j.jsb.2017.07.007), [code](https://github.com/turonova/novaCTF))
- UCSF Chimera ([homepage](https://www.rbvi.ucsf.edu/chimera/)) with [Pick Particle](https://www.biochem.mpg.de/7940000/Pick-Particle) and [Place Object](https://www.biochem.mpg.de/7939908/Place-Object) plugins
