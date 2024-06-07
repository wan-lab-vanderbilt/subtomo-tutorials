# Bin 4 Subtomogram Averaging with STOPGAP

## Preparing for Bin4 Processing

The main goal here is to prepare the motivelist for bin2 processing.
This includes rescaling the motivelist, applying the shifts to the extraction positions, and splitting halfsets.

1. In the STOPGAP Console, we will re-number the particles and assign odd/even halfsets:

        sg_motl_assign_halfsets('allmotl_dclean_sclean_2.star', 'allmotl_dclean_sclean_halfset_2.star', 'oddeven', 1);

    >NOTE: Since we have renumbered the subtomograms, this new motivelist will NOT work with the current averaging folder.

2. Rescale the motivelist.
This will rescale the extraction positions and apply shifts.

        sg_motl_rescale('allmotl_dclean_sclean_halfset_2.star', 'allmotl_bin4_1.star', 2, 1);

3. Make a new subtomogram averaging folder (`subtomo/bin4/`) and initialize it (`stopgap_initialize_folder.sh subtomo`).
Copy the wedgelist, STOPGAP tomolist, bin4 motivelist, and scripts into this new folder.
    - For simplicity, you can also rename the motivelist to `allmotl_1.star`.
    - The wedgelist keeps all information in unbinned pixels, so it can be used "as is" for any binning.
    - The paths in the tomolist will need to be updated; this can be done by making a new one using TOMOMAN or simply using a text editor.
    - Change `rootdir` in the bash scripts

4. Extract subtomograms.
Set the boxsize to 64 and update the pixelsize to 5.4 (1.35 Å/pix × 4 = 5.4 Å/pix).

## Bin4 Angular Refinement

Here, we will go over how to set up parameters for angular refinement.

1. Before we can generate our first average, we need to provide an alignment mask.
As with our bin8 processing, we can generate a simple sphere for this step.
E.g.:

        sphere = sg_sphere(64, 26, 3);
        sg_mrcwrite('masks/sphere.mrc', sphere);
   >NOTE: Remember to move your STOPGAP console to the new bin4 folder.

   Notice that the dimensions of the sphere are increased because binning is reduced.

1. Generate an average using the bin4 motivelist.

1. Generate a new alignment mask.
A cylinder should be sufficient again, but this time make one that only includes the outer structured layer.
For my average this works but as before you may need to adjust its size and position for yours:

        cyl = sg_cylinder(64, 26, 22, 3, [33, 33, 31]);
        sg_mrcwrite('masks/cyl_mask.mrc', cyl);

   >NOTE: This mask is a bit thinner than previous ones. Here, we are focusing on the ordered outer layer, not the disordered inner layer.

1. Generate a new CC mask.
Since we’ve already determined the true particle positions at bin8, the goal is no longer to generate a CC mask that allows our randomly seeded particles to find the nearest true particle.
Instead, we want to make a CC mask that allows for proper sampling but not too large that results in getting trapped in false local minima.
Arguably, a sphere with a 2 pixel radius should be sufficient to account for the bin8 precision, but a 4 pixel radius should still be safe.

        ccmask = sg_sphere(64,4);
        sg_mrcwrite('masks/ccmask.mrc',ccmask);

1. Set alignment parameters.
   We can keep the same low-pass filter resolution for the first alignment run.
   Since we have unbinned by a factor of 2 and increased the boxsize by a factor of 2, the same `lp_rad=6` setting should be fine.

   For angular alignment, the `angincr` of 2 deg we used earlier should still be sufficient (in practice, 2 deg will get you well past 10 Å resolution).
   Since our box is still relatively small, we can leave `angiter=3` without too much computational expense.

   In our last run, the `phi_angincr` and `phi_angiter` were set to sample a full 60 deg range.
   At this point, our angular error in phi is likely somewhere around 4 deg `phi_angincr` we set in the last iteration.
   As such, setting `phi_angincr=2` and `phi_angiter=3` should provide enough sampling.

1. Parse subtomo parameters and run one iteration of alignment.
   Since we haven't really updated parameters since last unbinning, it's not necessary to run 2 iterations.

   Looking at the output FSC, the average is nearly at sub-nanometer resolution, though we are limited in visualizing this due to pixelsize.
   Given that we have only used ~60 Å information in our alignment, this is a clear example that the resolution of high-resolution features is driven by the alignment of low-resolution data.

1. As noted above, `angincr=2` should be sufficient, so our main parameters to change ares the `angiter` and `lp_rad` parameters.
Given that our resolution high compared to the previous `lp_rad` setting, we can safely set our low pass radius to ~20 Å (`lp_rad=17`).
Since we have limited computational power, we can play with some methods for cutting down computation time.
Set the `angiter=2` and `search_mode='shc'`.

    <details><summary>
    Stochastic Hill Climbing (SHC)</summary>
    In standard hill climbing, the goal is to sample all possible orientations (within the desired search range) and take the highest scoring orientation; i.e. to move up the hill as quickly as possible.
    SHC instead randomizes the order of search angles, scores the prior best angle, and accepts the first better-scoring orientation.
    As a result, you are still moving up the hill, but potentially not as quickly as possible.

    Even though alignments are potentially suboptimal, SHC results in an incrementally better reference more quickly, so more iterations can be done in the same amount of time.
    Low to medium resolution information, i.e. the information you are using to align, is  still well-resolved, so further iterations will still improve the overall alignment of the dataset.

   SHC also scales well with respect to resolution.
   When aligning against lower resolution data, the difference between the optimal orientation and a slightly suboptimal orientation are minimal, and the CC may not pick up on the difference.
   As you progressively align with higher resolution information, it becomes easier to score the difference between a optimal and suboptimal orientations, so the chances of finding a better solution to the prior one is lower.
   When this approaches maximum computation time, SHC essentially becomes standard hill climbing.

    >NOTE: SHC is only  useful when refining angles of particles that are close to their true orientations.
    >SHC should NEVER be used during *de novo* reference generation or finding true particle positions from oversampled starting positions.
    </details></p>

1. At this point, the rest of the refinement towards the dataset’s information limit is largely the same process.

We will go over some of basic post-processing steps in the next section.

## Post-Processing

Here we will discuss some peculiarities with interpreting the structure of continuous surfaces where the structure runs off the box edges.

STOPGAP’s FSC calculations during subtomogram averaging are required for figure-of-merit weighting references prior to alignment.
As such, it is essential that they are calculated on the full reference structure.
This is different than when you are interpreting your structures, where only the central subunits are important.

Here, will generate an FSC mask to focus on the central hexamer, calculate FSC, and generate a sharpened reference.

1. Generate a cylindrical mask that focuses on the central subunit.
This will likely be similar to your alignment mask, but smaller in radius.
One I made was:

        fsc_mask = sg_cylinder(64, 12, 22, 3, [33, 33, 31]);
        sg_mrcwrite('masks/fsc_mask.mrc', fsc_mask);

2. In the STOPGAP Console, run FSC calculation with `sg_calculate_FSC`.
   There are a large number of parameters for this function, depending on what output you want. For the full parameter list, you can run `sg_calculate_FSC('help')`.

   Here, we will calculate the FSC and plot it and generate a b-factor sharpened, contrast inverted (density is positive) map.
   The`bfactor` value can be determined empirically, but 100 is a reasonable starting point.

        sg_calculate_FSC('refA_name','ref/ref_A_4.mrc','refB_name','ref/ref_B_4.mrc','mask_name','masks/fsc_mask.mrc','pixelsize',5.4,'symmetry','c6','bfactor',100,'ref_avg_name','ref/filt_4.mrc','x_label',1);

3. You should find that the FSC plot is significantly better than what STOPGAP outputs.
The output reference should also be less noisy and sharper.

    >NOTE: FSC estimations can be more accurate with tighter "body" masks, such as those generated using RELION.
    >However, it's important to provide a wide enough Gaussian dropoff, otherwise the masking will produce sharp edges and spurious correlations.
    >This is particularly true for structures like this HIV capsid, where the structure is continuous and runs off the edges of the box.
