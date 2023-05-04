
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
    Replace myenv with the name you want to give to your virtual environment.Also check which python version is installed on your system ```python --version``` and add its version as python2 or python3 accordingly. 
    
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

 - To install all the dependencies listed in your requirements.txt file make sure you have sources the virtual environment, you can use the following command:
 ```shell 
 pip install -r /path/to/requirements.txt
 ```
 - Make sure to clone and change the directory to fastserver repository. https://github.com/Deep-MI/FastSurfer
 - ### Example 3: Native FastSurfer on subjectX (with parallel processing of hemis)
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


### Example: Run Freesurfer on docker 

After pulling one of our images from Dockerhub, you do not need to have a separate installation of FreeSurfer on your computer (it is already included in the Docker image). However, if you want to run more than just the segmentation CNN, you need to register at the FreeSurfer website (https://surfer.nmr.mgh.harvard.edu/registration.html) to acquire a valid license for free. The directory containing the license needs to be mounted and passed to the script via the --fs_license flag. Basically for Docker (as for Singularity below) you are starting a container image (with the run command) and pass several parameters for that, e.g. if GPUs will be used and mounting (linking) the input and output directories to the inside of the container image. In the second half of that call you pass parameters to our run_fastsurfer.sh script that runs inside the container (e.g. where to find the FreeSurfer license file, and the input data and other flags).

To run FastSurfer on a given subject using the provided GPU-Docker, execute the following command:

```shell
# 1. get the fastsurfer docker image (if it does not exist yet)
docker pull deepmi/fastsurfer 

# 2. Run command
docker run --gpus all -v /home/user/my_mri_data:/data \
                      -v /home/user/my_fastsurfer_analysis:/output \
                      -v /home/user/my_fs_license_dir:/fs_license \
                      --rm --user $(id -u):$(id -g) deepmi/fastsurfer:latest \
                      --fs_license /fs_license/license.txt \
                      --t1 /data/subjectX/t1-weighted.nii.gz \
                      --sid subjectX --sd /output \
                      --parallel
```
Note: If you would like to process the segmentation part from the pipeline add the parameter ```shell --seg_only``` in the add, this reduces the processing to 15 seconds.
Docker Flags:
* The `--gpus` flag is used to allow Docker to access GPU resources. With it, you can also specify how many GPUs to use. In the example above, _all_ will use all available GPUS. To use a single one (e.g. GPU 0), set `--gpus device=0`. To use multiple specific ones (e.g. GPU 0, 1 and 3), set `--gpus 'device=0,1,3'`.
* The `-v` commands mount your data, output, and directory with the FreeSurfer license file into the docker container. Inside the container these are visible under the name following the colon (in this case /data, /output, and /fs_license). 
* The `--rm` flag takes care of removing the container once the analysis finished. 
* The `--user $(id -u):$(id -g)` part automatically runs the container with your group- (id -g) and user-id (id -u). All generated files will then belong to the specified user. Without the flag, the docker container will be run as root which is discouraged.

FastSurfer Flags:
* The `--fs_license` points to your FreeSurfer license which needs to be available on your computer in the my_fs_license_dir that was mapped above. 
* The `--t1` points to the t1-weighted MRI image to analyse (full path, with mounted name inside docker: /home/user/my_mri_data => /data)
* The `--sid` is the subject ID name (output folder name)
* The `--sd` points to the output directory (its mounted name inside docker: /home/user/my_fastsurfer_analysis => /output)
* The `--parallel` activates processing left and right hemisphere in parallel

Note, that the paths following `--fs_license`, `--t1`, and `--sd` are __inside__ the container, not global paths on your system, so they should point to the places where you mapped these paths above with the `-v` arguments (part after colon). 

A directory with the name as specified in `--sid` (here subjectX) will be created in the output directory if it does not exist. So in this example output will be written to /home/user/my_fastsurfer_analysis/subjectX/ . Make sure the output directory is empty, to avoid overwriting existing files. 

If you do not have a GPU, you can also run our CPU-Docker with very similar commands. See [Docker/README.md](Docker/README.md) for more details.


Parameter guide:
 - Create input directory 
 - create output directory 
 - Get the path of fastsurfer license
 - Find the user id and group id using the command ```shell id```
Implemented example:
```shell
docker run --gpus all -v /home/ashrafo/docker_mri_data:/data \
                      -v /home/ashrafo/docker_output:/output \
                      -v /groups/ag-reuter/software-centos/fs732:/fs_license \
                      --rm --user 6899:1001 deepmi/fastsurfer:latest \
                      --fs_license /fs_license/.license \
                      --t1 /data/bert/001.mgz \
                      --sid bert --sd /output \
                      --parallel
```
### After getting the output in output directory run freeview
```shell 
export FREESURFER_HOME=/path/to/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh
freeview /home/user/outputdir/subjectx/mri/aparc.DKTatlas+aseg.deep.mgz /home/user/outputdir/subjectx/mri/norm.mgz 
```


### Parameters/Required arguments
* `--sd`: Output directory \$SUBJECTS_DIR (equivalent to FreeSurfer setup --> $SUBJECTS_DIR/sid/mri; $SUBJECTS_DIR/sid/surf ... will be created).
* `--sid`: Subject ID for directory inside \$SUBJECTS_DIR to be created ($SUBJECTS_DIR/sid/...)
* `--t1`: T1 full head input (not bias corrected, global path). The network was trained with conformed images (UCHAR, 256x256x256, 1-0.7 mm voxels and standard slice orientation). These specifications are checked in the run_prediction.py script and the image is automatically conformed if it does not comply.

### Required for docker when running surface module
* `--fs_license`: Path to FreeSurfer license key file. Register (for free) at https://surfer.nmr.mgh.harvard.edu/registration.html to obtain it if you do not have FreeSurfer installed so far. Strictly necessary if you use Docker, optional for local install (your local FreeSurfer license will automatically be used). The license file is usually located in $FREESURFER_HOME/license.txt or $FREESURFER_HOME/.license .

### Segmentation pipeline arguments (optional)
* `--seg_only`: only run FastSurferCNN (generate segmentation, do not run the surface pipeline)
* `--main_segfile`: Global path with filename of segmentation (where and under which name to store it). The file can be in mgz, nii, or nii.gz format. Default location: $SUBJECTS_DIR/$sid/mri/aparc.DKTatlas+aseg.deep.mgz
* `--seg_log`: Name and location for the log-file for the segmentation (FastSurferCNN). Default: $SUBJECTS_DIR/$sid/scripts/deep-seg.log
* `--viewagg_device`: Define where the view aggregation should be run on. Can be "auto" or a device (see --device). By default, the program checks if you have enough memory to run the view aggregation on the gpu. The total memory is considered for this decision. If this fails, or you actively overwrote the check with setting with "cpu" view agg is run on the cpu. Equivalently, if you pass a different device, view agg will be run on that device (no memory check will be done).
* `--device`: Select device for NN segmentation (_auto_, _cpu_, _cuda_, _cuda:<device_num>_), where cuda means Nvidia GPU, you can select which one e.g. "cuda:1". Default: "auto", check GPU and then CPU
* `--batch`: Batch size for inference. Default: 1. 
* `--vol_segstats`: Additionally return volume-based aparc.DKTatlas+aseg statistics for DL-based segmentation (does not  require surfaces). Can be used in combination with `--seg_only` in which case recon-surf only runs till CC is added.
* `--aparc_aseg_segfile <filename>`: Name of the segmentation file, which includes the aparc+DKTatlas-aseg segmentations. If not provided, this intermediate DL-based segmentation will not be stored, but only the merged segmentation will be stored (see --main_segfile <filename>). Requires an ABSOLUTE Path! Default location: \$SUBJECTS_DIR/\$sid/mri/aparc.DKTatlas+aseg.deep.mgz

### Surface pipeline arguments (optional)
* `--surf_only`: only run the surface pipeline recon_surf. The segmentation created by FastSurferCNN must already exist in this case.
* `--fstess`: Use mri_tesselate instead of marching cube (default) for surface creation
* `--fsqsphere`: Use FreeSurfer default instead of novel spectral spherical projection for qsphere
* `--fsaparc`: Use FS aparc segmentations in addition to DL prediction (slower in this case and usually the mapped ones from the DL prediction are fine)
* `--parallel`: Run both hemispheres in parallel
* `--threads`: Set openMP and ITK threads to <int>
* `--no_surfreg`: Skip surface registration with FreeSurfer (if only stats are needed), for the sake of speed. 
* `--no_fs_T1`: Do not generate T1.mgz (normalized nu.mgz included in standard FreeSurfer output) and create brainmask.mgz directly from norm.mgz instead. Saves 1:30 min.

### Other
* `--vox_size <0.7-1|min>`  Forces processing at a specific voxel size.
                            If a number between 0.7 and 1 is specified (below
                            is experimental) the T1w image is conformed to
                            that voxel size and processed.
                            If "min" is specified (default), the voxel size is
                            read from the size of the minimal voxel size
                            (smallest per-direction voxel size) in the T1w
                            image:
                              If the minimal voxel size is bigger than 0.98mm,
                                the image is conformed to 1mm isometric.
                              If the minimal voxel size is smaller or equal to
                                0.98mm, the T1w image will be conformed to
                                isometric voxels of that voxel size.
                            The voxel size (whether set manually or derived)
                            determines whether the surfaces are processed with
                            highres options (below 1mm) or not.
* `--py`: Command for python, used in both pipelines. Default: python3.8
* `--conformed_name`: Name of the file in which the conformed input image will be saved. Default location: \$SUBJECTS_DIR/\$sid/mri/orig.mgz
* `--ignore_fs_version`: Switch on to avoid check for FreeSurfer version. Program will terminate if the supported version (see recon-surf.sh) is not sourced. Can be used for testing dev versions.
* `-h`, `--help`: Prints help text


### Example: Run Freesurfer on docker
    
    

