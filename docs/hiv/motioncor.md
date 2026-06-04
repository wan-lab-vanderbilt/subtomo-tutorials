# Aligning Frames and Making Stacks

The tilt series directory has been set up, but before we can examine it, we must first perform motion correction on the frames and assemble the motion corrected images into a stack.
For this dataset, we will be using MotionCor3.

## Running MotionCor3

Open the `tomoman_motioncor3.param` file in a text editor and review its parameters.

1. The tomolist parameters should already be correctly set.

2. The TOMOMAN parameters set parameters for how TOMOMAN runs.

    1. Most tasks have some "force" parameter, which tells TOMOMAN to repeat the task on tilt series that have already been processed.
    If set off, TOMOMAN only runs the task on tilt series that have not yet been processed.
    For this tutorial, leave `force_realign` as 0.

    2. The `image_size` parameter sets the output image size.
    This dataset was collected on a K2 in super-resolution mode but we want a normal resolution output image.
    Set `image_size` to `3712,3712`; this pads a K2 image by 2 pixels in one axis, but results in an image size amenable to binning at factors of 2.

3. The MotionCor3 parameters block contains MotionCor3’s parameters.
For this dataset, most of the defaults are fine. However, because the data was collected in super-resolution, set the `FtBin` parameter to 2 so that the aligned frames are Fourier cropped back to normal size and set the `Gpu` to `0`.

4. Since this dataset has .mrc frames, the “EER specific part” block can be ignored.

5. We will not be doing noise2noise training, so odd/even stacks are not needed.

6. Save the param file and run the TOMOMAN in the standalone with the new `paramfilename`:

        tomoman(pwd, 'tomoman_motioncor3.param');