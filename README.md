# Onboarding new lab members
Introductory material and some tips and good practices for new lab members.

## Table of Contents
- [UBELIX HPC](#ubelix-hpc)
    - [Basic usage](#basic-usage)
    - [Working space](#working-space)
    - [Port forwarding](#port-forwarding)
- [GitHub](#github)

<br/>

## UBELIX HPC
### Basic usage
* If you don't have a UBELIX account yet, [create one](https://hpc-unibe-ch.github.io/firststeps/accessUBELIX/).
* Let us know your user name so that we can add it `dbmr_luisierlab` group.
* If you have never worked on an HPC or SLURM, check [these basic tips](https://docs.google.com/document/d/1T1WE07CCaQuTME6duUjRsqCtY_z3mvwrzndhF4DK49g/edit?usp=sharing).
> [!IMPORTANT]
> Read the entire [UBELIX manual](https://hpc-unibe-ch.github.io/) before starting.

### Working space
* Make a project folder in `/storage/research/dbmr_luisierlab/projects/`. All your work should be there, including code, sessions, and results.
* All computation should be done on UBELIX. Avoid working locally! You can run a Jupyter notebook or RStudio via [UBELIX OnDemand](https://hpc-unibe-ch.github.io/runjobs/ondemand/), use port forwarding (see below).
* Use 'gratis' or 'preemptable' QOS when possible.
* Keep your project space tidy.
* Be mindful of disk quotas; they are limited! This includes total storage size, AND the number of files. Never store large uncompressed text files (e.g., FASTA/FASTQ, annotations, or other large text files). Most libraries and tools can natively import and export compressed files. Folders containing hundreds or thousands of files should be tarballed when not used anymore (if the files in the folder are already compressed, make a `.tar`, otherwise make a `.tar.gz`).
* Remember, one can use `scratch` for temporary large files at `/storage/scratch/dbmr_luisierlab/` (data get automatically deleted after 30 days).
* Use `$TMPDDIR` as much as possible to avoid I/O bottlenecks and disk stress (especially if the job creates a lot of temporary files). [Read more](https://hpc-unibe-ch.github.io/storage/scratch/).

### Port forwarding
You might want to launch an interactive visaulisation tool (Jupyter, RStudio, etc.) on the compute node, and access it on your localhost. For this you will need to configure port forwarding. See [this section](https://hpc-unibe-ch.github.io/software/packages/JupyterLab/#setup-ssh-with-port-forwarding) of the UBELIX manual.
> [!NOTE]
> * If you are accessing UBELIX via VS Code, or MobaXterm, you can also configure port forwarding via their UI.
> * To be on the safe side, avoid using default port numbers (there might be unexpected errors in a rare event where multiple users run the same tool from the same compute node on the same port). Tip: You can use `$UID` as port number (make sure it's in the range 2000-65000; if your UID has 6 digits use something like ${UID:0:5}).
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

> In summary, your project's repo should contain everything to reproduce your work. Anyone with access to our HPC should be able to recreate all your results!
