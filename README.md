## Installing and Using Nextstrain in a Traditional HPC Environment


Traditional High Performance Computing centers are typically Unix/Linux-based, distributed, shared, Slurm environments. Users schedule resources in the form of a number of abstract "nodes" that represent independent CPU, memory and sometimes GPU units. This environment works perfectly well for many bioinformatic projects: the dependencies are older, established applications, they have minimal graphical display needs, and the lack of dedicated hardware doesn't pose a problem. For applications like [Nextstrain](https://nextstrain.org/) that depend on graphical display, a local web server, Nodejs, and Docker, a dedicated local machine or cloud hosting (Google Cloud, AWS etc) is much easier to accomodate.

If you don't have the option to install Nextstrain on a local machine, cloud host, or another dedicated environment, however, a traditional HPC environment will work just as well with some adjustments. And it comes with learning how to use Snakemake, which is a useful tool for orchestrating complex jobs particularly on a distributed HPC environment.

#### Assumptions about your environment:

  * Docker is not allowed. (Docker requires root access so many HPC centers don't allow it due to security concerns.)
  * Nodejs is less well-supported, and running a browser and local web server won't work.
  * The scheduler is Slurm. (sbatch files, `srun` etc)
  * (Optionally) Singularity is supported.

The Nextstrain suite is composed of the `nextstrain` command line interface, Augur, Auspice, and a short list of standard applications like mafft and iqtree. Nextstrain CLI and Augur are Python applications. Most centers should support specific versions of Python, which is all you need since their dependencies can be installed in a personal virtual environment.

Auspice depends on Nodejs and running a web server, which is common and well-supported on local machines or cloud services, but usually not allowed on higher security, shared environments. However, the data needed for an Auspice visualization can be transferred to another machine for viewing.

The standard bioinformatics tools such as mafft and iqtree are relatively easy to install even if your HPC doesn't provide them, which it generally will. We'll assume the worst case scenario for Python, mafft, and iqtree in case they're not available. Loading some of these with `module load python/3.6` or `module load mafft` should be equivalent if they're available.

## Installing requirements

1. Install [Spack](https://spack-tutorial.readthedocs.io/en/latest/tutorial_basics.html). This will allow you to install applications in your home directory and load them as you would with the `module` command.

_Note: You may need to add something like this to ~/.bashrc or ~/.zshrc to use spack easily._

```
# spack
PATH="$HOME/spack/bin:$PATH"
export PATH
```

2. Install and load mafft.

```bash
spack install mafft
spack load mafft
```

3. Install Python.

```bash
spack install python@3.9.0
spack load python
```

4. Create a virtual environment, activate it, and install libraries.

```bash
python -m venv nextstrain-venv
source nextstrain-venv/bin/activate
pip install nextstrain-cli nextstrain-augur snakemake
```
5. Install iqtree.

Release files are on [github](https://github.com/Cibiv/IQ-TREE/releases). This example uses 2.06.

```bash
# from home directory
cd
wget https://github.com/Cibiv/IQ-TREE/releases/download/v2.0.6/iqtree-2.0.6-Linux.tar.gz
tar xfz iqtree-2.0.6-Linux.tar.gz
mkdir bin
cp iqtree-2.0.6-Linux/bin/iqtree2 bin/
```

_Note: This assumes ~/bin is a standard binary directory and on your path. It may not be for some environments._

6. Run snakemake on a Nextstrain job. Parameters are for a small job with three nodes. The `-A` parameter will need to be modified, and may be `-P` for partition in your environment.

```bash
cd <directory with a Snakemake file>
snakemake --cluster "sbatch -A <some account> -t 20:00 -N 3 -J nextstrain_snakemake" --jobs 3
```

### Non-interactive Jobs

For long-running jobs that are easier to submit with sbatch, a node needs to be scheduled to run Snakemake, which will then spawn new jobs. If you have a job that will run by submitting all jobs at once with the `--immediate-submit` flag, then this step is unnecessary.

#### Example sbatch file:


```bash
#!/bin/bash
#SBATCH --job-name=nextstrain
#SBATCH --output=%x_%A.out
#SBATCH --error=%x_%A.err
#SBATCH --time=00:20:00
#SBATCH --nodes=6
#SBATCH --mem=2G
#SBATCH --mail-user=youruser@youremailaddress.edu
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --account=<some account or partition with --partition>


# load any dependencies, the virtual environment, variables etc here.
spack load python mafft
source ~/nextstrain-venv/bin/activate
export AUGUR_RECURSION_LIMIT=2000

snakemake --cluster "sbatch -A <some account> -t 20:00 -N 3 -J nextstrain_snakemake" --jobs 3
```
