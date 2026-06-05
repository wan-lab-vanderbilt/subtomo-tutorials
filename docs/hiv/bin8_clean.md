
# Removing Overlapping and Bad Particles

Now that the structure has converged, we can take a look at how the particles have aligned by visualizing them as a lattice map.
We will use the Place Objects Chimera plugin for this.

1. Covert the motivelist to AV3 `.em` format in the STOPGAP Console using `sg_motl_stopgap_to_av3`.
For example:

        sg_motl_stopgap_to_av3('lists/allmotl_tomo1_obj1_shift_7.star');

1. Start Chimera and open the tomogram.
Remember that your tomogram is in your `tomo/uncorrected_sirt15_bin8/` directory.

1. Remember to open the coordinates panel in the Volume Viewer and set Origin index to 0 and Voxel size to 1, and that you should visualize the tomogram as planes.

1. Open the Place Object plugin (Tools > Utilities > Place Object).
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
For `d_cut`, choose a value that is smaller than the true interparticle distance. This can be measured in 3dmod.
Set `s_cut` to the CC value cutoff you determined in the previous step.
The settings I used are:

        sg_motl_distance_clean('lists/allmotl_tomo1_obj1_shift_7.star','lists/allmotl_tomo1_obj1_shift_dclean_7.star', 6, 0.35);

1. After cleaning, convert the motivelist to AV3 format and check it in Chimera.

    > NOTE: most of your particles may now look red; this is because the color scaling is relative to the lowest and highest CC values.

1. If you are satisfied with the cleaning, generate a new average with the cleaned motivelist.

1. If you check your FSC plot pre- and post-cleaning, you may find it has worsened.
Remember, FSC is NOT an objective resolution measure but instead a self-consistency measure.
Your FSC was likely over-inflated due to identical particles in both halfsets.
At this point, we can consider this final average the initial *de novo* reference.