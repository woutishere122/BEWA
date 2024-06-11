# Case Studies in the Analysis of Experimental Data
Documentation of an amygdala-based functional connectivity analysis using Neurodesk. We used the Terminal in Neurodesk to execute the code, as well as the interfaces of the programmes on Neurodesk (e.g. `FSL`).

## Install Neurodesk
We had to install `Neurodesktop` to analyse and preprocess the fMRI data. However, we first had to install `Docker Desktop` for Windows. We followed the instructions on the [Neurodesk website](https://www.neurodesk.org/docs/getting-started/neurodesktop/neurodeskapp/).

### Install Docker Desktop
We downloaded Docker Desktop for Windows, and installed it on our computers. We made a profile so we could access the application. After Docker Desktop was installed, we opened the command prompt in Windows an ran the following command in the Windows terminal to test that Docker was working correctly.

```bash
docker --version
docker run hello-world
```
In the Docker Desktop application, a container called `neurodeskapp` started running, ensuring us that Docker was working properly:

![image](https://github.com/woutishere122/BEWA/assets/120474930/ade7e5f0-fe17-4056-8469-544a9091e500)


### Open Neurodesk
Afterwards, we installed the Neurodesk app with the `Windows Installer`. When accepting the license agreement, it started to download. When the installation was complete, we launched the `Neurodesk app`.

#### Pitfall when trying to open Neurodesk and redownloading Docker Desktop
Normally, setting up takes a couple of minutes, after which you are forwarded to the Welcome page. However, this didn't work for one of us. We read that uninstalling and then reinstalling Docker Desktop could fix this issue, but there was no uninstaller in the files. Therefore, we decided to manually delete them. This was a big mistake: installing the Docker Desktop app didn't work, as it said it was already installed. After multiple attempts, we found a [website](https://docs.docker.com/desktop/uninstall/) that gave us instructions and places where to find residual files, so we could remove all files linked to Docker Desktop. 

However, this still didn't fix the issue. We had to go to the registery of the computer, and delete all things Docker-related in order to finally fix this issue.

### Setting up Neurodesk
After fixing all Docker-related issues, we opened `a local neurodesk` on the Welcome page of Neurodesk. There, we were greeted by a launcher called `JupyterLab`, where you could open `Neurodesktop` from. We opened it and chose the `Desktop Dynamic-Resolution (RDP)` option. This opened `Neurodesk`, which essentially is a `Linux desktop` containing various fMRI applications in it, and we were ready to go.

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
find "/neurodesktop-storage/ds000201" -type d -name "fmap" -exec rm -rf {} +
# Delete all subfolders 'dwi' in ds000201
find "/neurodesktop-storage/ds000201" -type d -name "dwi" -exec rm -rf {} +
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
Dear professor, if you read this: thank you for the pizza!

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

#### Pitfall: Registration not done correctly
When trying to conduct our second level analysis, we found out that because we were running multiple first-level analyses at once, the registration process was not done correctly. Therefore, we had to run almost all of our first-level analyses again. However, this time, we decided to make a bash script for it to automatize the process. 

This is what we did in the Unix terminal:
```bash
# Set working directory
cd /neurodesktop-storage/ds000201
# Make a .sh script in geditor
gedit first-level-analysis.sh
```
This opened up a text editor to make a script. We used the following code:
```bash
#!/bin/bash

for feat_dir in NormalSleep/*/
do
  feat_dir=${feat_dir%*/}
  echo "Processing $feat_dir"
  
  # Navigate to the FEAT directory
  cd $feat_dir
  
  # Load the design file
  feat design.fsf
  
  # Navigate back to the parent directory
  cd ../..
done

for feat_dir in SleepDeprivation/*/
do
  feat_dir=${feat_dir%*/}
  echo "Processing $feat_dir"
  
  # Navigate to the FEAT directory
  cd $feat_dir
  
  # Load the design file
  feat design.fsf
  
  # Navigate back to the parent directory
  cd ../..
done
```
For the sake of trying to be efficient, we also made a script that goes in the reversed order. In that way, one of us could start at the start, and one at the end and we could work our ways toward each other.

```bash
# Reverse order
#!/bin/bash

for feat_dir in $(ls -1 NormalSleep | sort -r)
do
  feat_dir="NormalSleep/${feat_dir}"
  echo "Processing $feat_dir"
  
  # Navigate to the FEAT directory
  cd $feat_dir
  
  # Load the design file
  feat design.fsf
  
  # Navigate back to the parent directory
  cd ../..
done

for feat_dir in $(ls -1 SleepDeprivation | sort -r)
do
  feat_dir="SleepDeprivation/${feat_dir}"
  echo "Processing $feat_dir"
  
  # Navigate to the FEAT directory
  cd $feat_dir
  
  # Load the design file
  feat design.fsf
  
  # Navigate back to the parent directory
  cd ../..
done
```

Next, we clicked `Save` in the text editor and closed it.

In the terminal, we then had to make this script executable in the following way:

```bash
chmod u+x first-level-analysis.sh
```

Finally, we executed the script. This script runs all `FEAT`-analyses consecutively for subfolders `NormalSleep` and `SleepDeprivation`.
```bash
./first-level-analysis.sh
```

### The FSL FEAT Higher-level Analysis
Now it was time for the group analysis across all participants. 

These analyses rely on three things:
- The specific conditions and explanatory variables
- Each of the individual subjects first-level analyses 
- A description of the `GLM model`

This one was quite complex, since it had to include all conditions and contrasts we wanted to test. For each participant, we wanted to investigate the functional connectivity of both the left and right amygdala, before and after sleep deprivation. 

First of all, we thought this would result in 4 conditions per participant: Normal sleep, left amygdala; Normal sleep, right amygdala; Sleep deprivation, left amygdala; Sleep deprivation, right amygdala). However, we later found out that the effect of the left and right amygdala was already embedded into our first-level analysis, and that we did not need to include this in our second-level analysis anymore. Furthermore, due to time constraints, we decided not to include the effect of sleep condition, and only focus on the fMRI scans following a normal night of sleep. 

Taking this into account, we just ended up with 43 inputs, one for each participant. Additionally, we had to make the distinction between low and high trait anxiety between participants, resulting in two explanatory variables.

#### Prepare model
We prepared an excel file to give ourselves an overview of every file that we would have to upload to run the analysis and prevent errors (and keep our sanity). This file included the specific file we would need, as well as the coding of the explanatory variables in our second level model and looked like this:

![image](https://github.com/woutishere122/BEWA/assets/120474930/c3a95940-ffc4-45a5-abcf-9758a79bac71)

#### Higher-level analysis: normal sleep only
Next, we started by selecting the `Higher-level analysis` in the `FEAT` – FMRI Expert Analysis Tool Window:

![image](https://github.com/woutishere122/BEWA/assets/120474930/36abe32a-19b2-4dcb-bcdf-abacfde20c39)

In case this is not opened anymore, you can open it via the terminal:
```bash
fsl &
```

In the `Data` tab:
- We selected the option `Inputs are lower-level FEAT directories`
- Number of inputs = 43

![image](https://github.com/woutishere122/BEWA/assets/120474930/5adf23aa-e847-4aa7-b5f5-386f0d198ce7)

Then we clicked `Select FEAT directories`. A Select input data window then appeared.

We then clicked the yellow folder on the right of each row to select the `.feat`-file that we previously generated for each participant from each first-level analysis. After selecting all relevant folders, it looked like this:

![image](https://github.com/woutishere122/BEWA/assets/120474930/20492eff-7e4b-42ef-8893-88bd75711265)

We then clicked `OK` (the squished button on the bottom right).

#### Name an Output Directory¶
Next, we clicked the yellow folder to the right of `Output directory`, chose `/neurodesktop-storage/ds000201/NormalSleepAnxietyComparison` and clicked `OK`.

![image](https://github.com/woutishere122/BEWA/assets/120474930/d913cb53-a73e-44f6-978e-69bd71643bbb)

#### General Linear Model setup
Next, in the `Stats` tab:

We used the default `Mixed effects: FLAME 1`. We use FLAME 1 in FSL for mixed effects modeling when for a robust, efficient, and accurate analysis that balances sensitivity and specificity. It is particularly suitable for typical fMRI studies with small to moderate sample sizes and provides reliable variance estimation and outlier handling. For most higher-level fMRI analyses, FLAME 1 offers a practical and effective approach.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/bcc0311a-ad47-4088-bfef-b234469631b5)

In the `Model setup wizard` we selected `two groups, unpaired` with the first group (low anxiety) counting up to 23 people.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/662dfd7a-dfa5-4df6-8bab-baa272bbb94c)

This caused us to have 2 main EVs, which we called: `Low trait anxiety` and `High trait anxiety`.

When looking at `Full model setup`, we see these two groups. However, because our input data was not categorised in order of anxiety groups, we had to manually fill in our matrix. In the columns with the EVs, `1` stands for having that explanatory variable, `0` for not having it. The `Group`-column was also matched to this, with `1` for having a low trait anxiety and 2 having a high trait anxiety.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/079673fd-3c0d-43c6-be4a-8b78e938b5ce)

For the `Contrasts & F-tests` tab in the full model setup, we contrasted Low and High trait anxiety in both directions to see the difference between them [1 -1] [-1 1]. Furthermore, we included the main effects of both groups as well (contrast 3 and 4, [1 0] [0 1]).

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/9b008f6c-298a-4ca2-8728-0e47ad2e424e)

Finally, in the `Post-stats` tab, we selected a Z threshold of `2.3` and a Cluster P threshold of `0.05`. By using a Z-value of `2.3`, researchers can effectively detect significant brain activations while controlling for false positives, making their findings robust and reliable.
Then, we pressed `Go` and ran our analysis.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/18aa83cf-cc64-4cb2-b7d8-c69215605fde)

The analysis ran, however, no differences between our `Trait anxiety groups` was found. We realised that, because one of us run the brain anatomy extraction for the first half of the participants, and the other the second half, that our lower-level analysis had too little data again, as we ran this reverse, meaning we both ran the lower-level analysis for the participants where we did not have the anatomical brains extracted for. This meant that our functional brain was mapped directly to the standart brain template, without being mapped to their anatomical brain first. This could cause relevant and significant brain activity to get lost. Realizing this, we started generating the anatomical brain extraction for all participants.

### Second analyses
Additionally, we wanted to run a second analyses which included both the `normal sleep` and the `sleep deprived` conditions. 

In the `Data` tab:
- We selected the option `Inputs are lower-level FEAT directories`
- Number of inputs = 88

#### General Linear Model setup
Next, in the `Stats` tab:

We used the default `Mixed effects: FLAME 1`. 

In the `Model setup wizard` we selected `two groups, unpaired` with the first group (low anxiety) counting up to 23 people.

This caused us to have 2 main EVs, which we called: `Low trait anxiety` and `High trait anxiety`. 
When looking at `Full model setup`, we see these two groups. However, because our input data was not categorised in order of anxiety groups, we had to manually fill in our matrix. In the columns with the EVs, `1` stands for having that explanatory variable, `0` for not having it. The `Group`-column was also matched to this, with `1` for having a low trait anxiety and 2 having a high trait anxiety.

Additionally, we added 3 EVs: `Normal sleep`, `Sleep Deprived`, and `Interaction`

`Normal Sleep` stands for the resting state fMRI session where people had slept normally the night before. `Sleep Deprived` stands for the resting state fMRI session where people slept less than normal the night before. `Interaction` stands for the interaction between someones trait anxiety and their sleep condition. We filled in a `1` if the condition was appliccable for the data. 

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/47d3c696-ff18-4963-aa60-74f73d8111e3)

For the Contrasts, we compared the `Trait anxiety groups` [1 -1 0 0 0], the difference between `Trait anxiety groups` between `Normal sleep` and `Deprived sleep` [1 -1 1 -1 0] [1 -1 -1 1 0]. A F-test checked for all 3 contrasts.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/8234a9db-63f5-4ec1-9d30-4f80a4a02ad4)

Finally, in the `Post-stats` tab, we selected a Z threshold of `2.3` and a Cluster P threshold of `0.05`. By using a Z-value of `2.3`, researchers can effectively detect significant brain activations while controlling for false positives, making their findings robust and reliable.
Then, we pressed `Go` and ran our analysis.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/11902ed0-d4b8-4ee7-81f0-628c114dd8f0)

However, when an error message immediatly popped up, saying we had incomplete registeries. It seems that our new method for doing the lower-level analysis made the same registration errors as before for one of us who ran it. This meant, due to time contraints, that we could not run all of the lower-level analysis again. We decided to prioritize the `Normal sleep` analysis, as this would cut the lower-level analysis time in half.

### Final analysis, normal sleep only

We reran all the lower-level analysis for the `Normal sleep` resting state data. When this was finished, we ran our first analysis again.

## Results

This is an example image out of our report.

![afbeelding](https://github.com/woutishere122/BEWA/assets/167521585/d798e292-c5e7-4c85-bfc4-8310f1c4b213)

We talk about these results further in the paper. Congrats! You have reached the end of our markdown :D !

## Common issues
### Error `command not found`
This is often the case when Neurodesk or the Unix terminal was closed off and the modules still have to be loaded. To fix this issue:
```bash
# For example, load the fsl module
module load fsl
```

### Not in the right working directory
Always make sure you are in the right working directory that contains the files you want to access. You can swith to a new working directory using the command `cd`.









