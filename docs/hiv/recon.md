# Tomogram Reconstruction using NovaCTF

Final tomogram reconstruction for subtomogram averaging will be performed using novaCTF, which allows for 3D CTF-correction during reconstruction.
TOMOMAN will generate the appropriate scripts and output directories for running novaCTF and binning tomograms by Fourier cropping using Fourier3D.

Open `tomoman_novactf.param`.

1. The directory parameters should already be correct.

1. The parallelization parameters determine how jobs are split between cores.
Set `n_cores` to 30.

1. Stack parameters are parameters for generating the aligned stacks prior to tomogram reconstruction.

    - `ali_dim` allows for resizing, though I recommend using the full image size: `3712,3712`.

    - `erase_radius` is for gold fiducial erasing. Since we used autmoated tilt-series alignment, we don't have a model for the gold fiducials. Set this to `none`.

    - `taper_pixels` is used to taper the edges of the rotated images when generating an aligned stack; `100` is usually sufficient.

    - `ali_stack_bin` is used to bin the image stack before reconstruction.
      In general, we recommend setting this to `1` and performing serial binning.

1. The 3D CTF correction parameters set how novaCTF will perform 3D CTF correction.
We always recommend using the dose-filtered stack (`process_stack = dose-filtered`) and correcting CTF using phase flipping (`correction_type = phaseflip`).
For `defocus_step`, smaller steps produce more precise results at the cost of more computation time (see [the novaCTF publication](../reading.md#methods) for more information).
For this tutorial, set it to `50`.

1. Tomogram reconstruction parameters have some specifics on how to perform the reconstruction.
We typically skip radial filtering.
The `tomo_bin` parameter allows you to set the final binning factors desired.
For this tutorial, set binnings of 1, 2, 4, and 8 with `tomo_bin = 1,2,4,8`.

    > NOTE: The minimum allowed value for binning is equal to the `ali_stack_bin`. E.g., if `ali_stack_bin` is set to 4, the minimum allowed value here is 4.

1. The `output_dir_prefix` sets the name of the tomogram output directories, which will be placed within the `root_dir`.
For instance, bin 4 tomograms will be placed in: `[root_dir]/[output_dir_prefix]_bin4/`.
For this tutorial, set it this to `novactf_`.

1. The additional parameters include the `recons_list`, which allows for reconstructing a subset of tomograms.
You can leave it as none to reconstruct all non-skipped tomograms in the tomolist.

1. Fourier3D is a program for Fourier cropping volumes written by Beata Turoňová.
The `f3d_memlimit` parameter sets a limit to how much memory Fourier3D can use; more memory allows for faster computation times.
For this tutorial, set `f3d_memlimit` to `60000`.

1. NovaCTF’s approach to CTF-correction assumes that the center of mass is at the center of the tomograms.
If this is off, the reconstructed tomogram will contain a systematic error in all planes.
To refine the tomogram center, novaCTF allows you to generate an offset value for recentering.
TOMOMAN can take an input STOPGAP motivelist, and use the center of mass of the particles from each tomogram as the refined center.
Since we have no such motivelist now, this can be left off.

1. Run novaCTF in the TOMOMAN console.
       tomoman(pwd,'tomoman_novactf.param');
   This is typically the single longest computation step in the workflow.
   We suggest starting this job to ensure that it is running, but precompued tomograms are provided.
   For now, you can copy the precomputed 8x and 4x binned tomogram to your working directory.

       !rsync -a /data/EMPIAR-10184/precomputed/novactf_bin8 .
       !rsync -a /data/EMPIAR-10184/precomputed/novactf_bin4 .

    > NOTE: Make sure to leave off the last `/` to copy the directory and its contents.

1. After running novaCTF or copying the precomputed tomograms, you can preview your them in 3dmod, for example:

        !3dmod novactf_bin8/TS_01_dose-filt.rec

    > NOTE: The unbinned tomogram is VERY large (>90GB). As such, we recommend you only look at the lower binned tomograms.

Preprocessing in TOMOMAN is now finished!
