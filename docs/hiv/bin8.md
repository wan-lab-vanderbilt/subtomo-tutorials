# Bin 8 Subtomogram Averaging with STOPGAP

In this part, we will first prepare the necessary files and folders for a STOPGAP subtomogram averaging job.
Then, we will perform "reference-free" subtomogram averaging on a single HIV VLP.
The product of this average will serve as a *de novo* reference for subtomogram averaging on the whole dataset.

## Preparing a STOPGAP Folder

Here we will initialize a subtomogram averaging folder with the necessary files and structure.

1. In a terminal (not the TOMOMAN standalone) change to your HIV_tutorial directory.

1. Make `subtomo/` and `subtomo/init_ref/` subdirectories.
Change into the `init_ref/` directory.

1. Copy the initalization, parsing, and running scripts from the `$STOPGAPHOME` directory into `init_ref/`:

        cp $STOPGAPHOME/bash/stopgap_extract_parser.sh .
        cp $STOPGAPHOME/bash/stopgap_subtomo_parser.sh .
        cp $STOPGAPHOME/bash/run_stopgap.sh .

1. Run the initialize folder command with the subtomo task:

        stopgap_initialize_folder.sh subtomo

    The folder now has the required structure for subtomogram averaging jobs.
    Re-running `stopgap_intialize_folder` for other jobs will add the additional required folders without affecting old ones.

## Preparing Lists

For subtomogram averaging with STOPGAP, three lists are required.

The first is a motivelist.
We already have a motivelist but we will parse out the particles from a single VLP for generating our initial reference.

The second is a wedgelist which contains the necessary information for missing wedge compensation.

The third is a STOPGAP tomolist.
The tomolist links the paths and names of the tomograms to a `tomo_num` field which matches the motivelist, this is used for subtomogram extraction.

### Motivelist

1. In the TOMOMAN standalone, load the motivelist from the `tomo/` directory we have already generated.

        motl = sg_motl_read2('allmotl_1.star');

    >NOTE: There are two `sg_motl_read` functions; the difference is in how they load the data.
    While output from `sg_motl_read()` is formatted in a way that is a bit easier to read, it requires substantially more memory.

2. We will parse our desired subtomograms using logical indexing.
We will first index by tomo_num; in this case we want tomo 1.
(Since we only have one tomogram in this example, this will match all particles, but it would not if there were multiple tomograms in the motivelist.)

        idx1 = motl.tomo_num == 1;

3. Next we will index by object.
We will pick object 1:

        idx2 = motl.object == 1;

4. We will parse the subtomograms into a new motivelist:

        new_motl = sg_motl_parse_type2(motl, (idx1 & idx2));

    >NOTE: we combined the two indices using MATLAB’s logical operators.

5. Save the new motivelist:

        sg_motl_write2('allmotl_tomo1_obj1_1.star', new_motl);

    >NOTE: STOPGAP motivelists have the following format `[name]_[iteration].star`, where iteration is the iteration of the subtomogram averaging run.
    Our file ends with `_1.star` because it will be used for the first run.
    The rest of the name is arbitrary but should not contain non-letter characters except for underscores.

### Wedgelist

Generate a wedgelist from the TOMOMAN tomolist:

    sg_wedgelist_from_tomolist('tomolist.mat', 'wedgelist.star');

### Tomolist

Generate a STOPGAP tomolist:

    sg_extract_make_tomolist('tomolist.mat', [pwd,'/novactf_bin8/'], 'sg_tomolist.txt');

Copy the three lists into the `lists/` subfolder in your `subtomo/` directory.

        !cp allmotl_tomo1_obj1_1.star ~/HIV_dataset/subtomo/init_ref/lists/
        !cp wedgelist.star ~/HIV_dataset/subtomo/init_ref/lists/
        !cp sg_tomolist.txt ~/HIV_dataset/subtomo/init_ref/lists/

## Preparing to Run STOPGAP with MPI

STOPGAP jobs are run by using a task-specific parser script (named `stopgap_*_parser.sh`) to generate a parameter file (named `*_param.star`) and then running that parameter file using the `run_stopgap.sh` script.

1. Open `run_stopgap.sh` in any text editor, for example, gedit using a terminal window:

        gedit run_stopgap.sh

    The main parameters here are the "run options" which manage parallelization and the "directories" block, which manages directories and paths.

2. For parallelization parameters, set `run_type` to `'local'`, `nodes` to 1, `n_cores` to 14, and `copy_local` to 0.
The rest of the run options are SLURM-specific and can be ignored.

3. Set `rootdir` to the absolute path of your `init_ref/` folder (e.g. `~/HIV_dataset/subtomo/init_ref/`).
We will update `paramfilename` before running each job.

    >NOTE: Remember to end your `rootdir` path with a `/`!

4. Save `run_stopgap.sh`.

## Extract Subtomograms

With the lists and run script prepared, we are now ready to extract our subtomograms.

1. Open `stopgap_extract_parser.sh` in a text editor.

1. There are four blocks of options here: "Parser Options" which contains the path to the output parameter file; "Folder Options" which provides paths to folders; "File options" which are the input files for extraction; and "Extraction Parameters" which are settings for the output subtomograms.

1. In the Parser Options, the default `param_name` is fine.

1. Under Folder options, update the rootdir to the absolute path of the `init_ref/` directory.
The other directory parameters can be left alone; they are overrides to the standard STOPGAP structure.

1. Under File options, update the lists.

    - `motl_name='allmotl_tomo1_obj1_1.star'`

    - `wedgelist_name='wedgelist.star'`

    - `tomolist_name='sg_tomolist.txt'`

    Since these are all lists, they are assumed to be in the listdir.

    > NOTE: since we are providing a tomolist, `tomodir` is ignored and can be left as `'none'`.

1. Set the extraction parameters.
The default subtomo_name is `'subtomo'`.
For `boxsize`, `32` should be sufficient here.
Set `pixelsize` to `10.8` since we binned our 1.35 Å pixels by a factor of 8.
For `output_format`, we find that `'mrc8'` works well, this saves the subtomogram as an 8-bit .mrc file.

    > NOTE: While 8-bit only provides 256 gradations, we generally find this is sufficient for the local information contained within a subtomogram.
    During extraction, the subtomogram is cropped and its values are floated between 0 and 255, rounded, and saved.

1. Save and close the file.
Run in the terminal.

        ./stopgap_extract_parser.sh

This will generate a new parameter file in the `params/` folder.
Feel free to preview it if you're curious.

1. Return to `run_stopgap.sh` and set `paramfilename` to `params/extract_param.star`.
Save the file and run STOPGAP by running `run_stopgap.sh` in a terminal.

        ./run_stopgap.sh

    >NOTE: STOPGAP is setup here to run through the stopgap_watcher, which is a separate program to track STOPGAP progress.
    This is not required; for running on clusters where programs are not allowed to be run on submission nodes, stopgap_watcher can be run on any computer that has access to the working directory.

## Launching the STOPGAP Toolbox

We are now ready to load and use the STOPGAP Toolbox.
Its purpose is the same as the TOMOMAN standalone, to run MATLAB commands without needing a MATLAB license and installation.

Exit the TOMOMAN standalone with `exit`.
Use `cd` to change to the `init_ref/` directory:

    cd ../subtomo/init_ref/

Start the STOPGAP Toolbox by running:

    stopgap_toolbox.sh

## Calculate Starting Reference

"Reference-free" refers to the fact that we are not using an *external* reference.
Since a reference is always required for iterative alignment, we generate an starting reference by averaging the extracted subtomograms.
In this example, since we have picked our positions using geometry, we have rough starting angles and our initial reference will not be a sphere but instead have rough features.

1. Subtomogram averaging in STOPGAP always involves calculating a Fourier Shell Correlation (FSC) in order to output two halfmaps and a figure-of-merit weighted average.
Our motivelist doesn’t currently have A/B halfsets defined, so halfmaps will be randomly generated.

    For FSC calculation, an alignment mask is always required.
Since we don’t know the reference structure, we can simply provide a basic sphere with a Gaussian dropoff (always include a soft edge on your alignment masks).
To save a sphere mask into the `mask/` folder, change into your `subtomo/` directory, run this in the STOPGAP Toolbox:

        sphere = sg_sphere(32, 10, 3);
        sg_mrcwrite('masks/sphere.mrc', sphere);

    Check the mask using 3dmod.

        !3dmod masks/sphere.mrc

    What you want is a soft-edged mask that tapers to 0 before hitting the box edges.

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

### Translational Alignment

Since the HIV VLPs are not true spheres, our initial positions are quite rough.
This is particularly true for the radial position (Z-axis in the subtomograms).
In this step, we will perform a quick translational alignment with no angular search; this will improve the radial density in our reference, which will allow us to generate a tighter reference mask.

#### `ccmask` Creation

A cross-correlation mask (ccmask) is used to restrict the particle shifts during alignment.
For this dataset, there is potentially a large error in the Z-direction, but error in the XY-plane is well defined.
The appropriate shape of mask for this type of error is a cylinder.
Since we seeded our positions at half the inter-subunit spacing, this is the maximum XY error and will be the radius of the mask.

Run this command in the STOPGAP Console to save a cylindrical mask 4 pixels wide and 24 high:

    ccmask = sg_cylinder(32, 4, 24);
    sg_mrcwrite('masks/ccmask.mrc', ccmask);

>NOTE: A ccmask should always be binary! Do not use any Gaussian dropoff.

#### Run Translational Alignment

1. Open the subtomo parser.
Update the `subtomo_mode` to `'ali_singleref'`.

2. Set the angular search parameters.
STOPGAP has multiple search strategies, with overlapping parameter sets.
For now, set `search_mode` to `'hc'`, `search_type` to `'cone'`, and `cone_search_type` to `'coarse'`.
Since we don’t want to do any angular search for this iteration, set `angincr`, `angiter`, `phi_angincr`, and `phi_angiter` to `0`.

3. Set the bandpass filter settings.
In general, the high pass filter defaults (`hp_rad=1`, `hp_sigma=2`) is fine; this mainly suppresses any normalization issues with the central voxel in Fourier space.
An `lb_sigma` of 3 is usually fine.
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

### Angular Alignment

Now that we have a reference with some level of structure we can do several things.
First, we will make a new alignment mask to focus on our structure.

#### Alignment Mask Creation

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

#### Run Angular Alignment

Now, we will start angular alignment.
Since we have not done any angular search yet, we will start with a rough angular alignment using large angular steps.

1. Generate alignment parameters using `stopgap_subtomo_parser.sh`.
    1. You will need to increment your `startidx` by one and update your `mask_name` to `cyl_mask.mrc`.
    1. We will use a coarse cone search with hill climbing, so the final parameters to decide on are the angular increments.
    The `angincr` and `angiter` parameters control the off-plane (i.e. off the XY-plane) search.
    If you want to be very precise, you could calculate half the angular offset between two particles from your inter-particle distance and radius; for me this is ~2°, so `angincr=2` and `angiter=3` should be plenty.
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
With the shift, there may be a bit of off-plane error introduced, so increase the angular iterations to 4; `angiter=4`.
Parse parameters and perform another round of alignment.

1. The reference should look much better now.
Keep in mind, the output references from STOPGAP do NOT have symmetry applied. Symmetry is applied to the reference prior to alignment, but not during averging.

1. From here, we can refine the average a bit by reducing the angular search.
Since the out-of-plane search already used a small angle, we can leave the increment alone and reduce the iterations to 2; `angiter=2`.
For phi, we are arguably accurate within 12 degrees; reducing the phi increment to 4 with 4 iterations should be safe; `phi_angincr=4` and `phi_angiter=4`.
Update the parameters and run 2 iterations.

1. At this point the reference is largely converged.
If you check the FSC plot in the `fsc/` subfolder, the structure should be well beyond Nyquist.

### Clearing Overlapping & Bad Particles

Now that the structure has converged, we can take a look at how the particles have aligned by visualizing them as a lattice map.
We will use the Place Objects Chimera plugin for this.

1. Covert the motivelist to AV3 `.em` format in the STOPGAP Console using `sg_motl_stopgap_to_av3`.
For example:

        sg_motl_stopgap_to_av3('lists/allmotl_tomo1_obj1_shift_7.star');

1. Start Chimera and open the tomogram.
Remember that your tomogram is in your `tomo/bin8_aretomo/` directory.

1. Remember to open the coordinates panel in the Volume Viewer and set Origin index to 0 and Voxel size to 1, and that you should visualize the tomogram as planes.

1. Open the Place Object plugin (Tools > Utilities > Pick Particle).
Browse for and open the `.em` motivelist with Place Object.
Visualize using Hexagons with voxel-size 0.2 and Colour Style as Cross-Correlation.
Click Apply.

1. You may notice that the hexagon edges do not line up; this is because the rotation in your average is unlikely to be the same as Place Object’s particles.
You can adjust the Phi-Offset parameter to fix this.

1. You should see that most of the oversampled positions have converged and overlapped.
This is a good sign of true subunit positions.
In general, cross correlation (CC) scores are lower at the tops and bottoms, owing to the missing wedge.
There will also be defects in the lattice; this is expected as it is impossible to close a surface using just hexagons.
Areas around the defects will also typically have lower CC values.

1. Some particles with low CC values will be completely misaligned; this can be due getting trapped in local minima or particles that are in regions where there is no lattice.
We can determine what an appropriate CC value cutoff is by setting Visualization to Cross-Correlation and adjusting the Lower CC Threshold slider.
Determine and write down an appropriate threshold value to exclude low-scoring particles while preserving as many high-scoring as possible.

    > NOTE: the CC threshold is relative value that is affected by many factors such as binning and defocus of the tomogram, so you cannot reuse the same value between tomograms or datasets.

1. Clean the motivelist in the STOPGAP Console.
Set `s_cut` to the cutoff you determined in the previous step.
For `d_cut`, choose a value that is smaller than the true interparticle distance. This can be measured in 3dmod.
The settings I used are:

        sg_motl_distance_clean('lists/allmotl_tomo1_obj1_shift_7.star','lists/allmotl_tomo1_obj1_shift_dclean_7.star', 6, 0.35);

1. After cleaning, convert the motivelist to AV3 format and check it in Chimera.

    > NOTE: most of your particles may now look red; this is because the color scaling is relative to the lowest and highest CC values.

1. If you are satisfied with the cleaning, generate a new average with the cleaned motivelist.

1. If you check your FSC plot pre- and post-cleaning, you may find it has worsened.
Remember, FSC is NOT an objective resolution measure but instead a self-consistency measure.
Your FSC was likely over-inflated due to identical particles in both halfsets.
At this point, we can consider this final average the initial *de novo* reference.

## Aligning the Full Dataset

Here we will go over how to take your initial reference and align it against the full dataset.

1. Make a new folder for averaging the full dataset (`subtomo/full/`) and initialize it for subtomogram averaging as you did with `init_ref/`.
Copy your previous wedgelist, tomolist, masks, and `.sh` scripts into their approrpiate places in `full/`.

1. Copy the full motivelist from `tomo/` to `subtomo/full/lists/`.

1. Copy the references from your final initial average into `full/ref/` and rename them as iteration 1.
I.e., `ref_shift_dclean_A_7.mrc` would become `ref_A_1.mrc`.
Technically, the weighted summed reference is not required, only the halfsets.

1. Extract subtomograms from all VLPs using the full motivelist.

1. Align the full dataset.
This problem is distinct from the *de novo* structure determine we performed for the initial dataset.
In *de novo* structure determination, we slowly coax the structure out by iterative refinement and gradually reducing our angular search space.
Here, we already have a good reference, so if our parameters are too coarse we may generate a worse reference than the one we put in.

   As such, our goal is to align the full dataset to the same precision that we aligned the initial reference; i.e. our angular increments should be the same as in the final round of the intial reference alignment.
   Therefore, the main parameter to change here is the angular iterations so that we sample wide enough.
   The parameters I used were: `angincr=2`, `angiter=3`,`phi_angincr=4`, `phi_angiter=8`.
   Specifically for the phi settings, these settings allow for an in-plane search of ±32 degrees, which is sufficient to find the nearest symmetry group.

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

1. Generate a new average with the cleaned motivelist.
Since we are already well beyond Nyquist, it’s unnecessary to perform any more angular refinement.
We can go on to rescaling the motivelist to bin4.
