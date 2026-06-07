# Setup for 2026 U of M Workshop

These instructions are for setting up your system to run this tutorial in the 2026 University of Michigan cryo-ET workshop.

First, use DCV Viewer to connect to your AWS instance using the provided IP address, username, and password.
Right click on the desktop to start a terminal window.

## Load Modules

The instances we are using for this tutorial have all the required software installed as Lmod modules.
Since we will be opening multiple terminals where you will run TOMOMAN or STOPGAP, we will first edit our .bashrc to source the proper modules.

    gedit ~/.bashrc 

Once open, scroll to the bottom and add the following line:
    
    module load aretomo2 chimera ctffind/4.1.14 cuda/12.1.1 fourier3d motioncor3/1.2.4 imod novactf stopgap tomoman

Save the .bashrc in your text editor, then source our new bashrc file:

    source ~/.bashrc

>NOTE: To paste into a terminal, you can use the keyboard shortcut `CTRL + Shift + V`.

You can see what packages are loaded with `module list`.

> NOTE: loading modules makes software accessible as well as defining paths and aliases appropriately to run them.

## Tutorial Data

The test data we will be using for this workshop is tilt series 1 (TS_01) from the EMPIAR-10164 dataset.
The `/work/data/EMPIAR-10164/data/` directory contains `frames/` and `mdoc-files/` subfolders.
The `frames/` subfolder contains the raw .mrc frames for the tilt series, while the `mdoc-files/` folder contains the SerialEM .mdoc files for the tilt series.
With a larger dataset there would be one mdoc file for each tilt series.
Here, we will generate symbolic links for the files relevant to TS_01.

    mkdir ~/HIV_dataset/
    cd ~/HIV_dataset/
    mkdir tomo/
    cd tomo/
    ln -s /work/data/EMPIAR-10164/data/frames/ frames
    ln -s /work/data/EMPIAR-10164/data/mdoc-files/ rawdata

You are now ready to begin the tutorial.
