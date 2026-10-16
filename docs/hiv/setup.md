# Setup for 2026 Academia Sinica Workshop

These instructions are for setting up your system to run this tutorial in the 2026 Academia Sinica cryo-ET workshop.

First, log into the DiCOS Apps and start a session with the cryoET Tool. After the session is started, you can open it to begin.

## Load Modules

The instances we are using for this tutorial have the software installed but not directly on path. To source the necessary parameters to run TOMOMAN and STOPGAP, you can source the following file:

    source /opt/scripts/source_stopgap.sh


>NOTE: To paste into a terminal, you can use the keyboard shortcut `CTRL + Shift + V`.

Keep in mind that this will need to be sourced again whenever you open a new terminal.


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
    mkdir frames/
    mkdir rawdata/
    ln -s /activeEM/cryoet_workshop/EMPIAR-10164/frames/TS_01_*.mrc frames/
    ln -s /activeEM/cryoet_workshop/EMPIAR-10164/mdoc-files/TS_01.mrc.mdoc rawdata/

You are now ready to begin the tutorial.
