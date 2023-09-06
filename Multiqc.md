These are my notes for the quality check of the genomic data generated for the Monarch project using the UC Davis Sequencing Core in April-June 2023 at UC Berkeley. I am using MultiQC to assess the qualtiy of the sequences. 

# 1. Fastp for adapter trimming
I did adapter trimming for the raw data using Fastp (https://github.com/OpenGene/fastp). Fastp uses a custom algorithms and does not neccessarily require us to specify adapter sequences. I installed fastp on the cluster using conda.

## Installation using conda

```bash

module load python
conda create -n fastp_env python=3.8.8
conda activate fastp_env
conda install -c bioconda fastp

#check installation
$ fastp
```

## Running fastp on the cluster

1. Running for offsprings
```bash
#!/bin/bash
#SBATCH --job-name=fastp
#SBATCH --account=fc_flyminer
#SBATCH --partition=savio2_htc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --time=70:00:00
#SBATCH --qos=savio_normal
#SBATCH --mail-type=ALL
#SBATCH --mail-user=schaturvedi@berkeley.edu

. ~/.bashrc

module load python
conda activate fastp_env

cd /global/scratch/users/schaturvedi/UC_Davis_sequencing_data_2023/Lane2_offspring/Data/d7kjt94iyc/Un_DTSA779/Project_NWSC_Nova905P_Chaturvedi/

OUTDIR=/global/scratch/users/schaturvedi/monarchs/fastp/offsprings/
LOGS=/global/scratch/users/schaturvedi/monarchs/fastp/offsprings/

for i in $(ls SC*.fastq.gz | sed -r 's/_R[12]_001.fastq.gz//' | uniq); do \
	fastp \
	--in1 "${i}_R1_001.fastq.gz" --in2 "${i}_R2_001.fastq.gz" \
	--out1 "${i}_R1_001.trimmed.fastq.gz" --out2 "${i}_R2_001.trimmed.fastq.gz" \
	--detect_adapter_for_pe -l 50 -h ${i}.html &> ${i}.log \
		mv "${i}_R1_001.trimmed.fastq.gz" $OUTDIR \
		mv "${i}_R2_001.trimmed.fastq.gz" $OUTDIR \
		mv ${i}.html $LOGS \
		mv ${i}.log $LOGS; \
done
```

2. Running for parents
```bash
#!/bin/bash
#SBATCH --job-name=fastp
#SBATCH --account=fc_flyminer
#SBATCH --partition=savio2_htc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --time=70:00:00
#SBATCH --qos=savio_normal
#SBATCH --mail-type=ALL
#SBATCH --mail-user=schaturvedi@berkeley.edu

. ~/.bashrc

module load python
conda activate fastp_env

cd /global/scratch/users/schaturvedi/UC_Davis_sequencing_data_2023/Lane1_parents/Data/uhpnv2kzf0/Un_DTSA778/Project_NWSC_Nova904P_Chaturvedi

OUTDIR=/global/scratch/users/schaturvedi/monarchs/fastp/trimmed_reads
LOGS=/global/scratch/users/schaturvedi/monarchs/fastp/logs

for i in $(ls SC*.fastq.gz | sed -r 's/_R[12]_001.fastq.gz//' | uniq); do \
	fastp \
	--in1 "${i}_R1_001.fastq.gz" --in2 "${i}_R2_001.fastq.gz" \
	--out1 "${i}_R1_001.trimmed.fastq.gz" --out2 "${i}_R2_001.trimmed.fastq.gz" \
	--detect_adapter_for_pe -l 50 -h ${i}.html &> ${i}.log \
		mv "${i}_R1_001.trimmed.fastq.gz" $OUTDIR \
		mv "${i}_R2_001.trimmed.fastq.gz" $OUTDIR \
		mv ${i}.html $LOGS \
		mv ${i}.log $LOGS; \
done
```

# 2. Running fastQC 

Fist, I ran fastQC on each individual data file to generate quality summaries for each file.

*SCRIPT 1 = fastqc_offsprings.sh*

```bash

#!/bin/bash
#SBATCH --job-name=fqc
#SBATCH --account=fc_flyminer
#SBATCH --partition=savio2_htc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --time=70:00:00
#SBATCH --qos=savio_normal
#SBATCH --mail-type=ALL
#SBATCH --mail-user=schaturvedi@berkeley.edu

. ~/.bashrc

module load fastqc

FILES="/global/scratch/users/schaturvedi/monarchs/fastp/offsprings/trimmed_reads/SC*"

for f in $FILES
do
  fastqc $f -o /global/scratch/users/schaturvedi/monarchs/fastqc/offsprings/
done

```
*SCRIPT 2 = fastqc_parents.sh*

```bash
#!/bin/bash
#SBATCH --job-name=fqc_pa
#SBATCH --account=fc_flyminer
#SBATCH --partition=savio2_htc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --time=70:00:00
#SBATCH --qos=savio_normal
#SBATCH --mail-type=ALL
#SBATCH --mail-user=schaturvedi@berkeley.edu

. ~/.bashrc

module load fastqc

FILES="/global/scratch/users/schaturvedi/monarchs/fastp/parents/trimmed_reads/SC*"

for f in $FILES
do
  fastqc $f -o /global/scratch/users/schaturvedi/monarchs/fastqc/parents/
done
```
# 3. MultiQC

## Installing MultiQC using conda

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
## Running multiQC on FastQC output

```bash
#!/bin/bash
#SBATCH --job-name=mulqc
#SBATCH --account=fc_flyminer
#SBATCH --partition=savio2_htc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --time=70:00:00
#SBATCH --qos=savio_normal
#SBATCH --mail-type=ALL
#SBATCH --mail-user=schaturvedi@berkeley.edu

. ~/.bashrc

module load python
conda activate multiqc_env

cd /global/scratch/users/schaturvedi/monarchs/fastqc/offsprings/
multiqc . -o /global/scratch/users/schaturvedi/monarchs/multiqc

mv multiqc_report.html offsprings_multiqc_report.html
mv multiqc_data/ offsprings_multiqc_data/

cd /global/scratch/users/schaturvedi/monarchs/fastqc/parents/
multiqc . -o /global/scratch/users/schaturvedi/monarchs/multiqc

mv multiqc_report.html parents_multiqc_report.html
mv multiqc_data/ parents_multiqc_data/
```
