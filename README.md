# Rubescens_PacBIO_HiFi_assembly
## Installing basecaller
We'll use [pbtk](https://github.com/PacificBiosciences/pbbioconda) V3.1.0
```
  mamba create -n pbtk -c bioconda pbtk
```
```
  conda activate pbtk
```
Then run pbtk on the bam files from UCDavis
```
  bam2fastq \
  -o out \
  -j 70 \
  m84066_230908_205816_s3.hifi_reads.default.bam \
  m84066_230914_182946_s2.hifi_reads.default.bam
```
## Installing IPA V1.1.2
From [IPA's github](https://github.com/PacificBiosciences/pbipa)
Select channels
```
  conda config --prepend channels defaults
  conda config --prepend channels conda-forge
  conda config --prepend channels bioconda
```
Then install latest version of IPA
```
  mamba create -n ipa
  mamba activate ipa
  mamba install pbipa
```

Once IPA is installed, we need to concatenate our fastqs into a single file for use with IPA
```
  cat \
  m84066_230908_205816_s3.hifi_reads.default.fastq \
  m84066_230914_182946_s2.hifi_reads.default.fastq \
  > O_Rubescens_merged.fastq
```

## Running IPA
```
  ipa local --nthreads 20 --njobs 4 -i O_Rubescens_merged.fastq
```
# Quality Assessment

## Inspector
Install inspector dependencies
  python
  pysam
  statsmodels (tested with version 0.10.1)
  minimap2 (tested with version 2.10 and 2.15)
  samtools (tested with version 1.9)
Inspector error correction module dependencies
  flye (tested with version 2.8.3)
Installation
```
  mamba create --name ins
  mamba activate ins
  mamba install -c bioconda inspector
```
Clone the repository and add Inspector to PATH
```
  git clone https://github.com/ChongLab/Inspector.git
  export PATH=$PWD/Inspector/:$PATH
```
Running Inspector

Using the example from Inspector's github for evaluation with hifi reads
```
inspector.py \
  -c O_Rubescens_assembly.fastq \
  -r O_Rubescens_merged.fastq \
  -o inspector_out/ --datatype hifi
```


## Blobtoolkit
Installing Blobtoolkit
```
pip install blobtoolkit
```
To make the output viewable through a browser we'll use firefox with geckodriver
```
mamba install -c conda-forge firefox geckodriver
```
