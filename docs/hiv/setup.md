# Setup for 2024 U of M Workshop

These instructions are for setting up your system to run this tutorial in the 2024 University of Michigan cryo-EM workshop.

First, use DCV Viewer to connect to your AWS instance using the provided IP address, username, and password.
Right click on the desktop to start a terminal window.

## Load Modules

The instaces we are using for this tutorial have all the required software installed as Lmod modules.
Install them with:

    module load imod chimera motioncor2 aretomo ctffind/4.1.14 novactf fourier3d tomoman stopgap/0.7.4

You can see what packages are loaded with `module list`.

> NOTE: loading modules makes software accessible as well as defining paths and aliases appropriately to run them.

## Tutorial Data

The test data we will be using for this workshop is tilt series 1 (TS_01) from the EMPIAR-10164 dataset.
The `/data/EMPIAR-10164/data/` directory contains `frames/` and `mdoc-files/` subfolders.
The `frames/` subfolder contains the raw .mrc frames for the tilt series, while the `mdoc-files/` folder contains the SerialEM .mdoc files for the tilt series.
With a larger dataset there would be one mdoc file for each tilt series.
Make the appropriate folders in your working directory and copy the files relevant to TS_01.

    mkdir ~/HIV_dataset/
    cd ~/HIV_dataset/
    mkdir tomo/
    cd tomo/
    mkdir frames/
    mkdir rawdata/
    rsync -a /data/EMPIAR-10164/data/frames/ frames/
    rsync -a /data/EMPIAR-10164/data/mdoc-files/ rawdata/

You are now ready to begin the tutorial.