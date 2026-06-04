# Initializing a TOMOMAN Project and Importing Data

## Launching the TOMOMAN Standalone Executable

Rather than using MATLAB in this tutorial, we will be using the TOMOMAN standalone executable.
This is a compiled executable that provides all the functionalities of running TOMOMAN in MATLAB without the need for a license.

Open a dedicated terminal window to run the TOMOMAN standalone executable.
Ensure you are in your `tomo/` directory.

Start TOMOMAN by running:

    tomoman_standalone.sh

This will return a console window in which we will run various TOMOMAN tasks and functions.
It may take a minute to start up the first time you run it.

> NOTE: If you want to run a system command (i.e. a terminal command not a MATLAB command) in the standalone, prepend an `!`.

## Initializing TOMOMAN Parameter Files

In this step, we will copy a set of TOMOMAN parameter files to the tomogram folder.
These parameter files are plain-text files that tell TOMOMAN which processing step you wish to run and the parameters for that step.

Copy empty parameter files for all TOMOMAN tasks to the current directory, using default TOMOMAN filenames, by running the below command in the TOMOMAN console.

    tomoman_copy_paramfiles(pwd);

Parameter files for all TOMOMAN tasks are now copied into the `root_dir`, in this case, `tomo/`.
You can view the contents of `tomo/` by running `ls`.
While files for all tasks are copied, they won’t all be used for this tutorial, and you may delete unused ones.

For the rest of the preprocessing tasks, we will be editing parameter files and running them in the TOMOMAN console. To facilitate this, we suggest you now open a new terminal window and go to the `tomo` folder.

    cd ~/HIV_dataset/tomo/

## Importing New Stacks

The first step in TOMOMAN processing is sorting new data into directories, one for each tilt series.
During this step, TOMOMAN scans a `raw_stack_dir` for .mdoc files.
For each one it finds, it generates a new folder to contain all the preprocessing data, links the .mdoc file, creates a `frames/` subdirectory, parses frame names from the .mdoc file, and generates links for all frames from the `raw_frame_dir` to the `frames/` subdirectory.

New data is imported using the `tomoman_import.param` file.
Open this file in any text editor, for example, you can use gedit:

    gedit tomoman_import.param

>NOTE: Most scripts and parameter files in this tutorial are formatted with very long lines.
>As such, they are typically best viewed without text wrapping.
>To disable text wrapping in gedit, open the preferences and untick the "Enable text wrapping" option.

Parameters specified here will determine how TOMOMAN imports and sorts new data.
Types of parameters are typically broken into comment blocks.

1. The directory parameters block contains information about working directories
The `root_dir` should already be set from copying.
For this tutorial, the `raw_stack_dir` and `raw_frame_dir` are `rawdata/` and `frames/`, respectively.

2. The tomolist block contains filenames for TOMOMAN’s output files.
This should already be set during copying.

3. The filename parameters are to provide some inputs on the file naming conventions.

    `raw_stack_ext` defines the extension for the raw tilt series file generated during data collection. For this dataset, we don’t have them so this can be left as `none`.

    For tilt-series with a numerical naming convention, e.g. "Position_1", the `prefix` parameter defines the prefix before the number and stores the number as a `tomo_num` in the tomolist. For this dataset, set `prefix` to `TS_`.

4. The data collection parameters block contains information specific to the parameters used for data collection and the setup of the microscope.
This dataset contains gain-normalized .mrc files, so set the `gainref` parameter to `none`.
All other defaults are fine.

5. The override .mdoc values block allows users to override fields that are normally parsed from the .mdoc file.
This may be important when certain settings aren’t properly calibrated in SerialEM.
For this dataset, set the `tilt_axis_angle` to `85.3`, the `dose_rate` to `8`, and the `pixelsize` to `1.35`.
`target_defocus` will be parsed from the .mdoc file.
When batch acquiring tilt series, it is typically the best choice to set `target_defocus` during data collection and subsequently parse it from the .mdoc during processing.

6. The final block is the sorting parameters, which allows you to ignore certain missing files.
Here raw stacks refer to tilt series image stacks generated during data collection; these are typically just non-motion corrected summed frame stacks, so they can be safely ignored.
TOMOMAN also allows you to ignore missing frames, though this is not recommended.

7. After setting the parameters, save the file.
   To run the import task, run this command in the TOMOMAN standalone:

        tomoman(pwd, 'tomoman_import.param');

    The `tomoman` command takes two inputs: a `root_dir` to search for a param file and the `paramfilename` to run.
    Since we started the standalone from `tomo/` we can use our working directory (obtained using the `pwd` command) along with the import parameters filename.

    >NOTE: You may get a warning about a missing `TS_01.mrc` file due to files referenced in the .mdoc but not copied for this tutorial.
    As long as importing still completes this can be disregarded.

The tilt series folder should now be properly set.

>NOTE: the import task can be repeatedly run and only new data will be imported and sorted.
This can be useful if you wish to process data during your data acquisition.