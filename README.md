# Case Studies in the Analysis of Experimental Data
Documentation of an amygdala-based functional connectivity analysis using Neurodesk. We used the Terminal in Neurodesk to execute the code, as well as the interfaces of the programmes on Neurodesk (e.g. `FSL`).

## Install Neurodesktop (To do)
Struggles met Docker
Hoe geïnstalleerd enz

## Download dataset
downloading all the necessary files was time-consuming. Therefore, we installed `datalad`, a data management tool to efficiently handle large-scale datasets. This code allowed us to download the dataset using the Neurodesk terminal, making the data available on our own Windows system, as well as on Neurodesk.
```bash
# Install the installer
pip install datalad-installer
datalad-installer git-annex -m datalad/packages
# Install datalad
pip install datalad
```
Next, we downloaded the folder `ds000201`, including subfolders for each participant.
```bash
# Set working directory
cd neurodesktop-storage
# Download dataset
datalad install ///openfmri/ds000201
```
However, we then saw that when we navigated to our current working directory, the files were not downloaded. The files existed but were empty, so we could not open them. 

![image](https://github.com/woutishere122/BEWA/assets/120474930/6f60b358-2f58-4b81-9744-b05335560837)

To fix this issue, we had to download all of the data via `datalad`:
```bash
# Download all data in the folder ds000201 from working directory neurodesktop-storage
datalad get ds000201
```
The datafiles are very large, therefore, you can also download the data for one subject only, to try out the analyses on one subject:
```bash
# Set wd to ds000201 which includes the subfolders per participant
cd ~/neurodesktop-storage/ds000201
# Download data of a subfolder, e.g. for participant 9001
datalad get sub-9001
```
If this does not work, you can also set the folder you want to download as your working directory, and download the contents of it like this:
```bash
datalad get .
```
## Tidy up the dataset structure (optional)
We did not know yet what the ideal structure of our data folders would be, however, we already deleted some folders including data that we would not need.

```bash
# Delete all subfolders 'fmap' in ds000201
find "~/neurodesktop-storage/ds000201" -type d -name "fmap" -exec rm -rf {} +
# Delete all subfolders 'dwi' in ds000201
find "~/neurodesktop-storage/ds000201" -type d -name "dwi" -exec rm -rf {} +
```
Next, we deleted the folders of the participants we would not be using.
```bash
# Set wd
cd ~/neurodesktop-storage/ds000201
# Remove specific participants
rm -rf sub-9002 sub-9002 sub-9003 sub-9004 sub-9005 sub-9008 sub-9009 sub-9011 sub-9014 sub-9020 sub-9023 sub-9025 sub-9026 sub-9029 sub-9032 sub-9033 sub-9036 sub-9038 sub-9039 sub-9040 sub-9045 sub-9046 sub-9047 sub-9048 sub-9049 sub-9054 sub-9058 sub-9064 sub-9065 sub-9069 sub-9075 sub-9078 sub-9079 sub-9086 sub-9087 sub-9089 sub-9093 sub-9094 sub-9095 sub-9096 sub-9098
```
Finally, we manually deleted the functional data of the remaining participants on the `hand task` and `sleepiness task`.

![image](https://github.com/woutishere122/BEWA/assets/120474930/489ff2e6-8b03-48a5-b1d9-31124ae7de9c)

## Seed-based functional connectivity analysis
We used the [Neuroimaging core website](https://neuroimaging-core-docs.readthedocs.io/en/latest/pages/fsl_fmri_restingstate-sbc.html), which provided us with an overview on how to create a seed-based connectivity analysis using `FSLeyes`. We used version `fsleyesGUI 6.0.7.4`, which is the newest version available in Neurodesk.

### Create seed mask of bilateral amygdala
First of all, using `FSLeyes`, we made two activation files selecting the Left Amygdala (`AmyLeft`), and the Right Amygdala (`AmyRight`). We identified these using the `Harvard Oxford Subcortical Structural Atlas` on a standard brain template (Click File ➜ Add standard ➜ Select MNI152_T1_2mm_brain.nii.gz ➜ Click Open).
![image](https://github.com/woutishere122/BEWA/assets/120474930/325801d3-bc7f-405a-b733-d41e03625e62)

To save the seed image, we clicked the save symbol (green box, bottom left) next to the seed image.

We saved the image as `Amygdala_Left` and `Amygdala_Right` in the `Seed` directory, a folder in our main folder `ds000201`.

Next, we needed to binarize these files (`AmyLeft_bin` and `AmyRight_bin`) to be able to use them as a mask. We did so in the therminal using `FSL`. 
```bash
# Go to the seed working directory
cd ~/neurodesktop-storage/ds000201/Seed
# Binarize the mask files
module load fsl
fslmaths AmyRight -thr 0.1 -bin AmyRight_bin
fslmaths AmyLeft -thr 0.1 -bin AmyLeft_bin
```
This is what it looked like after binarizing one amygdala file:
![image](https://github.com/woutishere122/BEWA/assets/120474930/789745e9-98f3-40b8-a2d6-ceac01932d27)
And after binarizing both of them:
![image](https://github.com/woutishere122/BEWA/assets/120474930/2890560f-15c1-4548-acde-224c73221e7d)


### Align seed mask to our data
After creating our seed masks, we tested how this looked on one of our own data files. However, when doing this, we realized that the seed masks were not aligned properly with our data. As you can see on the picture below, the dimensions of the mask and dimensions of the functional data file are different.
![image](https://github.com/woutishere122/BEWA/assets/120474930/eff81335-1dc4-415e-a582-1a2d32f0a212)

To fix this, we resampled the mask file to fit our brain data. We first tried this using this command, from the `ds000201 directory`:
```bash
# Resample Left Amygdala mask
flirt -in Seed/AmyLeft.nii.gz -ref sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -out Seed/LeftMask.nii.gz -applyxfm
# Resample Right Amygdala mask
flirt -in Seed/AmyRight.nii.gz -ref sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -out Seed/RightMask.nii.gz -applyxfm
```
However, this resulted in the masks not aligning properly with our data (see picture).
![image](https://github.com/woutishere122/BEWA/assets/120474930/6e942632-0ef7-4f80-ba53-481f0bd2bcce)

Therefore, we tried a different approach using `3dresample`, which ended up working. 
```bash
# Load correct package
module load afni
# Resample Left Amygdala mask file
3dresample -input Seed/AmyLeft.nii.gz -master sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -prefix Seed/FinalLeftMask.nii.gz
# Resample Right Amygdala mask file
3dresample -input Seed/AmyRight.nii.gz -master sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -prefix Seed/FinalRightMask.nii.gz
```
![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/4b87b431-46dc-4a58-b1fc-ff3affe7c269)


### Extract mean time series for each participant

Next, we extracted the time series of our amygdala by using the masks. In the `ds000201` directory, we created a new folder called `time-series`. We then used this code to extract the time series for each participant separately (in this case, `sub-9001`) into a `.txt` file in the `time-series` folder. The example below is to get the time series for participant 9001 at session 1 for the left amygdala.

```bash
fslmeants -i  sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -o time-series/sub-9001_ses-1_left_ts.txt -m Seed/FinalLeftMask.nii.gz
```
We repeated this code FOUR TIMES for each participant to extract the time series of for both sessions for both the left and right amygdala for each participant.


You can view the timecourse in the text file using `FSLeyes`:

- Display `sub-9001_ses-1_bold.nii.gz` in `FSLeyes` (File → Add from file → `sub-9001_ses-1_bold.nii.gz`)

- On the `FSLeyes` menu, click View → Time series. This displays the time series of the voxel at the crosshairs.

- Move the crosshairs to other parts of the brain, or move it off the brain, so the time series is a zero-line.

- Click Settings → Time series 2 → Import and select the newly created `sub-9001_ses-1_left_ts.txt` file.

- Click OK. The timecourse of the txt file should be displayed. 

![image](https://github.com/woutishere122/BEWA/assets/120474930/1a54b5a3-f335-47bf-b8c7-c730d262ade8)

### Normalize T1 weighted structural images
This was the point we realised that our data might not have been preprocessed (nothing about this could be found in the dataset description or manuscript [Dataset manuscript](https://openarchive.ki.se/xmlui/bitstream/handle/10616/45181/Manuscript_Gustav_Nilsonne_v2_2017.pdf?sequence=4&isAllowed=y)). 
Due to time constraints not allowing us to fully preprocess the data and start all over again, we decided to try and save what could be saved, and  preprocesses our data while also extracting the level 1 analysis through FSL's `FEAT`. However, we first needed to normalize our structural data for this using FSL's `BET`. The example below is how we did this for participant 9001 at session 1, however we had to do this for all participants seperately.

We first went to `FSL` using the Unix Terminal:

```bash
fsl &
```

Next, we selected `BET brain extraction`.

In the interface that pops up, we set the functional intensity threshold to 0.2.

Next, we selected input image `/neurodesktop-storage/ds000201/sub-9001/ses-1/anat/sub-9001_ses-1_T1w` and output image `/neurodesktop-storage/ds000201/sub-9001/ses-1/anat/sub-9001_ses-1_T1w_brain` and clicked `Go`.

This created the file `sub-9001_ses-1_T1w_brain.nii.gz` in the same directory as our input image.

### Run the FSL FEAT First-level Analysis¶
For each subject, we then ran a first level `FEAT` analysis showing us the brain regions that have activity correlated to the bilateral amygdala activity. Simultaneously, we performed some preprocessing steps. The instructions below will show how we did this for one subject (subject 9001) at session 1 for the bilateral amygdala.

First of all, we opened `FSL` using the Unix Terminal:
```bash
fsl &
```

This opened up a window from which we selected `FEAT FMRI analysis`, opening up `FEAT`: the FMRI Expert Analysis Tool.
![image](https://github.com/woutishere122/BEWA/assets/120474930/ed9dc44b-9e08-4fc4-ae84-2c159df46f0a)

From this window, we selected `Select 4D data`, to upload our input data for our first participant: `sub-9001_ses-1_task-rest_bold.nii.gz`.

Next, we selected `/neurodesktop-storage/ds000201/sub-9001_1` as our output directory to tell `FSL` to create a feat directory called sub-9001_1.feat at the same level as the subject and seed directories. 

![image](https://github.com/woutishere122/BEWA/assets/120474930/f0d068e0-0680-4d80-86f7-c02b732ac483)

Normally, FEAT should automatically fill out the `Total volumes` and `TR(s)`. However, if this is not the case, you can check the number of volumes and repetition time of your data using the Unix Terminal (make sure you are in the `ds000201` directory):
```bash
fslinfo sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz
```

In the output, `dim4` represents the amount of volumes, and `pixdim4` the repetition time. In this case, 193 and 2.5, respectively.

![image](https://github.com/woutishere122/BEWA/assets/120474930/8768bcda-d992-45cf-a374-560916bfc4dd)

We then filled in these numbers in `Total volumes` and `TR(s)` on the `FEAT` interface.

![image](https://github.com/woutishere122/BEWA/assets/120474930/365dadb5-bda5-4535-b54b-ade13fcf2da8)

Next, in the `Pre-stats` tab, we changed the parameters like the picture below. Note that we checked `BET brain extraction` to extract only the brain structures from our functional image, as well as an interleaved `Slice timing correction` since our data was acquired using an interleaved method:

![image](https://github.com/woutishere122/BEWA/assets/120474930/aff2001d-b8f3-46cf-8355-bd08aef67d78)

Next, in the `Registration` tab, we selected the brain-extracted (which was done previously using `BET)` T1 weighted image of participant 9001 at session 1 as `Main structural image`, together with `Full search` and `BBR`. Next, as `Standard space`, we selected the `MNI152_T1_2mm_brain template`, `Full search` and `12 DOF` to optimize the fit of the mapping.

![image](https://github.com/woutishere122/BEWA/assets/120474930/fcd5c9dd-3c00-4b47-b897-96c885106e73)


We then used these settings in the Stats tab, and clicked `Full model setup`.

![image](https://github.com/woutishere122/BEWA/assets/120474930/f75ce0ac-38af-4b44-adc5-975c83b7a359)

This opened up a `General Linear Model` window. The GLM window is where you define the parameters of your model, including explanatory variables (EVs), contrasts, and how they will be used to analyze the time series data from your fMRI scan. These are the settings we used:

- Set `Number of original EVs` to 2 (one for the left amygdala, one for the right amygdala). We did this to see the combined or separate effects of the left and right amygdala on the rest of the brain.
- We named the first EV `AmyLeft`, the second one `AmyRight`.
- Next, we selected `Custom` (1 entry per volume) for the Basic Shape for Both EVs, to ensure that each EV is defined by its respective time series file.
- Next to `Filename`, we selected `/neurodesktop-storage/ds000201/time_series/sub-9001_ses-1_left_ts.txt` for the first EV and `/neurodesktop-storage/ds000201/time_series/sub-9001_ses-1_right_ts.txt` for the second EV. These are the mean time series of the left and right amygdala for sub-9001 at session 1 and are the statistical regressors in our GLM model.
- The first-level analysis will identify brain voxels that show a significant correlation with the seed (left and right amygdala) time series data.
- We then selected `None` for Convolution, and deactivated `Add temporal derivate` and `Apply temporal filtering`.

The GLM looked like this for both EVs:

![image](https://github.com/woutishere122/BEWA/assets/120474930/39cb26dd-2ee5-4ab3-bb6b-7ed9519d28fc)

Next, we clicked the `Contrasts & F-tests` tab, and followed these steps:

- Set `Contrasts` to 2, one for each amygdala.
- Set title `AmyLeft` and set `EV1` to one for the first contrast.
- Set title `AmyRight` and set `EV2` to one for the second contrast.
- Click `Done`

 This displayed a blue and red design matrix, which we then closed:
 
 ![image](https://github.com/woutishere122/BEWA/assets/120474930/05cd804a-19fc-4aaa-939d-b9e1af1d9f95)

We then didn't change anything in the `Post-stats` tab of `FEAT`, and clicked `Go` to run the first-level analysis.

We then found a newly created output folder in our chosen output directory, which included a `report`-file. When opening this report in our browser, we saw this `FEAT Report` window that indicated that the analysis was still running.

![image](https://github.com/woutishere122/BEWA/assets/120474930/e26c133a-cacb-427a-958c-ea2f2cc9c7a7)

After 15-20 minutes of waiting for the analysis to finish, we could click `Post-stats`, to see the following results:

![image](https://github.com/woutishere122/BEWA/assets/120474930/a43d5c77-84d4-4826-af4e-6181f011f104)

![image](https://github.com/woutishere122/BEWA/assets/120474930/158dd6d7-72e9-482e-9f8a-9a0426a231da)

![image](https://github.com/woutishere122/BEWA/assets/120474930/480a6e40-4bd9-4bc2-a462-5f4546ee3edd)

We then repeated this process for each participant. This took a LONG time. We tried running multiple analyses at once but the memory and processor could only handle a few analyses at once, or the programme would shut down and cause errors in the analyses, in which case we had to run the analysis again. Finally, after various hours, we completed all first level analyses and could move on to the higher level analysis.

### The FSL FEAT Higher-level Analysis
Now it was time for the group analysis across all participants. 

These analyses rely on three things:
- The specific conditions and explanatory variables
- Each of the individual subjects first-level analyses 
- A description of the `GLM model`

This one was quite complex, since it had to include all conditions and contrasts we wanted to test. For each participant, we wanted to investigate the functional connectivity of both the left and right amygdala, before and after sleep deprivation. This resulted in 4 conditions per participant: `NS_Left` = Normal sleep, left amygdala; `NS_Right` = Normal sleep, right amygdala; `SD_Left` = Sleep deprivation, left amygdala; `SD_Right` = Sleep deprivation, right amygdala). Additionally, each participant was also assigned to either the `Low trait anxiety` or `High trait anxiety` group. With 46 participants (and 4 participants missing the data from the second fMRI recording session), this resulted in `176 input files` in total.

#### Find relevant files for left and right amygdala activation
First of all, since we did our first level analyses for the left and right amygdala simultaneously for each participant, we had to find the relevant files for the left and right amygdala separately in the `FEAT`-folders. 

When looking in the `FEAT` folders, we found `zstat1.nii.gz` for the left amygdala and `zstat2.nii.gz` for the right amygdala. These files represent the amygdala (left or right) - based functional connectivity for a specific participant, and look like this:

![image](https://github.com/woutishere122/BEWA/assets/120474930/3bb910d4-ed0f-4fca-89bf-e21778c3989b)

#### Prepare model
Next, we prepared an excel file to give ourselves an overview of every file that we would have to upload to run the analysis and prevent errors (and keep our sanity). This file included the specific file we would need, as well as the coding of all explanatory variables in our second level model and looked like this:

![image](https://github.com/woutishere122/BEWA/assets/120474930/c9feea5a-022f-4fc2-a5e0-cdc105604459)

#### Higher-level analysis using FEAT
First of all, we started by selecting the `Higher-level analysis` in the `FEAT` – FMRI Expert Analysis Tool Window:

![image](https://github.com/woutishere122/BEWA/assets/120474930/36abe32a-19b2-4dcb-bcdf-abacfde20c39)

In the `Data` tab:
- We selected the option `Inputs are 3D cope images from FEAT directories`
- Number of inputs = 176

![image](https://github.com/woutishere122/BEWA/assets/120474930/27083d11-dd42-4eb6-ba68-152f9120b868)

Then we clicked `Select cope images`. A Select input data window then appeared:

![image](https://github.com/woutishere122/BEWA/assets/120474930/ccaed905-f87d-44b3-a5ef-605a0516d99b)

We then clicked the yellow folder on the right of each row to select the `zstat2.nii.gz`-file or `zstat2.nii.gz`-file that we previously generated for each participant from each first-level analysis. After selecting all relevant folders, it looked like this:

INSERT PICTURE

We then clicked `OK` (the squished button on the bottom right).

#### Name an Output Directory¶
Next, we clicked the yellow folder to the right of `Output directory`, chose `/neurodesktop-storage/ds000201/HigherLevelAnalysis` and clicked `OK`.

![image](https://github.com/woutishere122/BEWA/assets/120474930/180a5a61-94aa-4e19-a730-dc149b98f7ea)



WIP
Next, on the `Stats` tab, we clicked `Full model setup`. This opened up a `General Linear Model` window, in which we named the model PCC and ensure the interface looks like this:
























## Common issues
### Error `command not found`
This is often the case when Neurodesk was closed off and the modules still have to be loaded. To fix this issue:
```bash
# For example, load the fsl module
module load fsl
```

### Not in the right working directory
Always make sure you are in the right working directory that contains the files you want to access. You can swith to a new working directory using the command `cd`.









