# Case Studies in Applied Experimental Psychology
Documentation of an amygdala-based functional connectivity analysis using Neurodesk.

## Install Neurodesktop (To do)
Struggles met Docker
Hoe geïnstalleerd enz

## Download and tidy up data
downloading all the necessary files was time-consuming. To address this issue, we installed `datalad`, a data management tool to efficiently handle large-scale datasets.
```bash
pip install datalad-installer
datalad-installer git-annex -m datalad/packages
pip install datalad
```
This enabled us to download the dataset using the Neurodesk terminal. This made the data available in our own Windows system, as well as on Neurodesk.
```bash
# set current directory
cd neurodesktop-storage
```

## Seed-based functional connectivity analysis

### Create seed mask of bilateral amygdala
![image](https://github.com/woutishere122/BEWA/assets/120474930/325801d3-bc7f-405a-b733-d41e03625e62)
