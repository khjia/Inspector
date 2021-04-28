# Inspector

A reference-free assembly evaluator.

Author: Maggi Chen

Email: maggic@uab.edu

Draft date: Apr. 20, 2021

## Quick Start
```sh
git clone https://github.com/Maggi-Chen/Inspector.git
cd Inspector/
./inspector.py -h

# Evaluate assembly with raw reads
./inspector.py -c contig.fa -r rawreads.1.fastq rawreads.2.fastq -o inspector_out/ --datatype clr 
# Evaluate assembly with hifi reads
./inspector.py -c contig.fa -r ccsreads.1.fastq ccsreads.2.fastq -o inspector_out/ --datatype hifi

# With reference-based evaluation
./inspector.py -c contig.fa -r rawreads.1.fastq --ref reference.fa -o inspector_out/ --datatype clr

# Reference-based only evaluation
./inspector.py -c contig.fa -r emptyfile --ref reference.fa -o inspector_out/ 

# Error correction


```



## Description

Inspector is a tool for assembly evaluation with long read data. The input includes a contig file, long reads (PacBio CLR, PacBio HiFi, Oxford Nanopore, or mixed platform), and a reference genome (optional). The output includes A summary report, read-to-contig alignment file, a list of structrual errors and small-scale errors. This program was tested on a x86_64 Linux system with a 8GB physical memory.

## Depencency

Dependencies for Inspector:

python 2.7  
* pysam
* statsmodels

* minimap2  (tested with version 2.10 and 2.15)
* samtools  (tested with version 1.9)


Dependencies for Inspector error correction module:
* flye  (tested with version 2.8.2)


## Installation

```
git clone https://github.com/Maggi-Chen/Inspector.git
```
Then, please also add this directory to your PATH:
```
export PATH=$PWD/Inspector/:$PATH
```


To simplify the environment setup process, Anaconda (https://www.anaconda.com/) is recommended.
To create an environment with conda:
```
conda create --name ins python=2.7
conda activate ins
conda install -c bioconda minimap2=2.15
conda install -c bioconda samtools=1.9
conda install -c bioconda pysam=0.16.0.1
conda install -c anaconda statsmodels=0.10.1
conda install -c bioconda flye=2.8.2

inspector.py -h
```


## General usage


```

inspector.py [-h] -c contig.fa -r raw_reads.fa -o output_dict/
  required arguments:
  --contig,-c           FASTA/FASTQ file containing contig sequences to be evaluated
  --read,-r             A list of FASTA/FASTQ files containing long read sequences

  optional arguments:
  -h, --help            Show this help message and exit
  --version             Show program's version number and exit
  --datatype,-d         Input read type. (clr, hifi, nanopore, mixed) [clr]
  --ref                 OPTIONAL reference genome in .fa format
  --thread,-t           Number of threads. [8]
  --min_contig_length   Minimal length for a contig to be evaluated [10000]
  --min_contig_length_assemblyerror    Minimal contig length for assembly error detection. [1000000]
  --pvalue              Maximal p-value for small-scale error identification [0.01 for HiFi, 0.05 for others]
  --skip_read_mapping   Skip the step of mapping reads to contig
  --skip_structural_error       Skip the step of identifying large structural errors
  --skip_structural_error_detect       Skip the step of detecting large structural errors
  --skip_base_error     Skip the step of identifying small-scale errors
  --skip_base_error_detect      Skip the step of detecting small-scale errors from pileup



inspector-correct.py [-h] -i inspector_out/ --data pacbio-raw 
  required arguments:
  --inspector,-i        Inspector evaluation directory with original file names
  --data                Type of read used for Inspector evaluation. Required for structural error correction
  --outpath,-o          Output directory
  --thread,-t           Number of threads
  --skip_structural     Do not correct structural errors. Local assembly will not be performed
  --skip_baseerror      Do not correct small-scale errors
  

```

## Use cases
Inspector evaluates the contigs and identifies assembly errors with sequencing reads. You can use reads from single platform:
```
./inspector.py -c contig.fa -r rawreads.1.fastq rawreads.2.fastq -o inspector_out/ --datatype clr
```
Or use a mixed data type:
```
./inspector.py -c contig.fa -r rawreads.fastq nanopore.fastq -o inspector_out/ --datatype mixed
```
Reference-based evaluation is also supported:
(Note that reported assembly error from reference-based mode will contain genetic variants)
```
./inspector.py -c contig.fa -r rawreads.fastq --ref reference.fa -o inspector_out/ --datatype clr
```
If only the continuity analysis is needed, simply provide an empty file for --read:
```
./inspector.py -c contig.fa -r emptyfile -o inspector_out/ --skip_base_error --skip_structural_error
```
For the '--skip' options, do not use unless you are repeating the evaluation with same contig and read files in the same output directory. These may help save time when testing different options for error detection.



Inspector provides an error-correction module to improve assembly accuracy. High-accuracy reads are recommended, especially for small-scale error correction:
(Note that only reads from single platform are supported for error correction.)
```
./inspector-correct.py -i inspector_out/ --datatype hifi -o inspector_out/ 
```


