
# Orientational Alignment

Now that we have a reference with some level of structure we can do several things.
First, we will make a new alignment mask to focus on our structure.
After this, we can perform a 6D alignment that refines both 3D shifts and orientations.

## Alignment Mask Creation

Our goal is to produce an alignment mask that contains the entire structure.
Since alignment masks should always have soft edges, we want the soft edge to start just outside of our reference.

1. Start chimera (by running `chimera` in the terminal) and open `ref_2.mrc`.

    1. Maps written by STOPGAP are not contrast-inverted, so you will need to uncheck the “Cap high values at box faces” option in Volume Viewer > Features > Surface and Mesh Options.

    1. Set the voxel size to 1 in Volume Viewer > Features > Coordinates.

    1. Adjust the histogram slider so you can see the three layers in your reference.

2. Open the sphere mask.
To view the mask on top of the structure, it can be helpful to adjust the transparency of the mask under Volume Viewer > Features > Brightness and Transparency.
The position of your average in Z depends on a few factors such as your initial particle centering and radius, and as such, it will be different for everyone.
However, it is likely that the sphere mask does not adequately mask in your average.
For example, the sphere may not hold the entire reference or it may not be centered on it.

3. The shape of this structure is reasonably well-suited for a cylindrical mask.
Since the structure continues beyond the box boundaries in the XY-plane, the mask can be as large as possible without touching the box boundaries.
Through trial and error, produce a cylindrical mask that suits your reference.
An example that worked for me is:

        cyl_mask = sg_cylinder(32, 10, 16, 3, [17, 17, 14]);
        sg_mrcwrite('masks/cyl_mask.mrc', cyl_mask);

    >NOTE: Since your structure is probably a bit offset, you will need to define the center when using the `sg_cylinder` function.
    Center is the last argument in `sg_cylinder` and given as `[x, y, z]` coordinates.
    You can guess the distance you need to adjust or measure it in 3dmod.

## Run Angular Alignment

Now, we will start angular alignment.
Since we have not done any angular search yet, we will start with a rough angular alignment using large angular steps.

1. Generate alignment parameters using `stopgap_subtomo_parser.sh`.
    1. You will need to increment your `startidx` by one and update your `mask_name` to `cyl_mask.mrc`.
    1. We will use a coarse cone search with hill climbing, so the final parameters to decide on are the angular increments.
    The `angincr` and `angiter` parameters control the off-plane (i.e. off the XY-plane) search.
    If you want to be very precise, you could calculate half the angular offset between two particles from your inter-particle distance and radius; for me this is ~2°, so `angincr=2` and `angiter=2` should be plenty.
    1. For `phi_angincr` and `phi_angiter`, which control the in-plane search, we can use our knowledge that there is C6 symmetry, so the maximum error is ± 30°.
    For an initial coarse search, we can then set `phi_angincr=12` and `phi_angiter=3` to find the nearest symmetry element (with a bit extra).

1. Parse parameters and run alignment.

1. The reference should look pretty structured now.
Keep in mind, for iterative averaging, the quality of your alignment depends on the reference from the last round.
As such, it is often useful to run 2 iterations per parameter set and rarely useful to run more than 2.
Parse another iteration (remember to increment `startidx`) with the same parameters and run alignment again.

1. At this point, the reference should be relatively well resolved, looking like a grid of filled and empty spaces.
The symmetry axis we want to use is in one of the empty spaces, so if an empty space is not centered we need to shift the reference in the XY plane.
You can determine the amount of shift required in 3dmod.
Then, use `sg_motl_shift_and_rotate` in the STOPGAP Console to shift positions.
Inputs to this command are `input_name`, `output_name`, `shifts`, and `rotations`.
`shifts` are provided as a x-, y-, and z-shifts in square brackets while `rotations` are provided as Euler angles in square brackets.
For example, the shifting I performed was:

        sg_motl_shift_and_rotate('lists/allmotl_tomo1_obj1_4.star', 'lists/allmotl_tomo1_obj1_shift_4.star', [3,1,3], [0,0,0]);

1. Update the motivelist and reference names in the parser and generate an averaging run.
I typically append the reference name with whatever I appended the motivelist name with.
In this case I set `ref_name='ref_shift'`.
Generate a new average.

1. Compare the old and new references to make sure it was shifted properly.
If it wasn’t you may have applied the shifts with the wrong sign.
If so, re-shift the motivelist and re-average.

    >NOTE: If you applied a Z-shift, your cylinder mask is probably not in a correct position anymore.
    You can re-generate the same mask, but with the appropriate Z-centering.
    In my case it was:

        cyl_mask2 = sg_cylinder(32, 10, 16, 3, [17, 17, 17]);
        sg_mrcwrite('masks/cyl_mask2.mrc', cyl_mask2);

1. Now that the reference is properly centered along the symmetry axis, we can apply a C6 symmetry by setting `symmetry='C6'` in the parser.
With the shift, there may be a bit of off-plane error introduced, so increase the angular iterations to 3; `angiter=3`.
Parse parameters and perform another round of alignment.

1. The reference should look much better now.
Keep in mind, the output references from STOPGAP do NOT have symmetry applied. Symmetry is applied to the reference prior to alignment, but not during averging.

1. From here, we can refine the average a bit by reducing the angular search.
Since the out-of-plane search already used a small angle, we can leave the increment alone and reduce the iterations to 2; `angiter=2`.
For phi, we are arguably accurate within 12 degrees; reducing the phi increment to 4 with 4 iterations should be safe; `phi_angincr=4` and `phi_angiter=4`.
Update the parameters and run 2 iterations.

1. At this point the reference is largely converged.
If you check the FSC plot in the `fsc/` subfolder, the structure should be well beyond Nyquist.
