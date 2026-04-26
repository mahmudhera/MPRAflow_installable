.. _Installation:

=====================
Installation
=====================

Installation should take less than 10 minutes for the runner environment. The
first real workflow run will also create the process-specific Conda environments.

System Requirements
===================

MPRAflow is a Linux-oriented Bioconda/Nextflow workflow. Windows is not
supported. Use a recent Miniforge/Miniconda installation with Mamba or Conda's
libmamba solver when possible.

Required packages
=================

.. code-block:: bash

   conda
   mamba  # recommended, optional

Set up channels
===============

Configure Bioconda-compatible channel priority once:

.. code-block:: bash

   conda config --add channels bioconda
   conda config --add channels conda-forge
   conda config --set channel_priority strict

Clone repository
================

.. code-block:: bash

    git clone https://github.com/shendurelab/MPRAflow.git
    cd MPRAflow

For this installable fork, use the modified repository directory instead.

Set up the runner environment
=============================

The runner environment is called ``MPRAflow`` and contains Nextflow 20.10.0 and
Java 11. Nextflow 20.10.0 matches ``conf/global.config`` and is intentionally
used instead of the latest Nextflow release.

.. code-block:: bash

    mamba env create -n MPRAflow -f environment.yml
    # or: conda env create -n MPRAflow -f environment.yml
    conda activate MPRAflow
    nextflow -version

Process environments
====================

The workflow files reference three process environment YAMLs in ``conf/``:

* ``conf/mpraflow_py27.yml`` for the legacy Python 2.7 read/BAM/variant scripts.
* ``conf/mpraflow_py36.yml`` for Python-3-compatible scripts and command-line
  bioinformatics tools. The filename is kept for workflow compatibility, but the
  environment now uses Python 3.9.
* ``conf/mpraflow_r.yml`` for the R plotting/modeling scripts.

Nextflow creates these process-specific environments automatically. This fork
sets ``conda.enabled = true`` in ``nextflow.config``. If your site configuration
overrides that setting, pass ``-with-conda`` to ``nextflow run``.

Quick test
==========

.. code-block:: bash

    conda activate MPRAflow
    nextflow run count.nf --help
    nextflow run association.nf --help
    nextflow run association_saturationMutagenesis.nf --help
    nextflow run saturationMutagenesis.nf --help

Cleanup after failed old installs
=================================

If you previously ran MPRAflow with the old fully-pinned YAMLs, remove the failed
Nextflow Conda cache before retrying:

.. code-block:: bash

    rm -rf work/conda

Optional YAML validation
========================

These commands are only for debugging Conda availability on a system. Normal
workflow execution lets Nextflow create equivalent hashed environments.

.. code-block:: bash

    mamba env create -f conf/mpraflow_py27.yml
    mamba env create -f conf/mpraflow_py36.yml
    mamba env create -f conf/mpraflow_r.yml
