# Initializing a STOPGAP folder for Bin 8 Averaging

In this part, we will first prepare the necessary files and folders for a STOPGAP subtomogram averaging job.
Then, we will perform "reference-free" subtomogram averaging on a single HIV VLP.
The product of this average will serve as a *de novo* reference for subtomogram averaging on the whole dataset.

## Preparing a STOPGAP Folder

Here we will initialize a subtomogram averaging folder with the necessary files and structure.

1. In a terminal (not the TOMOMAN standalone) change to your HIV_dataset directory.

        cd ~/HIV_dataset/

1. Make `subtomo/` and `subtomo/init_ref/` subdirectories.
   Change into the `init_ref/` directory.

        mkdir subtomo/
        mkdir subtomo/init_ref/
        cd subtomo/init_ref/     

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
