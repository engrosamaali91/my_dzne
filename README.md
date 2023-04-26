
## Steps to run fastsurfer using command promt on linux
- In order to run the whole FastSurfer pipeline (for surface creation etc.) locally on your machine, a working version of FreeSurfer (v7.3.2, https://surfer.nmr.mgh.harvard.edu/fswiki/rel7downloads) is required to be pre-installed and sourced

- Activate the root directory from your home page (This step is valid if the installed version is installed in root directory)
```shell 
ls /groups/ag-reuter
```
- Go to the root directory where freesurfer is already installed, in this case freesurfer is installed in this software-centor directory
```shell
cd /groups/ag-reuter/software-centos
```
- List and locate $fs72$  which the installed version on the system 
- Copy the path to this directory 
```shell
pwd
```
- The path should look as follows
```shell 
ashrafo@euclid:/groups/ag-reuter/software-centos/fs72$ 
```
- Create python environment 
To create a Python virtual environment, you can follow these steps:

    1. Open a command prompt or terminal window.

    2. Navigate to the directory where you want to create the virtual environment.

    3. Type the following command to create a virtual environment:
    ```shell
    python -m venv myenv
    ```
    Replace myenv with the name you want to give to your virtual environment.
    
    4. Activate the virtual environment by running the following command:

    - On Windows:
    ```shell
    myenv\Scripts\activate.bat
    ```
    - On Linux or macOS:
    ```shell
    source myenv/bin/activate
    ```
    Once activated, you will see the name of your virtual environment in the command prompt or terminal.
    
    5. You can install packages and run Python scripts within the virtual environment without affecting the system-wide Python installation.

    When you're done working in the virtual environment, you can deactivate it by running the command:
    ```shell
    deactivate
    ```
    This will return you to your system's default Python installation.

- After activating the python envirnment install all the necessary Python 3 libraries installed (see requirements.txt) as well as bash-4.0 or higher (when using pip, upgrade pip first as older versions will fail).

 - To install all the dependencies listed in your requirements.txt file, you can use the following command:
 ```shell 
 pip install -r requirements.txt
 ```
 - Make sure to clone and change the directory to fastserver repository. https://github.com/Deep-MI/FastSurfer
 - ### Example 1: Native FastSurfer on subjectX (with parallel processing of hemis)
 ```shell
 # Source FreeSurfer
export FREESURFER_HOME=/path/to/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh

# Define data directory
datadir=/home/user/my_mri_data
fastsurferdir=/home/user/my_fastsurfer_analysis

# Run FastSurfer
./run_fastsurfer.sh --t1 $datadir/subjectX/t1-weighted-nii.gz \
                    --sid subjectX --sd $fastsurferdir \
                    --parallel --threads 4 \
                    --py python3
```

-For example the command line should look as follows
```shell 
export FREESURFER_HOME=/groups/ag-reuter/software-centos/fs732
source $FREESURFER_HOME/SetUpFreeSurfer.sh
datadir=/home/ashrafo/my_mri_data
fastsurferdir=/home/ashrafo/my_fastsurfer_analysis
./run_fastsurfer.sh --t1 $datadir/bert/001.mgz --sid bert --sd $datadir --parallel --threads 4 --py python3
```

- For visualizatoin on freeview run the following command 
```shell
freeview /home/ashrafo/my_fastsurfer_analysis/bert/mri/aparc.DKTatlas+aseg.deep.mgz /home/ashrafo/my_fastsurfer_analysis/bert/mri/norm.mgz
```
### Example 2: Example 5 Quick Segmentation
For many applications you won't need the surfaces. You can run only the segmentation (in 1 minute on a GPU) via
```shell
./run_fastsurfer.sh --t1 $datadir/subject1/t1-weighted.nii.gz \
                    --main_segfile $ouputdir/subject1/aparc.DKTatlas+aseg.deep.mgz \
                    --conformed_name $ouputdir/subject1/conformed.mgz \
                    --parallel --threads 4 --seg_only --py python3 --sid subject1 --sd /outputdir/subject1
```

This will produce the segmentation in a conformed space (just as FreeSurfer would do). It also writes the conformed image that fits the segmentation. Conformed means that the image will be isotropic in LIA orientation. It will furthermore output a brain mask (mask.mgz) and a simplified segmentation file (aseg.auto_noCCseg.mgz).


    
Here is the example 
```shell 
./run_fastsurfer.sh --t1 $datadir/bert_qs/001.mgz \
--main_segfile /home/ashrafo/quick_segmentation_output/osama/aparc.DKTatlas+aseg.deep.mgz \
--conformed_name /home/ashrafo/quick_segmentation_output/osama/conformed.mgz \
--parallel --threads 4 --seg_only --py python3 --sid osama \
--sd /home/ashrafo/quick_segmentation_output/osama/

```

Alternatively - but this requires a FreeSurfer install - you can get mask and also statistics after insertion of the corpus callosum by adding --vol_segstats in the run_fastsurfer.sh command:
```shell

./run_fastsurfer.sh --t1 $datadir/subject1/t1-weighted.nii.gz \
                    --main_segfile $ouputdir/subject1/aparc.DKTatlas+aseg.deep.mgz \
                    --conformed_name $ouputdir/subject1/conformed.mgz \
                    --parallel --threads 4 --seg_only --vol_segstats
```
    
    

    
    

