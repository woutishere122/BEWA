# Case Studies in Applied Experimental Psychology
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

Finally, we combined the files for both amygdala's into one file (`Combined_Amygdala`) in the `Seed` directory.
```bash
fslmaths AmyLeft_bin.nii.gz -add AmyRight_bin.nii.gz Combined_Amygdala.nii.gz
```

### Align seed mask to our data
After creating our seed mask, we tested how this looked on one of our own data files. However, when doing this, we realized that the seed mask was not aligned properly with our data. As you can see on the picture below, the dimensions and dimensions of the pixels are different.
![image](https://github.com/woutishere122/BEWA/assets/120474930/eff81335-1dc4-415e-a582-1a2d32f0a212)

To fix this, we resampled the mask file to fit our brain data. We first tried this using this command:
```bash
flirt -in Seed/Combined_Amygdala.nii.gz -ref sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -out Seed/mask.nii.gz -applyxfm
```
However, this resulted in the masks not aligning properly with our data (see picture).
![image](https://github.com/woutishere122/BEWA/assets/120474930/6e942632-0ef7-4f80-ba53-481f0bd2bcce)

Therefore, we tried a different approach using `3dresample`, which ended up working.
```bash
3dresample -input Seed/Combined_Amygdala.nii.gz -master sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -prefix Seed/mas2k.nii.gz
```
![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/4b87b431-46dc-4a58-b1fc-ff3affe7c269)


### Extract mean time series for each participant

We extract the time series of our amygdala by using the mask for each participant seperately. 

```bash
fslmeants -i  sub-9001/ses-1/func/sub-9001_ses-1_task-rest_bold.nii.gz -o sub-9001_ses-1_ts.txt -m Seed/mas2k.nii.gz
```
We repeat this code for all participants to extract the time series of all of the sessions of all participants


![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/204a7625-efe0-43b5-952a-36f946590824)












