
# Translational Alignment
Since the HIV VLPs are not true spheres, our initial positions are quite rough.
This is particularly true for the radial position (Z-axis in the subtomograms).
In this step, we will perform a quick translational alignment with no angular search; this will improve the radial density in our reference, which will allow us to generate a tighter reference mask.

### `ccmask` Creation

A cross-correlation mask (ccmask) is used to restrict the particle shifts during alignment.
For this dataset, there is potentially a large error in the Z-direction, but error in the XY-plane is well defined.
The appropriate shape of mask for this type of error is a cylinder.
Since we seeded our positions at half the inter-subunit spacing, this is the maximum XY error and will be the radius of the mask.

Run this command in the STOPGAP Console to save a cylindrical mask 4 pixels wide and 24 high:

    ccmask = sg_cylinder(32, 4, 24);
    sg_mrcwrite('masks/ccmask.mrc', ccmask);

>NOTE: A ccmask should always be binary! Do not use any Gaussian dropoff.

### Run Translational Alignment

1. Open the subtomo parser.
Update the `subtomo_mode` to `'ali_singleref'`.

2. Set the angular search parameters.
STOPGAP has multiple search strategies, with overlapping parameter sets.
For now, set `search_mode` to `'hc'`, `search_type` to `'cone'`, and `cone_search_type` to `'coarse'`.
Since we don’t want to do any angular search for this iteration, set `angincr`, `angiter`, `phi_angincr`, and `phi_angiter` to `0`.

3. Set the bandpass filter settings.
In general, the high pass filter defaults (`hp_rad=1`, `hp_sigma=2`) is fine; this mainly suppresses any normalization issues with the central voxel in Fourier space.
An `lp_sigma` of 3 is usually fine.
More important is to keep track of the low-pass filter radius (`lp_rad`) during your run.
A rule of thumb is to make sure the `lp_rad` is less-than or equal to the Fourier radius where FSC=0.5.
Since we don’t really have any resolution in our map, we can arbitrarily set it to 60 Å for now.
STOPGAP sets filter values in Fourier pixels since real-space values do not round well, particularly for small boxsizes or high binnings.
You can covert resolution to Fourier pixels with:

    $$fpix = \frac{boxsize × pixelsize}{resolution}$$

    In our current settings we have a 32 pixel boxsize and a 10.8 Å pixelsize so 60 Å resolution corresponds to 5.76 Fourier pixels.
    Since we cannot set fractional pixels, set `lp_rad` to 6, which corresponds to a resolution of 57.6 Å.

4. Run the parser and run STOPGAP.

5. Check `ref_2.mrc` in 3dmod.
After this alignment, we now have the 3 layers we saw in the tomograms.
(Use the XYZ or isosurface view.)
Despite no angular alignment, we also already have some resolution of the in-plane structure.

