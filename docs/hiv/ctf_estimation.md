# CTF Estimation

Contrast Transfer Function (CTF) estimation is similar for tomography and single particle analysis in that Thon ring fitting is used in both.
However, tilted images have a defocus gradient as different areas of the field of view have different heights.
As such, Thon rings from each area of the specimen are different; these Thon rings do not sum coherently, and higher-resolution Thon rings are lost at higher tilt angles.
TOMOMAN includes `tiltctf`, an algorithm we developed to use tilt series alignment parameters to generate power spectra that account for this defocus gradient.
We then pass this power spectrum to CTFFIND4 for Thon ring fitting.

Open the `tomoman_tiltctf.param` file.

1. The directory parameters should already be correct.

2. The tiltctf parameters include the parameters for calculating power spectra.
Except where noted, default values are fine.

    1. A `ps_size` of 512 and a `def_tol`, defocus tolerance, of `0.05` µm is sufficient.
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
    For this dataset, the right side has greater defocus at positive tilt so set `handedness` to `1`.

3. The CTFFIND4 parameter block contains the same parameters for CTFFIND4.

    1. In general, the defaults work, though you may wish to increase the `min_res` and reduce the `max_res` for tomography.

    2. The `nthreads` parameter sets parallelization for CTFFIND4, set it to `14`.

4. Run `tiltctf`.

        tomoman(pwd,'tomoman_tiltctf.param');

5. You can examine the results by opening the diagnostic .mrc file in the `tiltctf/` subfolder with 3dmod.

        3dmod TS_01/tiltctf/diagnostic_TS_01_dose-filt_tiltctf_ps.mrc
