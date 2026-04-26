# Revised installation notes for this fork

This fork keeps the MPRAflow workflow and Python/R source code unchanged, but replaces the old exported Conda environments with smaller, solver-friendly environment specifications.

## What changed

- `environment.yml` now installs the runner environment only: `nextflow=20.10.0` and `openjdk=11`.
- `conf/mpraflow_py27.yml` now contains only the Python 2.7 runtime and tools actually needed by the legacy Python-2 MPRAflow scripts: `pysam`, `samtools`, `bcftools`, and `htslib`.
- `conf/mpraflow_py36.yml` keeps the original filename because the `.nf` workflows reference that path, but it now uses Python 3.9. The scripts that use this file are Python-3-compatible; the old Python 3.6 stack was unnecessary.
- `conf/mpraflow_r.yml` now uses a modern R 4.x stack with the packages imported by the R scripts: `dplyr`, `ggplot2`, `tidyr`, `stringr`, plus Cairo support for PNG output.
- `nextflow.config` now enables Conda by default with `conda.enabled = true` and gives environment creation a longer timeout.

No `.nf` workflow files and no `.py` or `.R` source files were changed.

## Recommended setup

Use a recent Miniforge/Miniconda installation with the libmamba solver or `mamba`. MPRAflow is a Linux-oriented Bioconda workflow; Windows is not supported.

Configure channels once:

```bash
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
```

Create and activate the runner environment:

```bash
cd MPRAflow-installable
mamba env create -n MPRAflow -f environment.yml
# or: conda env create -n MPRAflow -f environment.yml
conda activate MPRAflow
nextflow -version
```

Expected Nextflow version: `20.10.0`.

## Running

The process-specific environments are referenced from the workflow files and are created by Nextflow automatically from:

- `conf/mpraflow_py27.yml`
- `conf/mpraflow_py36.yml`
- `conf/mpraflow_r.yml`

The default config enables Conda automatically. If your site configuration overrides this, add `-with-conda` to the command.

Examples:

```bash
conda activate MPRAflow
nextflow run association.nf --help
nextflow run count.nf --help
nextflow run association_saturationMutagenesis.nf --help
nextflow run saturationMutagenesis.nf --help
```

For a real run, use the same MPRAflow commands as before, for example:

```bash
nextflow run count.nf \
  --dir "bulk_FASTQ_directory" \
  --e "experiment.csv" \
  --design "ordered_candidate_sequences.fa" \
  --association "dictionary_of_candidate_sequences_to_barcodes.p"
```

## Important cleanup after previous failed installs

If you previously ran MPRAflow with the old YAMLs, remove the failed Nextflow Conda cache before retrying:

```bash
rm -rf work/conda
```

If you use a shared Nextflow Conda cache, remove only the MPRAflow environments from that cache, or start with a fresh work directory.

## Optional: validate the YAMLs directly

These commands are useful for debugging a site-wide Conda problem. They are not required for normal MPRAflow execution, because Nextflow creates hashed process environments from the same YAMLs.

```bash
mamba env create -f conf/mpraflow_py27.yml
mamba env create -f conf/mpraflow_py36.yml
mamba env create -f conf/mpraflow_r.yml
```

If these succeed but Nextflow still fails, check that the `conda` executable is visible inside the activated `MPRAflow` runner environment:

```bash
which conda
conda info
```

## Dependency map used to simplify the YAMLs

| Environment file | Used for | Direct tools/imports found in the repo |
| --- | --- | --- |
| `environment.yml` | launching workflows | `nextflow`, Java runtime |
| `conf/mpraflow_py27.yml` | legacy read/BAM/variant scripts | `python=2.7`, `pysam`, `samtools`, `bcftools` |
| `conf/mpraflow_py36.yml` | association/count Python-3 scripts and mapping/QC tools | `python`, `pandas`, `numpy`, `dask`, `pyarrow`, `biopython`, `pysam`, `click`, `matplotlib`, `seaborn`, `tqdm`, `bwa`, `samtools`, `fastq-join`, `fastqc`, `picard` |
| `conf/mpraflow_r.yml` | count/saturation-mutagenesis plotting and modeling | `Rscript`, `dplyr`, `ggplot2`, `tidyr`, `stringr`, Cairo PNG support |
