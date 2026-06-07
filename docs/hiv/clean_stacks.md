# Removing Bad Tilt Images

After generating your motion-corrected tilt series stacks, you now have something you can reasonably look at to assess the quality of the images.
Bad images can include images where something blocks the field of view, such as a grid bar or ice crystal, or bad imaging conditions such as sample charging.
The best way to assess this is by manually looking at the images.
TOMOMAN facilitates the process using the clean_stacks task.

## Cleaning Stacks
Open `tomoman_clean_stacks.param`.

1. The tomolist parameters block should be set up already.

2. For the stack cleaning parameters block, `clean_binning` refers to the binning factor for displaying the tilt series, `clean_append` is whether to save the cleaned stack with an appended name, and `force_cleaning` forces repeating of stacks that have already been cleaned.
The default parameters should work fine.

3. Save the param file and run stack cleaning.

        tomoman(pwd, 'tomoman_clean_stacks.param');

4. Follow the instructions in the console to remove bad images.
For this dataset, there is 1 blocked image that should be removed.

    There are some issues with running 3dmod in the TOMOMAN standalone on these AWS instances.
    You can instead open another terminal from the TOMOMAN standalone (`Ctrl + Shift + T`) and open the tilt series manually:

        3dmod TS_01/TS_01.st

    >NOTE: Remember to close your 3dmod windows; TOMOMAN is unable to close spawned windows from external packages.
