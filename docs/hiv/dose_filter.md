# Dose Filtering Stacks

We find that dose filtering (a.k.a. exposure filtering) greatly improves the contrast, alignment quality, and high-resolution signals of reconstructed tomograms.
TOMOMAN has its own function for doing this based off the empirical values determined by Grant and Grigorieff (doi: [10.7554/eLife.06980](https://doi.org/10.7554/eLife.06980)).

## Perform Dose Filtering
Open `tomoman_dosefilter.param`.

1. The tomolist parameter block should already be set correctly.

2. The dose filtering parameter block is usually fine with the default values.

    1. Pre-exposure is typically from tasks such as mapping and realigning during data collection, but is generally quite low and can be ignored.
    TOMOMAN lets you dose filter frames (i.e., not just motion corrected images), if you first generate an aligned frame stack.
    This can better preserve high resolution signals, but for the practical we will just filter the aligned images.
    Critical exposure constants can be provided, but these are typically not known *a priori*, so we generally use the default values determined by Grant and Grigorieff.

    2. We are not using odd and even stacks so you can set `check_oddeven` to `false` to avoid warnings about missing stacks.

3. Run dose filtering in the TOMOMAN console.

        tomoman(pwd,'tomoman_dosefilter.param');

4. If you would like to see the results of dose filtering, you can open the unfiltered and filtered stacks in 3dmod.
   Remember to open this in your other terminal; you can open it with:

        3dmod TS_01/TS_01.st TS_01/TS_01_dose-filt.st