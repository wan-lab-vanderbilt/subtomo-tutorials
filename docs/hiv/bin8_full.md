# Aligning the Full Dataset

Here we will go over how to take your initial reference and align it against the full dataset.

## Prepare new folder

1. Make a new folder for averaging the full dataset (`subtomo/full/`) and initialize it for subtomogram averaging as you did with `init_ref/`.
Copy your previous wedgelist, tomolist, masks, and `.sh` scripts into their approrpiate places in `full/`.

1. Copy the full motivelist from `tomo/` to `subtomo/full/lists/`.

1. Copy the references from your final initial average into `full/ref/` and rename them as iteration 1.
I.e., `ref_shift_dclean_A_7.mrc` would become `ref_A_1.mrc`.
Technically, the weighted summed reference is not required, only the halfsets.

1. Extract subtomograms from all VLPs using the full motivelist.


## Perform alignment on full dataset with prior reference

1. Align the full dataset.
This problem is distinct from the *de novo* structure determine we performed for the initial dataset.
In *de novo* structure determination, we slowly coax the structure out by iterative refinement and gradually reducing our angular search space.
Here, we already have a good reference, so if our parameters are too coarse we may generate a worse reference than the one we put in.

    As such, our goal is to align the full dataset to the same precision that we aligned the initial reference; i.e. our angular increments should be the same as in the final round of the intial reference alignment.
    Therefore, the main parameter to change here is the angular iterations so that we sample wide enough.

    For computational efficiency, we can go a little coarser and get similar results.
    The parameters I used were: `angincr=2`, `angiter=2`,`phi_angincr=6`, `phi_angiter=5`.
    Specifically for the phi settings, these settings allow for an in-plane search of ±30 degrees, which is sufficient to find the nearest symmetry group.

    Set your parameters and run 1 iteration of alignment.

1. After alignment, the reference should look less noisy, though the resolution is still limited by the binning.
Using the full motivelist requires a lot of memory so we can first distance clean the overlapping particles.
Do this as before but don’t apply a score cutoff as we haven’t determined what it should be yet.

        sg_motl_distance_clean('lists/allmotl_2.star', 'lists/allmotl_dclean_2.star', 6, 0);

1. Convert the cleaned motivelist to AV3 format and open in Chimera.
   >NOTE: Sometimes there are rounding errors that results in CC values being slightly over 1; this will cause a "CC Range Error" in the Place Objects tool.
   If this occurs, manually set the CC-Range such that the maximum value is 1.

1. Determine an appropriate CC cutoff and parse the good particles.
E.g.:

        sg_motl_score_clean('lists/allmotl_dclean_2.star', 'lists/allmotl_dclean_sclean_2.star', 0.4);

1. Lastly, it can be worth randomizing the orientations around the symmetry axis. This can minimize any missing wedge related bias.
        sg_motl_randomize_eulers_by_symmetry('lists/allmotl_dclean_sclean_2.star','c6','lists/allmotl_dclean_sclean_randsym_2.star')

1. Generate a new average with the cleaned motivelist.
Since we are already well beyond Nyquist, it’s unnecessary to perform any more angular refinement.
We can go on to rescaling the motivelist to bin4.
