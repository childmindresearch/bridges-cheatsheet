# Bridges cheat sheet

## Environment variables

|Variable|Meaning|
|--------|-------|
|`$PROJECT`|`/ocean/projects/{med_project_number}/{username}`|
|`$LOCAL`|Local storage in current node - temporary but much faster|

## Simple commands

### Interactive session

```bash
interact -p RM-shared -N 1 --ntasks-per-node=10 -t 03:00:00
```

### List current jobs

```bash
squeue --me
```

### Live print log file as it gets updated

```bash
tail -f my_out_file.log
```

### Kill all your jobs

```bash
scancel -u $USER
```

## Singularity/Apptainer

### `.bashrc` additions

```bash
# Singularity cache
export SINGULARITY_CACHEDIR=$PROJECT/.singularity/cache
export SINGULARITY_LOCALCACHEDIR=$PROJECT/.singularity/tmp
export SINGULARITY_TMPDIR=$LOCAL

export APPTAINER_CACHEDIR=$SINGULARITY_CACHEDIR
export APPTAINER_LOCALCACHEDIR=$SINGULARITY_LOCALCACHEDIR
export APPTAINER_TMPDIR=$SINGULARITY_TMPDIR
```

- Avoids overflowing home directory
- Uses local storage for temporary files - much faster image builds

### Build C-PAC nightly container

```bash
singularity build $PROJECT/images/nightly.sif docker://ghcr.io/fcp-indi/c-pac:nightly
```

## Anaconda

```bash
module load anaconda3/2022.10
conda create -n myenvironment python=3.11
conda activate myenvironment
```

(List available modules with `module av`)

### Install / update github hosted Python package

```bash
pip uninstall clmunch && sleep 2 && pip install git+https://github.com/cmi-dair/cpac-log-muncher
```

- For some reason `pip install -U ...` does not always work.

## C-PAC job file template 

```bash
#!/usr/bin/bash
#SBATCH --job-name my_job_name
#SBATCH --output my_job_out.log
#SBATCH --nodes 1
#SBATCH --partition RM-shared
#SBATCH --time 48:00:00
#SBATCH --ntasks-per-node 16

set -x

cd /my/job/folder

singularity run --cleanenv \
-B "$PROJECT"/input:"$PROJECT"/input:ro \
-B "$PROJECT"/output:"$PROJECT"/output \
"$PROJECT"/images/nightly.sif \
"$PROJECT"/input \
"$PROJECT"/output \
participant \
--skip_bids_validator \
--n_cpus 15 \
--mem_gb 31.0 \
--participant_label sub-123 \
--preconfig abcd-options \
--save_working_dir "$PROJECT"/my/job/folder 
```

- Memory is always 2GB / task

### Overwrite image C-PAC version with local development version

```bash
-B {cpac_dir}/CPAC:/code/CPAC \
-B {cpac_dir}/dev/docker_data/run.py:/code/run.py \
-B {cpac_dir}/dev/docker_data:/cpac_resources \
```

## Utilities
  
- Easier way to launch C-PAC on ACCESS: https://github.com/cmi-dair/ecpac
- Auto search and aggregate C-PAC run information: https://github.com/cmi-dair/cpac-log-muncher
- Subset large BIDS datasets (by symlinking) so C-PAC does not choke: https://github.com/cmi-dair/bids-subset
- View nifti files in the terminal / without GUI: https://github.com/cmi-dair/headjack

## Links

- Bridges user guide https://www.psc.edu/resources/bridges-2/user-guide/
- Connect to bridges in the browser: https://ondemand.bridges2.psc.edu
