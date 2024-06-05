# Tilt-Series Preprocessing

## Launching the TOMOMAN Standalone Executable

Rather than using MATLAB in this tutorial, we will be using the TOMOMAN standalone executable.
This is a compiled executable that provides all the functionalities of running TOMOMAN in MATLAB without the need for a license.

Open a dedicated terminal window to run the TOMOMAN standalone executable.
Ensure you are in your `tomo/` directory.

Start TOMOMAN by running:

        $TOMOMANHOME/bin/tomoman_standalone.sh

This will return a console window in which we will run various TOMOMAN tasks and functions.
It may take a minute to start up the first time you run it.

> NOTE: If you want to run a system command (i.e. a terminal command not a MATLAB command) in the standalone, prepend an `!`. For example, `!3dmod`.

## Initializing TOMOMAN Parameter Files

In this step, we will copy a set of TOMOMAN parameter files to the tomogram folder.
These parameter files are plain-text files that tell TOMOMAN which processing step you wish to run and the parameters for that step.

Copy empty parameter files for all TOMOMAN tasks to the current directory, using default TOMOMAN filenames, by running the below command in the TOMOMAN console.

        tomoman_copy_paramfiles(pwd);

Parameter files for all TOMOMAN tasks are now copied into the `root_dir`, in this case, `tomo/`.
You can view the contents of `tomo/` by running `ls`.
While files for all tasks are copied, they won’t all be used for this tutorial, and you may delete unused ones.

## Importing New Stacks

The first step in TOMOMAN processing is sorting new data into directories, one for each tilt series.
During this step, TOMOMAN scans a `raw_stack_dir` for .mdoc files.
For each one it finds, it generates a new folder to contain all the preprocessing data, links the .mdoc file, creates a `frames/` subdirectory, parses frame names from the .mdoc file, and generates links for all frames from the `raw_frame_dir` to the `frames/` subdirectory.

New data is imported using the `tomoman_import.param` file.
Open this file in any text editor, for example, you can use gedit in a new terminal window:

    gedit tomoman_import.param

Parameters specified here will determine how TOMOMAN imports and sorts new data.
Types of parameters are typically broken into comment blocks.

1. The directory parameters block contains information about working directories
The `root_dir` should already be set from copying.
For this dataset, the `raw_stack_dir` and `raw_frame_dir` should be `rawdata/` and `frames/`, respectively.

2. The tomolist block contains filenames for TOMOMAN’s output files.
This should already be set during copying.

3. The filename parameters are for the raw stacks generated during data collection.
For this dataset, we don’t have them so this can be set to none.

4. The data collection parameters block contains information specific to the parameters used for data collection and the setup of the microscope.
This dataset contains gain-normalized .mrc files, so set the `gainref` parameter to `none`.
All other defaults are fine.

5. The override .mdoc values block allows users to override fields that are normally parsed from the .mdoc file.
This may be important when certain settings aren’t properly calibrated in SerialEM.
For this dataset, set the `tilt_axis_angle` to 85.3, the `dose_rate` to 8, and the `pixelsize` to 1.35.
`target_defocus` will be parsed from the .mdoc file.
When batch acquiring tilt series, it is typically the best choice to set `target_defocus` during data collection and subsequently parse it from the .mdoc during processing.

6. The final block is the sorting parameters, which allows you to ignore certain missing files.
Here raw stacks refer to tilt series image stacks generated during data collection; these are typically just non-motion corrected summed frame stacks, so they can be safely ignored.
TOMOMAN also allows you to ignore missing frames, though this is not recommended.

7. After setting the parameters, save the file.
To run the import task, run this command in the TOMOMAN standalone:

        tomoman(pwd, 'tomoman_import.param');

    The `tomoman` MATLAB command takes two inputs: a `root_dir` to search for a param file and the `paramfilename` to run.
    Since we started the standalone from `tomo/` we can use our working directory (`pwd`) along with the import parameters filename.

    >NOTE: You may get a warning about a missing `TS_01.mrc` file due to files referenced in the .mdoc but not copied for this tutorial.
    As long as importing still completes this can be disregarded.

The tilt series folder should now be properly set.

>NOTE: the import task can be repeatedly run and only new data will be imported and sorted.
This can be useful if you wish to process data during your data acquisition.

## Making Motion-Corrected Stacks

The tilt series directory has been set up, but before we can examine it, we must first perform motion correction on the frames and assemble the motion corrected images into a stack.
For this dataset, we will be using MotionCor2.

Open the `tomoman_motioncor2.param` file and review its parameters.

1. The tomolist parameters should already be correctly set.

2. The TOMOMAN parameters set parameters for how TOMOMAN runs.

    1. Most tasks have some "force" parameter, which tells TOMOMAN to repeat the task on tilt series that have already been processed.
    If set off, TOMOMAN only runs the task on tilt series that have not yet been processed.
    For this tutorial, leave `force_realign` as 0.

    2. This dataset was collected on a K2 in super-resolution mode.
    The image_size parameter sets the output image size; in this case we want a normal resolution output image.
    Set image_size to `3712,3712`; this pads a K2 image by 2 pixels in one axis, but results in an image size amenable to binning at factors of 2.

3. The MotionCor2 parameters block contains MotionCor2’s parameters.
For this dataset, most of the defaults are fine, but be sure to set the `FtBin` parameter to 2 so that the aligned frames are Fourier cropped back to normal size.
Set the `Gpu` to 0.

4. Since this dataset has .mrc frames, the “EER specific part” block can be ignored.

5. We will not be doing noise2noise training, so odd/even stacks are not needed.

6. Save the param file and run the TOMOMAN in the standalone with the new `paramfilename`:

        tomoman(pwd, 'tomoman_motioncor2.param');

## Cleaning Stacks

After generating your motion-corrected tilt series stacks, you now have something you can reasonably look at to assess the quality of the images.
Bad images can include images where something blocks the field of view, such as a grid bar or ice crystal, or bad imaging conditions such as sample charging.
The best way to assess this is by manually looking at the images.
TOMOMAN facilitates the process using the clean_stacks task.
Open `tomoman_clean_stacks.param`.

1. The tomolist parameters block should be set up already.

2. For the stack cleaning parameters block, `clean_binning` refers to the binning factor for displaying the tilt series, `clean_append` is whether to save the cleaned stack with an appended name, and `force_cleaning` forces repeating of stacks that have already been cleaned.
The default parameters should work fine.

3. Save the param file and run stack cleaning.

        tomoman(pwd, 'tomoman_clean_stacks.param');

4. Follow the instructions in the console to remove bad images.
For this dataset, there are at least 5-6 images that are black or too dark to see the specimen well.

    >NOTE: Remember to close your 3dmod windows; TOMOMAN is unable to close spawned windows from external packages.

## Dose Filtering

We find that dose filtering (a.k.a. exposure filtering) greatly improves the contrast, alignment quality, and high-resolution signals of reconstructed tomograms.
TOMOMAN has its own function for doing this based off the empirical values determined by Grant and Grigorieff (doi: [10.7554/eLife.06980](https://doi.org/10.7554/eLife.06980)).
Open `tomoman_dosefilter.param`.

1. The tomolist parameter block should already be set correctly.

2. The dose filtering parameter block is usually fine with the default values.

    1. Pre-exposure is typically from tasks such as mapping and realigning during data collection, but is generally quite low and can be ignored.
    TOMOMAN lets you dose filter frames (i.e., not just motion corrected images), if you first generate an aligned frame stack.
    This can better preserve high resolution signals, but for the practical we will just filter the aligned images.
    Critical exposure constants can be provided, but these are typically not known *a priori*, so we generally use the default values determined by Grant and Grigorieff.

    2. We are not using odd and even stacks so you can set `check_oddeven` to `false` to avoid warnings about missing stacks.

3. Run dose filtering in the standalone console.

4. If you would like to see the results of dose filtering, you can open "before" and "after" stacks in 3dmod.
