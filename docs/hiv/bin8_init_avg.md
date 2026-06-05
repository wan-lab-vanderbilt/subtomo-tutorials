# Generating an Initial Average

In this step, we will generate an initial average using our extracted subtomograms and the initial orientations we calculated from our manual picking.

Before we can do this, we will first need to prepare a real-space mask using the STOPGAP toolbox. 

After this, we will use the `stopgap_subtomo_parser` to generate a parameter file for averaging.


## Launching the STOPGAP Toolbox

We are now ready to load and use the STOPGAP Toolbox.
Its purpose is the same as the TOMOMAN standalone, to run MATLAB commands without needing a MATLAB license and installation.

Exit the TOMOMAN standalone with `exit`.
Use `cd` to change to the `init_ref/` directory:

    cd ../subtomo/init_ref/

Start the STOPGAP Toolbox by running:

    stopgap_toolbox.sh


## Preparing an alignment mask

Subtomogram averaging in STOPGAP always involves calculating a Fourier Shell Correlation (FSC) in order to output two halfmaps and a figure-of-merit weighted average.
Our motivelist doesn’t currently have A/B halfsets defined, so halfmaps will be randomly generated.

For FSC calculation, an alignment mask is always required.
Since we don’t know the reference structure, we can simply provide a basic sphere with a Gaussian dropoff (always include a soft edge on your alignment masks).
To save a sphere mask into the `mask/` folder, change into your `init_ref/` directory, run this in the STOPGAP Toolbox:

        sphere = sg_sphere(32, 10, 3);
        sg_mrcwrite('masks/sphere.mrc', sphere);

Check the mask using 3dmod.

        3dmod masks/sphere.mrc

What you want is a soft-edged mask that tapers to 0 before hitting the box edges.




## Calculate Starting Reference

"Reference-free" refers to the fact that we are not using an *external* reference.
Since a reference is always required for iterative alignment, we generate an starting reference by averaging the extracted subtomograms.
In this example, since we have picked our positions using geometry, we have rough starting angles and our initial reference will not be a sphere but instead have rough features.



1. Open `stopgap_subtomo_parser.sh` in a text editor.

    1. Update the `rootdir` as before.

    1. The main settings for this job are in the Job Parameters block.
    Since we are just averaging a single reference, set `subtomo_mode` to `'avg_singleref'`.
    Because we are on iteration 1, set `startidx` to 1.
    For averaging jobs, `iterations` is ignored.
    Set `binning` to `8`.

    1. Update Main File Options to indicate the correct files.

        - `motl_name` sets the rootname of the motivelist without an iteration number or extension, that is `'allmotl_tomo1_obj1'`.

        - `ref_name` sets the prefix for the references produced by averaging and may be chosen at your discretion, `'ref'` is standard.

        - `mask_name` should be the `'sphere.mrc'` that you just saved.

        - `ccmask_name` is ignored for averaging jobs.

    1. Also, ensure that `symmetry` is set to `'C1'` in the bottom "Other inputs" block.
    We won't use symmetry until later in the averaging.

1. Run the subtomo parser.

        ./stopgap_subtomo_parser.sh

1. Update `paramfilename` in `run_stopgap.sh` to the new param file.

1. Run STOPGAP to generate the average.

STOPGAP alignment and averaging runs always output 3 references, named `[ref_name]_[iteration].mrc`, `[ref_name]_A_[iteration].mrc`, and `[ref_name]_B_[iteration].mrc`.
A and B are raw halfsets; these are often noisy as they are not figure-of-merit weighted.
The reference without a halfset designation is a figure-of-merit weighted average of A and B; this is NOT a fully processed reference and is supplied as a quick check of your job.

>NOTE: Before structural interpretation, halfsets should be figure-of-merit weighted, low pass filtered to the estimated resolution, and B-factor sharpened.
    This can be done with STOPGAP using the `sg_calculate_FSC` function.

Preview the three `.mrc` files in the `ref/` folder in 3dmod.
You may wish to view them along all three axes by selecting Image > XYZ (or Ctrl+x) or as an isosurface by selecting Image > Isosurface (or Shift+u).
You may close 3dmod windows before continuing.