
# Table of Contents

1.  [LINC](#org5e33c6d)
    1.  [Installation](#orgae55e04)
    2.  [Downloading data](#org06ffdc3)
        1.  [Preparing data](#org569d651)
    3.  [Running](#orgc9784b5)
        1.  [Running LINC pipeline](#org2f6a8f6)
        2.  [Debugging](#org1e4fcc5)
    4.  [Scripts](#org4e08a46)
    5.  [Possible issues](#orgca19be5)
        1.  [Failing on `download_target_skymodel`](#org672ba6b)
        2.  [Wrong download URL](#org91848b6)
        3.  [A different wrong download URL?](#org1f93f9a)
        4.  [Failing on `target/structure_function`](#org9136479)
        5.  [OSError: [Errno 30] Read-only file system](#org9bb5285)
2.  [WSClean](#org52ae473)
    1.  [Basic Imaging Command](#org3712a72)
    2.  [msoverview](#org23cc750)
    3.  [Output](#org909ebac)
3.  [Struis](#org51380a6)
4.  [TRAP](#orgefe374e)
    1.  [Installation &#x2013; for tkp4.0 / python2.7 version](#org95dba30)
        1.  [change commands to conda create env commands](#org4153c8c)
    2.  [Setting up TRAP](#orgba80515)
        1.  [TKP PostgreSQL Login Details](#orgb0bccbb)
    3.  [Using TRAP](#org7896c3b)
5.  [PostgreSQL](#org0e05080)
    1.  [Deleting databases](#orgb610a8d)
6.  [Banana](#org72ddcbb)



<a id="org5e33c6d"></a>

# LINC

The software for processing LOFAR data.

Pipeline documents: <https://linc.readthedocs.io/en/latest/>
LOFAR LTA: <https://lta.lofar.eu>


<a id="orgae55e04"></a>

## Installation

LINC is the pipeline for processing LOFAR data.

Download LINC workflow files or use the already downloaded files on ALICE.
`git clone https://git.astron.nl/RD/LiNC.git <install dir>`
`/data/grbemission/shared/LINC/`

Next, obtaining Singularity container containing all the actual pipeline scripts. (Singularity is now called apptainer but runs effectively the same)

Installing apptainer isn&rsquo;t reqiured on Leicester HPC, as it&rsquo;s already a provided module. You don&rsquo;t need to load this before running the pipeline as this command can be included in the job submission script.
`module load apptainer`

There are two methods of running LINC - you can run cwltool which fetches the container as part of the pipeline, or get the container first and run cwltool within it. I&rsquo;ve had more success with the latter option. In this case, cwltool is part of the container so no additional installation is required. If you&rsquo;d like to try the other way, no neeed to pull the container manually, and installing cwltool is a simple pip install. The running commands are slightly different so check the LINC documents.

Pull the container from Astron or use the container already stored on ALICE.
`apptainer pull docker://astronrd/linc`
`/data/grbemission/shared/linc_latest.sif`


<a id="org06ffdc3"></a>

## Downloading data

You will first need to have an account on LOFAR LTA - inquire with Antonia or just email the LTA team yourself (if I remember correctly they&rsquo;ll have to contact Rhaana or Antonia anyway as PI&rsquo;s to add you to the unpublic projects).

Once this is done, you can go to the website > browse projects and select the cycle, and then the specific project you&rsquo;re interested in. Select show data > averaging pipeline to access the datasets. The names aren&rsquo;t so descriptive at this point, but you can usually easily identify the calibrator and target measurements. For the case of GRBs, you can select &rsquo;All Dataproducts&rsquo; and look under &rsquo;Start Time&rsquo; to get the date and hence identify which GRB it is.

Once you know what data you want, click the top checkbox to select all datasets and then click &rsquo;stage selected&rsquo; (top centre above table). You&rsquo;ll get a confirmation with filesize - click submit. You&rsquo;ll get an email confirming the data is queued which can take some hours to send. The data has to be moved from tape to disk so this step, depending on other user activity and data amount, can take some hours to a few days. Once the data is ready, you&rsquo;ll recieve an email.

Within the email there will be a .txt file, a list of paths to each measurement set that&rsquo;ll you&rsquo;ll download, copy the (contents) of the file to a .txt file in ALICE, where you want the data to be downloaded to. Before downloading you&rsquo;ll need to add credentials so the LTA knows it&rsquo;s you when you start requesting the files.

Create a file at `~/.wgetrc` and add to it, filling in your LTA details:

    username=<lta_username>
    password=<lta_password>

LTA reccomends updating the read/write permissions so other users can&rsquo;t read this password file, while I believe ALICE prevents other users looking into other people&rsquo;s homes, there&rsquo;s no harm doing it anyway:
`chmod 600 ~~/.wgetrc`

Now you are ready to download. The bottleneck is usually somewhere on ASTRON&rsquo;s end, but can just be UOL&rsquo;s wifi being busy, so this stage can take a fair amount of time. This can cause issues with sessions timing out if you do this through SSH. You could submit a long job request with low memory, but this can take time just for the job to actually start. Instead I prefer to use NoMachine (look at ALICE document pages for installation help) because these sessions don&rsquo;t time out for 3 days.

Open two terminals, one for calibrator and target, and open each to the directory in scratch where you want to download the respective data to. Then download:
`wget -ci lta_file.txt`

The c flag means that it&rsquo;ll carry on and resume from where it stopped in the case the download process is halted, the i flag just means it&rsquo;ll download files from the list within the .txt file. If you get 401 Authorization errors (more than once) then your LTA credentials aren&rsquo;t working.


<a id="org569d651"></a>

### Preparing data

Once done, the downloaded sets will be .tar files with long names. There is an ASTRON provided script to unzip and clean this up.
`python /data/grbemission/shared/linc_files/scripts/cleanupfiles.py .`

Run this when you&rsquo;re in the directory with the tar files and it&rsquo;ll clean this up to just include the observation code and measurement number.


<a id="orgc9784b5"></a>

## Running

With the data downloaded, you&rsquo;re ready to run the pipeline. The general outline at least for GRBs is to run the calibration pipeline on the calibrator data, take the output solutions and run the target pipeline on the target data with these solutions.

First create an input JSON file pointing towards all the measurement sets. At this stage you can include additional pipeline options as part of this file, details can be found in the documentation, but for a simple run nothing else is reqiured. There&rsquo;s another script provided that can do this automatically, unless you fancy manually typing out 244 filepaths.

Be careful about how you define <directory> - it should either be an absolute path to the measurement sets, or a correct relative path from where you&rsquo;ll run the job. For example if I&rsquo;m in `/scratch/grbemission/GRB210112A/`, then <directory> needs to point relatively to where the data is -> `/uncalibrated_data/calib/`. This will change how &ldquo;path&rdquo; by the script in the JSON file, and how the pipeline will find the measurement sets.

It&rsquo;s also more convenient to pipe the output of the script straight into a JSON file.

    python /data/grbemission/shared/linc_files/scripts/GetMSlist.py <directory> > input_calibrator.json
    
    # the output into the piped will produce something similar to:
    {
        "msin": [
            {"class": "Directory", "path": "/path/to/data_01.MS"},
            {"class": "Directory", "path": "/path/to/data_02.MS"},
            ...
        ]
    }

If you haven&rsquo;t separated the calib and target datasets into separate directories, that&rsquo;s ok, just be sure to include a wildcard selecting only the correct ones when you add <directory>. The first run should just be calibrator data.


<a id="org2f6a8f6"></a>

### Running LINC pipeline

An example command to run LINC is included, and then below a more full example of submitting the job to ALICE.

    apptainer exec --bind /scratch:/scratch
    <path/to/container.sif>
    cwltool \
    --outdir /path/to/cwd/outdir/ \
    --log-dir /path/to/cwd/logdir/ \
    --preserve-entire-environment \
    --parallel \
    --no-container \
    /data/grbemission/shared/linc_files/linc_latest.sif
    <path_to_cwl_files>/workflows/HBA_calibrator.cwl \
    <input.json>

Assuming this is run from the project base directory, then input.json is also stored here pointing /uncalibrated<sub>data</sub>/calib in my case. &rsquo;outdir&rsquo; is the directory to save the finished diagnostic plots and solutions (e.g. `/calibration_pipeline/outdir/`), &rsquo;logdir&rsquo; contains all the logs for each step of the pipeline (e.g. `/calibration_pipeline/logdir/`), and the other commands are for running within the singularity container. &rsquo;parallel&rsquo; allows for parallel processes to happen and significantly speeds up the pipeline.

<path<sub>to</sub><sub>container.sif</sub>> is the path to the container, either where you installed it or using the shared location:  `/data/grbemission/shared/linc_latest.sif`
<path<sub>to</sub><sub>cwl</sub><sub>files</sub>> is the path to your downloaded workflow files, alternatively if you&rsquo;re using the ones stored in ALICE already then set this to: `/data/grbemission/shared/LINC/`.
<input.json> is simply the name of your JSON file.

Be sure to include the bind statement otherwise apptainer can&rsquo;t access scratch and the pipeline will fail - if you&rsquo;re using the workflow files and container I left in /data - be sure to bind data too. This is included in the example script below already.

Depending on the size of the data files and speed of processing, each pipeline can take 3-7 hours for GRBs, closer to 7 if using debugging options (see LINC documentation).

When running on ALICE by default a whole process log file is created automatically.

1.  Full Leicester submission script

    A full example script is included below. Change the job name, memory, time, mail and account as required. I prefer to define the LINC and work directories above just so I can change thees a bit easier when I run on different datasets.
    
        #!/bin/bash
        
        #SBATCH --job-name=cal_200416a
        #SBATCH --nodes=1
        #SBATCH --cpus-per-task=16
        #SBATCH --mem=128gb
        #SBATCH --time=08:00:00
        #SBATCH --mail-type=BEGIN,END,FAIL
        #SBATCH --mail-user=ah724@leicester.ac.uk
        #SBATCH --export=None
        #SBATCH --account=grbemission
        
        # Edit job-name, mem, time and mail-user.
        
        # Your current project folder.
        export WORK_DIR=/scratch/grbemission/ah724/LOFAR_Followup/GRB200416A
        
        # Assuming a normal setup, within your work directory should be your JSON file.
        # The output directories will appear in this directory too, under /calibration_pipeline/
        
        ########################################################################################
        # You shouldn't need to make any changes below this line.
        
        # Log and output directories within project folder - probably don't change.
        export LINC_LOG_DIR=$WORKDIR/calibration_pipeline/logdir
        export LINC_OUT_DIR=$WORKDIR/calibration_pipeline/outdir
        
        module load apptainer
        
        export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
        export LINC_DIR=/data/grbemission/shared/linc_files
        
        cd $WORK_DIR
        
        # Create the output directories if they don't already exist.
        mkdir -p $LINC_LOG_DIR
        mkdir -p $LINC_OUT_DIR
        
        apptainer exec --bind /scratch:/scratch --bind /data:/data \
            $LINC_DIR/linc_latest.sif \
            cwltool \
                --outdir $LINC_OUT_DIR \
                --log-dir $LINC_LOG_DIR \
                --preserve-entire-environment \
                --parallel \
                --no-container \
                $LINC_DIR/LINC/workflows/HBA_calibrator.cwl \
                input_calib.json

2.  Target pipeline

    The process is mostly the same. The JSON file now needs to give a list of the target files. In the submission script, `HBA_calibrator.cwl` should now be `HBA_target.cwl`.
    When you create the target JSON file, you will also need to add a line to point the pipeline to the calibration solutions, as such:
    
        {
            "msin": [
                {"class": "Directory", "path": "/path/to/data_01.MS"},
                {"class": "Directory", 'path": "/path/to/data_02.MS"},
                ...
                ],
            "cal_solutions": {"class": "File", "path": "calibration_pipeline/outdir/cal_solutions.h5"}
        }


<a id="org1e4fcc5"></a>

### Debugging

Include these options for debugging.

    --debug
    --tmpdir-prefix /path/to/cwd/tempdir/
    --preserve-entire-environment
    --leave-tmpdir

Note this adds a whole lot of extra time (and I assume memory to the job). It will also generate a lot of log files within tmpdir which uses a lot of storage.


<a id="org4e08a46"></a>

## Scripts

Some useful scripts, example submission scripts etc.

`find_missing_ms.sh`
For GRB followup datasets there should be bands 000-243. On one occasion one was missing, so this is a simple script to just say which one is missing.


<a id="orgca19be5"></a>

## Possible issues

Just things that came up as potential issues for me. These may just be user error, specific problems with the datasets, or things that have since been fixed by ASTRON. Most common error will be the structure<sub>function</sub> issue.


<a id="org672ba6b"></a>

### Failing on `download_target_skymodel`


<a id="org91848b6"></a>

### Wrong download URL

Edit $LINCDIR/scripts/download<sub>skymodel</sub><sub>target.py</sub>
Line 120 - change URL `gsmv4` to `gsmv5`


<a id="org1f93f9a"></a>

### A different wrong download URL?

Finds an error 302 on normal URL with cgi, then tries and fails on same URL but acgi.
Checking the python files I can&rsquo;t find any reference to acgi - it should just true cgi x5 then fail if this isn&rsquo;t valid but instead tries cgi then acgi 5 times.
In this case, just follow the actual URL to the page. Create file target.skymodel somewhere in the directory, and add to target.json:
`"target_skymodel": {"class": "File", "path": "/path/to/target.skymodel"}`


<a id="org9136479"></a>

### Failing on `target/structure_function`

Usually due to flagged data making some or all bands band, or totally deleted.
To .json add `'make_structure_function': false`


<a id="org9bb5285"></a>

### OSError: [Errno 30] Read-only file system

The error is a bit misleading. Essentially it seems the pipeline doesn&rsquo;t like to create it&rsquo;s own folders - specifically where this is failing is the log and out directories. If you&rsquo;ve set the log directory to `/calibration_pipeline/logdir/`, then these directories should exist beforehand as the pipeline cannot create them. Same goes for outdir.


<a id="org52ae473"></a>

# WSClean

Software is included in the LINC Singularity container - this needs to be active to run WSClean.


<a id="org3712a72"></a>

## Basic Imaging Command

    wsclean \
    -mgain 0.8 \           # Cleaning parameters.
    -auto-mask 10 \
    -pol I \
    -maxuv-l 8000 \
    -auto-threshold 3 \
    -weight briggs -0.5 \
    -niter 100000 \
    -weighting-rank-filter 3.0 \
    -fit-beam \
    -reorder \
    -clean-border 0 \
    -apply-primary-beam \
    -join-channels \
    -no-update-model-required \
    -name <name> \        # User parameters.
    -channels-out 6 \
    -size 2048 2048 \
    -scale 1asec \
    *.ms | tee imaging.log

Timeslicing

    -reorder --> -no-reorder
    -intervals-out X            # Split observation into X chunks.
    -interval A B               # Only use slices A to B of the whole dataset, splitting it into X chunks.


<a id="org23cc750"></a>

## msoverview

`msoverview in=file.ms (verbose=T)`
View detailed information about the measurement sets. I believe this command is part of CASA, or in the Singularity container.


<a id="org909ebac"></a>

## Output

Produces primary beam images, dirty, model, individual beam visibilities for each outputted timeslice and per frequency channel requested.

\*-image-pb.fits are the files most interesting to us.

Example Recent Run

    wsclean \
    -mgain 0.8 \
    -auto-mask 10 \
    -pol I \
    -maxuv-l 8000 \
    -auto-threshold 3 \
    -weight-briggs -0.5 \
    -niter 100000 \
    -weighting-rank-filter 3.0 \
    -clean-border 0 \
    -fit-beam \
    -apply-primary-beam \
    -channel-division-frequencies 1.37e8,1.6e8 \
    -channels-out 1
    -reorder \
    -update-model-required \
    -name midf_wholetime_noslice \
    -size 2048 2048
    -scale 1asec \
    *.ms


<a id="org51380a6"></a>

# Struis

Struis - Amsterdam HPC system. You&rsquo;ll need to acquire login details for this.
`ssh <username>@struis.science.uva.nl`

[Login details](login-info.md)


<a id="orgefe374e"></a>

# TRAP

Software for analysing LOFAR data.

For Python3 - setup is easier (I&rsquo;ve saved the word to Documents somewhere but requires asking antonia for a python3 database I believe)


<a id="org95dba30"></a>

## Installation &#x2013; for tkp4.0 / python2.7 version

Clone latest version from Github.
`git clone https://github.com/transientskp/tkp.git`

Create a virtualenv if you don&rsquo;t already have one and source it. TraP 5.0 runs on Python2 still so ensure virtualenv is setup accordingly.

    virtualenv trap_env_2023 --python=python2.7
    
    
    conda activate

Install Jupyter notebook.

    pip install --upgrade pip
    pip install notebook

Install boost

    conda install -c conda-forge boost

**Changing TraP version**
Install tkp with developer mode with the &rsquo;-e&rsquo; tag, meaning we can use the Git checkout feature.

    cd ~/tkp
    pip install -e ".[pixelstore]"
    git tag             # Shows all available tags.
    git checkout r5.0   # 5.0 is the python 2.7 version I think current banana uses


<a id="org4153c8c"></a>

### TODO change commands to conda create env commands


<a id="orgba80515"></a>

## Setting up TRAP

Ensure you&rsquo;re the virtual environment you setup.

    # initialise a new project (only do once?)
    trap-manage.py initproject promptradio
    
    # create a new database
    creatdb -h vlo.science.uva.nl -U ahennessey <databasename>
    <postgresql password>
    
    # edit pipeline.cfg
    
    # initialise the databse with tkp
    trap-manage.py initdb


<a id="orgb0bccbb"></a>

### TKP PostgreSQL Login Details

`ahennessey`
`5tF69ShycX`


<a id="org7896c3b"></a>

## Using TRAP

\#+begin<sub>src</sub> shell

trap-manage.py initjob <jobname>

./<jobnames>/job<sub>params.cfg</sub> # job parameters
./<jobnames>/images<sub>to</sub><sub>process.py</sub> # point to image files

trap-manage.py run <jobname>

trap-manage.py run [-m MONITOR<sub>COORDS</sub>] [-l COORDS<sub>FILE</sub>] <jobname>

nohup trap-manage.py run <jobname> > trap<sub>output.log</sub> &
tail -f trap<sub>output.log</sub>
\#end<sub>src</sub>


<a id="org0e05080"></a>

# PostgreSQL

General information and commands for PostgreSQL can be found at: [software.org/postgresql](software.md)

LOFAR specific useful commands:

    # access psql terminal
    psql -U ahennessey -h vlo.science.uva.nl -d <databasename>
    
    # export database table to csv file
    \copy extractedsource TO '/scratch/ahennessey/extract_240414a.csv' CSV HEADER;


<a id="orgb610a8d"></a>

## Deleting databases

Access the database as above, then after using `\dt` to list tables, you can use `DELETE FROM table;` to remove each table. You will find a series of linked foreign keys, just delete the table that it&rsquo;s referencing from one by one until all is gone.

[Foreign key issues!](file:///Users/ah724/org/guides/software.md)


<a id="org72ddcbb"></a>

# Banana

Used for viewing the ran files from TRAP. This doesn&rsquo;t automatically work for newest LINC pipelines now due to using Python 3. Instead you must manually use SQL to download as csv files and process yourself.

[Login details](login-info.md)

