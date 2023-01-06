# Nextstrain on Midway

Nextstrain requires Auspice, Docker/Singularity, raxml, fasttree, and vcftools for some jobs. These are not supported on Midway (Docker) or are difficult to install (node-js for Auspice). However, the actual dependencies for particular jobs will depend on the contents of the Snakemake file.

How well the process scales will also depend on the Snakemake file. If each job  depends on input from the last job, it may not scale at all. (Adding more nodes will not make the process faster.)

This example only requires Python 3+, Snakemake, mafft, and iqtree so compatibility with Midway is much easier than it would be otherwise.

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

Note: ~/bin is a standard binary directory for Midway. `.bashrc` should already be adding it to your PATH. If you use Z shell, you may need to copy the .bashrc lines over.

6. Run snakemake on a Nextstrain job. Parameters are for a small job in the pi-cobey account (add `-p cobey` to use the private nodes).

```bash
cd <directory with a Snakemake file>
snakemake --cluster "sbatch -A pi-cobey -t 20:00 -N 3 -J nextstrain_snakemake" --jobs 3
```

For long-running jobs, you can run snakemake in the background:

```bash
export AUGUR_RECURSION_LIMIT=2000
spack load python mafft
# change time, nodes etc to fit size of job.
nohup snakemake --cluster "sbatch -A pi-cobey -t 20:00 -N 3 -J nextstrain_snakemake" --jobs 3 &
```
