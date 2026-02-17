# Onboarding new lab members
Introductory material and some tips and good practices for new lab members.

## Table of Contents
- [UBELIX HPC](#ubelix-hpc)
    - [Basic usage](#basic-usage)
    - [Working space](#working-space)
    - [Submitting jobs](#submitting-jobs)
    - [Environments](#environments)
    - [Port forwarding](#port-forwarding)
- [GitHub](#github)
   - [R Notebooks on GitHub](#r-notebooks-on-github)
<br/>

## UBELIX HPC
### Basic usage
* If you don't have a UBELIX account yet, [create one](https://hpc-unibe-ch.github.io/firststeps/accessUBELIX/).
* Let us know your user name so that we can add it to the `dbmr_luisierlab` group.
* If you have never worked on an HPC or SLURM, check [these basic tips](https://docs.google.com/document/d/1T1WE07CCaQuTME6duUjRsqCtY_z3mvwrzndhF4DK49g/edit?usp=sharing).
> [!IMPORTANT]
> Read the entire [UBELIX manual](https://hpc-unibe-ch.github.io/) before starting.

### Working space
* Make a project folder in `/storage/research/dbmr_luisierlab/projects/`. All your work should be there, including code, sessions, and results.
* All computation should be done on UBELIX. Avoid working locally! You can run a Jupyter notebook or RStudio via [UBELIX OnDemand](https://hpc-unibe-ch.github.io/runjobs/ondemand/), or use port forwarding (see below).
* Keep your project space tidy.
* Be mindful of disk quotas; they are limited! This includes total storage size, AND the number of files. Never store large uncompressed text files (e.g., FASTA/FASTQ, annotations, or other large text files). Most libraries and tools can natively import and export compressed files. Folders containing hundreds or thousands of files should be tarballed when not used anymore (if the files in the folder are already compressed, make a `.tar`, otherwise make a `.tar.gz`).
* Remember, one can use `scratch` for temporary large files at `/storage/scratch/dbmr_luisierlab/` (data get automatically deleted after 30 days).
* Use `$TMPDDIR` as much as possible to avoid I/O bottlenecks and disk stress (especially if the job creates a lot of temporary files). [Read more](https://hpc-unibe-ch.github.io/storage/scratch/).

### Submitting jobs
[Here](https://hpc-unibe-ch.github.io/runjobs/partitions/) you can find the list of partitions and QoS on UBELIX. Basically, there are three options we can use:
1. `gratis` (limited resources)
2. `preemptable` (6h max)
3. `paygo`

Use the `gratis` and `preemptable` QoS if it is reasonably feasible. For instance, say you need to process several samples, where each sample does not take very long to finish. If it's something you would do only occasionally (e.g., mapping), you could use `job_gratis` and submit the jobs in the evening; the results will be ready the next morning. If this is not feasible, say because there are too many samples to process, you could use `job_cpu_preemptable` (knowing that the 6h max allocation will be enough for each job).
> [!NOTE]
> *preemptable* jobs may be terminated by paygo or investor jobs at any time! In our experience, this rarely happens, but if it does, the job will be automatically rescheduled.
> Make sure to parallelise, i.e., submit a separate job for each sample. *preemptable* jobs are convenient for many short jobs that can be submitted in parallel.

#### paygo
If you need more resources than *gratis*, and *preemptable* QoS is not convenient, you need to request a `paygo` job. For this, you need to add these two options to your sbatch script:
```
#SBATCH --account paygo 
#SBATCH --wckey dbmr_luisierlab
```
> [!NOTE]
> * Only registered users can use wckey *dbmr_luisierlab* (make sure your user was added the *dbmr_luisierlab* group).
> * As for any sbatch option, `--account` and `--wckey` can also be passed to `srun` and `salloc`.

### Environments
Bioinformaticians often rely on tools that require complex environments (dependencies, etc.). The most common strategies to handle environments on an HPC are:
1. Conda
2. Containers

#### Conda
On UBELIX load the Anaconda3 module: `module load Anaconda3`.
See [here](https://hpc-unibe-ch.github.io/software/installing/python/#anaconda-conda) for basic conda usage on UBELIX.
> [!WARNING]
> Please do not run `conda init`. Instead, initialise the conda environment using:
> ```
> module load Anaconda3
> eval "$(conda shell.bash hook)"
> ```
> This will prevent conda from altering your `.bashrc` file and avoid loading the *base* environment at every login.

> [!TIP]
> * Create separate environments for each pipeline, subproject, or tool, etc.
> * Simpler environments (with fewer tools and dependencies) are easier to install and maintain, and most importantly, reproduce in the future.
> * Alternative to `eval "$(conda shell.bash hook)"` is `source activate ...`
> ```
> module load Anaconda3
> source activate myenv
> ```

> [!NOTE]
> Building conda environments can take substantial CPU and memory resources. Use interactive sessions!

#### Containers
The main disadvantage of conda environments is that they are not portable. When moving to another machine, your only option is rebuilding an environment.
In some cases, at one point, it might not be possible to reproduce a conda environment because some libraries and specific versions are not available in the conda channels anymore, or dependency trees change, or some system libraries differ, etc.
Containers are the best way to ensure portability and reproducibility. In general, if you can build a conda environment, you can build a container.

Nowadays, many bioinformatic tools are containerised (usually as Docker images), and you only need to "pull" them. 
See [this section](https://hpc-unibe-ch.github.io/software/containers/apptainer/) for a basic Apptainer/Singularity usage on UBELIX.

> [!TIP]
> If you need to build a singularity container that relies on installing most of the libraries via conda, you can build your image starting from a Miniconda docker image (where conda is preinstalled), like in this definition file:
> ```
> Bootstrap: docker
> From: continuumio/miniconda3:24.11.1-0
>
> %post
> conda install --yes -q conda-forge::foo==1.0.0 conda-forge::bar==2.0.0
> ```

> [!NOTE]
> For reproducibility and transparency, keep the definition file (`.def`) and the build log file (`.log`) along with the container file (`.sif`). For instance:
> ```
> singularity build --no-root mycontainer_v1.0.sif mycontainer_v1.0.def 2>&1 | tee mycontainer_v1.0.log
> ```
> Building containers can take substantial CPU and memory resources. Use interactive sessions!

### Port forwarding
If you need to launch an interactive visualisation tool (Jupyter, RStudio, etc.) on the compute node and access it on your localhost, you will need to configure port forwarding. See [this section](https://hpc-unibe-ch.github.io/software/packages/JupyterLab/#setup-ssh-with-port-forwarding) of the UBELIX manual.
> [!NOTE]
> If you are accessing UBELIX via VS Code, or MobaXterm, you can also configure port forwarding via their UI.

> [!TIP]
> To be on the safe side, avoid using default port numbers (there might be unexpected errors in the rare event where multiple users run the same tool from the same compute node on the same port).
> Instead, you can use your `$UID` as port number (make sure it's in the range 2000-65000; if your UID has 6 digits, use something like `${UID:0:5}`).
#### Examples
##### 1. Run RStudio from a Singularity image
Say you want to run RStudio from this container `/storage/research/dbmr_luisierlab/resources/img/R_RStudio/Rocker_rstudio_4.5.1/Rocker_rstudio_4.5.1.sif`.

Launch an interactive session from the login node:
```
srun --mem 20G --pty bash
```
Run rserver:
```
CONTAINER=/storage/research/dbmr_rubin_lab/containers/Rocker_rstudio/Rocker_rstudio_$VER/Rocker_rstudio_$VER.sif
PORT=${UID:0:5}

echo -e "$CONTAINER will be running at $HOSTNAME:$PORT"
echo -e "To forward the port to your local machine, run the following command on your local machine: \n
ssh -N -f -L $PORT:$HOSTNAME:$PORT $(whoami)@${SLURM_SUBMIT_HOST}.unibe.ch"
echo -e "Once forwarded, go to your browser and type localhost:$PORT"

SINGULARITYENV_PASSWORD=$(whoami) singularity exec \
   --bind run:/run,var-lib-rstudio-server:/var/lib/rstudio-server,database.conf:/etc/rstudio/database.conf,/storage/research/dbmr_luisierlab \
   $CONTAINER rserver \
   --www-port $PORT --server-user=$(whoami) \
   --auth-timeout-minutes=0 --auth-stay-signed-in-days=30 --auth-none=0 --auth-pam-helper-path=pam-helper # other rstudio options
```
> [!NOTE]
> * `$HOSTNAME` in this case is the compute node where the interactive session is running. `${SLURM_SUBMIT_HOST}` is a special variable that holds the login node name (here for convenience).
> * In this example we run `rstudio` from a container, but you would run it in the same way from a conda evironment or an HPC module. The main option here is `--www-port $PORT`.

## GitHub
* Get familiar with `git` and its basic usage. In most cases, each project's repo will be contributed to by one person. Therefore, the student will generally need basic git commands like `status`, `add`, `commit`, and `push`.
* Every project's repo **must** contain a **detailed** README that will summarise the project, and describe the content of the repo.
* The code should be stored in folders, organised by sub-projects or tasks if necessary.
* Main code should be stored as Python notebooks (`.ipynb`) or R Markdown files (`.Rmd`). Other scripts and auxiliary code can be kept as text files.
* Results (figures, tables, etc.) should be organised in folders.
* Dependencies, environments, and resources should be documented.
* Don't deposit large files on GitHub, but you should document them (instructions to download, location on the HPC, etc.).
* See [this](https://github.com/RLuisier/ALSdisMNs/tree/master), [this](https://github.com/RLuisier/AstroShape), and [this](https://github.com/RLuisier/my3UTRs) example of basic projects' repos, or [this](https://github.com/RLuisier/AxonLoc) for a more complex one.

> In summary, your repo should contain everything to reproduce your work. Anyone with access to our HPC should be able to recreate all your results!

#### R Notebooks on GitHub
At the moment, GitHub does not natively render R Markdown files (`.Rmd`), as it does for Jupyter notebooks (`.ipynb`). HTML files will also not be rendered.
As a workaround, one can convert an R Notebook (`.nb.html`) to a PDF to visualise it on GitHub.
> [!NOTE]
> A PDF notebook is only a visualisation convenience. Always upload the equivalent `.Rmd` file for reproducibility!

##### R Notebook vs Markdown
R Notebook is a newer RStudio feature that behaves similarly to Jupyter Notebook. The biggest difference is that R Notebook has a live preview of your code (and the outputs you choose to show), which is automatically saved as an HTML document, which you don't have to `knit`.
The practical advantage of Notebook's preview is that it does not need to execute all the code from the start to render it. You can run individual chunks of code as if you were running them in a terminal (they will behave depending on the objects loaded in your session).
Furthermore, if you resume working, all the Notebook's rendering from the previous session will be preserved! You don't need to rerun your code to generate plots, etc. 
> [!WARNING]
> * ***Knitting*** will always run the entire code from the beginning! This is not convenient when working with big data because of the heavy computation that comes with it.
> * Exporting an R Notebook to a PDF or a Word document in RStudio triggers knitting, which executes the entire code from the start of the notebook!

An alternative to knitting PDF notebooks is to convert an HTML notebook into a PDF using external tools:
1. Open the HTML notebook in your browser and export/print to PDF.
2. Convert HTML to PDF on the command line using a tool of your choice.
> [!NOTE]
> For convenience, there is a Chromium container on UBELIX that can be used like this:
> ```
> singularity run /storage/research/dbmr_luisierlab/resources/img/chromium/chromium_124.sif --no-pdf-header-footer --print-to-pdf=mynotebook.pdf mynotebook.nb.html
> ```

