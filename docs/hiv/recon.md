# Tomogram Reconstruction

## CTF Estimation

Contrast Transfer Function (CTF) estimation is similar for tomography and single particle analysis in that Thon ring fitting is used in both.
However, tilted images have a defocus gradient because different parts of the specimen are different distances from the lens and, as a result, different parts of the images have different Thon rings.
TOMOMAN includes `tiltctf`, an algorithm we developed to use tilt series alignment parameters to generate power spectra that account for this defocus gradient.
We then give this power spectrum to CTFFIND4 for Thon ring fitting.

Open the `tomoman_tiltctf.param` file.

1. The directory parameters should already be correct.

2. The tiltctf parameters include the parameters for calculating power spectra.
Except where noted, default values are fine.

    1. In general, a `ps_size` of 512 and a `def_tol`, defocus tolerance, of 0.05 um is sufficient.
    Defocus tolerance is the maximum allowed tolerance when deciding how to tile tilted images, but `tiltctf` also always uses a minimum tile overlap of ½ the power spectrum size, so increasing this number may not directly affect computation time.

    2. Fourier scaling, `fscaling`, should be used if the data is collected for high-resolution work (e.g. pixel size smaller than ~2 Å/pix).
    This helps with potential aliasing in the power spectrum.
    For this dataset, use `fscaling = 2`.
    This Fourier scaling means the Nyquist frequency in the power spectra will be 5.4 Å.
    In practice, this is not a problem as tilt series data is collected with such low dose per image that Thon rings cannot be fit to very high resolutions.

    3. Defocus range, `def_range`, is a TOMOMAN parameter that defines the search range that CTFFIND4 will use.
    TOMOMAN uses the defocus value stored in the tomolist and provides +/- the defocus range to CTFFIND4.
    If you know your target defocus was incorrect during data acquisition, you can increase this range.
    If not, the default is fine.

    4. Handedness describes which side of the image (left or right) has a greater defocus at positive tilt angles.
    For this dataset, the right side has greater defocus at positive tilt so set `handedness` to `+1`.

3. The CTFFIND4 parameter block contains the same parameters for CTFFIND4.

    1. In general, the defaults work, though you may wish to increase the `min_res` and reduce the `max_res` for tomography.

    2. The `nthreads` parameter sets parallelization for CTFFIND4, set it to 30.

4. Run `tiltctf`. You can examine the results by opening the diagnostic .mrc file in the `tiltctf/` subfolder with 3dmod.

## Tomogram Reconstruction in NovaCTF

Final tomogram reconstruction for subtomogram averaging will be performed using novaCTF, which allows for 3D CTF-correction during reconstruction.
TOMOMAN will generate the appropriate scripts and output directory for running novaCTF and binning tomograms by Fourier cropping.

Open `tomoman_novactf.param`.

1. The directory parameters should already be correct.

2. The parallelization parameters determine how jobs are split between cores.
Use the same `n_cores` as you used for `nthreads` in [CTF estimation](#ctf-estimation).

3. Stack parameters are parameters for generating the aligned stacks prior to tomogram reconstruction.

    - `ali_dim` allows for resizing, though I recommend using the full image size: `3712,3712`.

    - `erase_radius` is for gold fiducial erasing; you should have this number from performing tilt series alignment.

    - `taper_pixels` is used to taper the edges of the rotated images when generating an aligned stack; 100 is usually sufficient.

    - `ali_stack_bin` is used to bin the image stack before reconstruction.
    For this tutorial, to save on computation time, we will use an `ali_stack_bin` of 4.

    >NOTE: binning is performed immediately prior to tomogram reconstruction, so all other parameters are in unbinned pixels.

4. The 3D CTF correction parameters set how novaCTF will perform 3D CTF correction.
We always recommend using the dose-filtered stack and correcting CTF using phase flipping.
For `defocus_step`, smaller steps produce more precise results at the cost of more computation time (see [the novaCTF publication](../reading.md#methods) for more information).
For this tutorial, set it to 50.

5. Tomogram reconstruction parameters have some specifics on how to perform the reconstruction.
We typically skip radial filtering.
The `tomo_bin` parameter allows you to set the final binning factors desired. Since we set `ali_stack_bin` to 4, the minimum allowed value here is 4.
For this tutorial, set binnings of 4 and 8 with `tomo_bin = 4,8`.

6. The `output_dir_prefix` sets the name of the tomogram output directories, which will be placed within the `root_dir`.
For instance, bin 4 tomograms will be placed in: [root_dir]/[output_dir_prefix]_bin4/.

7. The additional parameters include the `recons_list`, which allows for reconstructing a subset of tomograms.
You can leave it as none to reconstruct all non-skipped tomograms in the tomolist.

8. Fourier3D is a program for Fourier cropping volumes written by Beata Turoňová.
The `f3d_memlimit` parameter sets a limit to how much memory Fourier3D can use; more memory allows for faster computation times.
For this tutorial, set `f3d_memlimit` to 60000.

9. NovaCTF’s approach to CTF-correction assumes that the center of mass is at the center of the tomograms.
If this is off, the reconstructed tomogram will contain a systematic error in all planes.
To refine the tomogram center, novaCTF allows you to generate an offset value for recentering.
TOMOMAN can take an input STOPGAP motivelist, and use the center of mass of the particles as the refined center.
Since we have no such motivelist now, this can be left off.

10. Run novaCTF in the standalone.
This is typically the single longest computation step in the workflow.
If you are following this tutorial at a workshop, your instructors will provide you with a reconstructed tomogram to use for subsequent steps to save time.

11. After running novaCTF you can preview your reconstructed tomograms in 3dmod, for example:

        !3dmod novactf_bin8/TS_01_dose-filt.rec

TOMOMAN reconstruction is now finished.
