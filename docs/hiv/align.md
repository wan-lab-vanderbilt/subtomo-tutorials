# Tilt-Series Alignment

Since this data set has gold fiducials, we could use bead tracking for manual tilt series alignment, for example in Etomo.
Manual alignment typically yields superior results to automated alignment but for sake of time we will use TOMOMAN to perform automated tilt-series alignment using AreTomo.
TOMOMAN provides some additional settings on top of the standard AreTomo settings, including the option to use unfiltered or dose-filtered tilt-series, as well as pre-binning the tilt-series prior to alignment.

For more information about AreTomo, see the publication or [manual](https://gensoft.pasteur.fr/docs/AreTomo/1.3.4/AreTomoManual_1.3.0_09292022.pdf).

Open `tomoman_aretomo.param`.

1. The directory parameters block should already be set correctly.

2. The AreTomo files and folders block should already be set correctly. We will generate a bin8 tomogram so `bin8_aretomo/` is a good output directory name.

3. The AreTomo parameters block determines how AreTomo will be run.

    1. Set `InBin` to 4.
    We find that processing the dose-filtered stack with an input binning of 4 yields better results.

    2. `AlignZ` and `VolZ` are measures of thickness along the z axis measured in unbinned pixels.
    `AlignZ` is the thickness of the temporary volume used for alignment.
    This should be approximately the same as the actual thickness of the sample.
    `VolZ` is the thickness of the reconstructed tomogram and should be larger than both `AlignZ` and the specimen thickness.
    In this case, `AlignZ=1400` and `VolZ=1800` should work well.
    After reconstruction, you will be able to see the tomogram extending beyond the specimen in XZ slices.
    The rest of the AreTomo parameters can be left as defaults.

    3. Note that `OutBin` is set to 8.

    4. Use the same `Gpu` setting as for [motion correction](preproc.md#making-motion-corrected-stacks).

4. Run reconstruction as before in the TOMOMAN console.

You now have a reconstructed tomogram you can visualize in 3dmod.
You can run the below command in the standalone, or omit the `!` and run it in a separate terminal.
If it does not work, you file may have a slightly different name.

    !3dmod bin8_aretomo/TS_01_dose-filt_bin8.mrc

In the 3dmod window, select Image > XYZ to view slices along all axes.
