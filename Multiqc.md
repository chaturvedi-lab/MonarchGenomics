These are my notes for the quality check of the genomic data generated for the Monarch project using the UC Davis Sequencing Core in April-June 2023 at UC Berkeley. I am using MultiQC to assess the qualtiy of the sequences. 

# 1. Installation
I installed MultiQC on the cluster using Conda. Here are my commands for the installation.

```bash

module load python
conda create -n multiqc_env python=3.8.8
conda activate multiqc_env
conda install -c bioconda multiqc

# I tested the installation using the following command and I got the right output pasted below

$ multiqc
Usage: multiqc [OPTIONS] [ANALYSIS DIRECTORY]

 This is MultiQC v1.15
 For more help, run 'multiqc --help' or visit http://multiqc.info
```

# 2. Running multiQC on the data
