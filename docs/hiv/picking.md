# Particle Picking in Chimera

In this section, we will pick spheres in UCSF Chimera using Kun Qu’s Pick Particle plugin ([link](https://www.biochem.mpg.de/7940000/Pick-Particle)).
We will then update these metadata on the tomolist and generate a STOPGAP motivelist of these particles.
We will then visualize the particle positions in UCSF Chimera using Kun Qu’s Place Object plugin ([link](https://www.biochem.mpg.de/7939908/Place-Object)).

## Picking Spheres

In Chimera, we will pick centers using Volume Tracer and set radii for each sphere using the Pick Particle tool.
The plugins mentioned above are not standard in Chimera, but we have them installed in a custom Chimera installation. Open a terminal and run `chimera2` to access this installation.

1. Open the bin 8 tomogram.
It should be saved in `tomo/novactf_bin8/`.
Chimera may open the tomogram as an isosurface volume.
If so, visualize it as planes (select Features > Planes, click "One," and use the slider) and set the appropriate levels on the histogram.
You may want to play around in the Chimera viewer to get familiar with the functions of all three mouse buttons in panning, zooming, moving planes, etc.

2. In the Volume Viewer window, open the Coordinates panel by going to Features > Coordinates.
Set the Origin index to 0 and the Voxel size to 1.
Press enter after changing each setting.
You may need to recenter and reorient the view; there are buttons to help with that.

3. If you want more contrast to better visualize the VLPs, you can apply a gaussian filter (Volume Viewer > Tools > Volume Filter).
A gaussian with of 1 should be sufficient, but remember to uncheck the "Displayed subregion only" under Options before clicking Filter.

4. Before picking VLPs, it may be useful to shift the camera to use orthographic projections (Main Window > Tools > Camera > Projection > Orthographic).
Orthographic side view disables "scaling with distance" of objects far from viewing plane due to perspective effect.

5. In the Volume Viewer, open the Volume Tracer tool.
Try setting the marker radius to 20, which makes a sphere smaller than the VLPs, but large enough to assess centering.
Move through the planes and place a marker at the center of each VLP.
We recommend taking only the complete VLPs.
This tomogram has around 9 complete VLPs.

6. When you have finished marking the VLPs, select File > "Save current marker set as..." and save the marker set into the tilt series folder (`TS_01/`) as `metadata/sphere/sphere.cmm`.
This naming convention is important for parsing the metadata files into the tomolist later.

7. In the main window, open the Pick Particles tool (Tools > Utilities > Pick Particle).
Browse for the marker file and press Display.
A series of sliders will a appear for each marker.
Adjust them until the radius for each as needed.
Since the VLPs are not perfect spheres, adjust the radius to a point with the best compromise.
Also, there are several concentric layers to each VLP; set the radius for each sphere so that the edge is on similar layers.
For these VLPs, we suggest setting the radius to between the two outermost layers.

8. When all radii are set, press Save.
For now, you can click reset to remove the spherical wireframes and close the Pick Particles window.
Leave Chimera open and go back to the TOMOMAN Console.

## Generating Motivelists

After picking our spheres, we will append this data to the tomolist and use functions in the STOPGAP toolbox to generate a motivelist.

1. We will add the picked spheres to the tomolist using the `tm_metadata_add_new` function.
The input parameters are `root_dir`, `tomolist_name`, and `type`.
This function loads a tomolist, goes through each tilt series directory and scans the `metadata/` subfolder for a subfolder named `type`, and stores the names of all files in that folder.
These types can then be used by other functions that parse the tomolist.
In this case, type is `sphere`.
If your TOMOMAN Console is still in `tomo/` you can use this command:

        tm_metadata_add_new([pwd,'/'], 'tomolist.mat', 'sphere');

2. To generate a motivelist, we will use the `sg_motl_batch_sphere` function.
This function generates particle coordinates along the surface of the input spheres in a defined way.
The parameters for this function are:
    * `tomolist_name` – Name of tomolist
    * `output_name` – Name of output motivelist (file extension should be .star)
    * `metadata_type` – Type of metadata; in this case ‘sphere’
    * `binning` – Binning of input files
    * `p_dist` – Distance between particles; set to half the true distance (e.g. 3.5) for oversampling
    * `rand_phi` – Randomize starting phi angles; set to 1
    * `padding` – Removes particles within a given distance of the edge of the tomogram
    * `subset_list` – Optional file listing a subset of tomograms to process

    As an example, this should generate around 1,850 motivelist entries per sphere:

        sg_motl_batch_sphere('tomolist.mat', 'allmotl_1.star', 'sphere', 8, 3.5, 1, 16, []);

    Note that we are "oversampling," creating more objects than there are truly proteins of interest in the tomogram.
    This ensures that every true particle is picked at least once and we will subsequently cull unecessary objects.

3. The Place Objects tool in Chimera only opens AV3-format .em files, so we will need to convert our STOPGAP motivelist using the `sg_motl_stopgap_to_av3` function.
As an example:

        sg_motl_stopgap_to_av3('allmotl_1.star');

4. In Chimera, open the Place Object function (Main Window > Tools > Utilities > Place Object).
Open the .em motivelist file and apply.
This will display the particle positions as spheres, colored by their class number.

5. To visualize particles with angular information, you can change the Object style using the dropdown.
Hexagons with voxel-size 0.1 are reasonable here.
Notice that the particle positions are evenly spaced with their planes along the surface of the spheres but their in-plane angles (phi) are randomized.
