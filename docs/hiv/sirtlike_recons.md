# Reconstuct Tomogram with a SIRT-like Filter

While we will use 3D-CTF corrected tomograms for subtomogram averaging, such tomograms often have very low contrast. As such, it can be useful to reconstruct filtered tomograms with higher contrast for particle picking. For this, we will use IMOD to reconstruct un-CTF-corrected tomograms with a SIRT-like filter for particle picking. 

## Tomogram Reconstruction using IMOD and a SIRT-like Filter

Open `tomoman_imod_reconstruct.param`.

1. The directory parameters should already be correct.

2. The parallelization parameters determine how jobs are split between cores. The default of `n_cores` of `10` is sufficient.

3. Stack parameters are parameters for generating the aligned stacks prior to tomogram reconstruction.

    -`process_stack` denotes which unaligned stack to use. To use the dose-filtered stack, set to `dose-filtered`

    - `erase_radius` is for gold fiducial erasing. Since we used autmoated tilt-series alignment, we don't have a model for the gold fiducials. Set this to `none`.

    - `ali_stack_bin` is used to bin the image stack before reconstruction.
    Since we will be picking and starting averaging with binned data, for this tutorial set it to `8`.

4. IMOD can do 2D-CTF correction using ctfphaseflip. Since we are just using this tomogram for picking, we can disable this by setting `ctfphaseflip` to `false`.

5. For the Tomogram Reconstruction Parameters, the main thing here for us is the `fakesirtiter`, which sets the number of iterations for IMOD's SIRT-like filter. Setting this to `15` will provide a good tradeoff between contrast and sharpness. 

The `radial` parameter can be left as `none`, and the `tomo_bin` should be set to `8` to output a 8x binned tomogram. 

We can leave the `output_dir_prefix` as `uncorrected_sirt15_`; the output tomogram will be in a folder called uncorrected_sirt15_bin8.

6. For the additional parameters, make sure that `gpu_id` is set to `0`.

7. Fourier3D is a program for Fourier cropping volumes written by Beata Turoňová.
The `f3d_memlimit` parameter sets a limit to how much memory Fourier3D can use; more memory allows for faster computation times.Since we will just be generating one tomogram binning, this will be ignored.

8. Save the prameter file and run IMOD reconstruction in the TOMOMAN console.

        tomoman(pwd,'tomoman_imod_reconstruct.param');


9. After reconstruction, you can preview your them in 3dmod, for example:

        3dmod uncorrected_sirt15_bin8/TS_01_dose-filt.rec

> NOTE: if you want to compare the different tomograms, you can open them at the same time. For example:
        3dmod aretomo_bin8/TS_01_dose-filt.rec uncorrected_sirt15_bin8/TS_01_dose-filt.rec
> You can flip between the two tomograms using the 1 and 2 buttons.
