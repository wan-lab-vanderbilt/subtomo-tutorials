# Extracting Subtomograms

In this part, we will familiarize ourselves with STOPGAP's approach to generating a parameter file using a parser, then running that file. STOPGAP jobs are run by using a task-specific parser script (named `stopgap_*_parser.sh`) to generate a parameter file (named `*_param.star`) and then running that parameter file using the `run_stopgap.sh` script.

The first STOPGAP task we will perform is extracting subtomograms.


## Preparing Extraction Parameter File

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


## Preparing to Run STOPGAP with MPI


1. Open `run_stopgap.sh` in any text editor, for example, gedit using a terminal window:

        gedit run_stopgap.sh

    The main parameters here are the "run options" which manage parallelization and the "directories" block, which manages directories and paths.

2. For parallelization parameters, set `run_type` to `'local'`, `nodes` to 1, `n_cores` to `1`, and `copy_local` to 0.
The rest of the run options are SLURM-specific and can be ignored.

3. Set `rootdir` to the absolute path of your `init_ref/` folder (e.g. `~/HIV_dataset/subtomo/init_ref/`).
We will update `paramfilename` before running each job.

    >NOTE: Remember to end your `rootdir` path with a `/`!

4. Set `paramfilename` to `params/extract_param.star`.
Save the file and run STOPGAP by running `run_stopgap.sh` in a terminal.

        ./run_stopgap.sh

    >NOTE: STOPGAP is setup here to run through the stopgap_watcher, which is a separate program to track STOPGAP progress.
    This is not required; for running on clusters where programs are not allowed to be run on submission nodes, stopgap_watcher can be run on any computer that has access to the working directory.