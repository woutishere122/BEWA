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



### Run the FSL FEAT First-level Analysis¶
For each subject, we then ran a first level FEAT analysis showing us the brain regions that have activity correlated to the bilateral amygdala activity. The instructions below will show how we did this for one subject, subject 9001.

First of all, we opened `FSL` using the Unix Terminal:
```bash
fsl &
```

This opened up a window from which we selected `FEAT FMRI analysis`, opening up `FEAT`: the FMRI Expert Analysis Tool.
![image](https://github.com/woutishere122/BEWA/assets/120474930/ed9dc44b-9e08-4fc4-ae84-2c159df46f0a)

From this window, we selected `Select 4D data`, to upload our input data for our first participant: `sub-9001_ses-1_task-rest_bold.nii.gz`, then we pressed OK.

Next, we selected `/neurodesktop-storage/ds000201/sub-9001` as our output directory to tell `FSL` to create a feat directory called sub-001.feat at the same level as the subject and seed directories. 

![image](https://github.com/woutishere122/BEWA/assets/120474930/f0d068e0-0680-4d80-86f7-c02b732ac483)

Next, in the Unix Terminal, check the number of volumes and repetition time of your data, to input into `FEAT` (make sure you are in the `ds000201` directory):
```bash
fslinfo sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz
```

In the output, `dim4` represents the amount of volumes, and `pixdim4` the repetition time. In this case, 193 and 2.5, respectively.

![image](https://github.com/woutishere122/BEWA/assets/120474930/8768bcda-d992-45cf-a374-560916bfc4dd)

We then filled in these numbers in `Total volumes` and `TR(s)` on the `FEAT` interface.

![image](https://github.com/woutishere122/BEWA/assets/120474930/365dadb5-bda5-4535-b54b-ade13fcf2da8)

Next, in the `Pre-stats` tab, since the data was already preprocessed, we changed the parameters like the picture below:

![image](https://github.com/woutishere122/BEWA/assets/120474930/efbbde9e-31a7-4839-8adf-702071f734ad)

Then, in the `Registration` tab, 
































## Common issues
### Error `command not found`
This is often the case when Neurodesk was closed off and the modules still have to be loaded. To fix this issue:
```bash
# For example, load the fsl module
module load fsl
```

### Not in the right working directory
Always make sure you are in the right working directory that contains the files you want to access. You can swith to a new working directory using the command `cd`.









