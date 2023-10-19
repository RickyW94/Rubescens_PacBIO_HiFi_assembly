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
## Then install IPA
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
cat m84066_230908_205816_s3.hifi_reads.default.fastq m84066_230914_182946_s2.hifi_reads.default.fastq > O_Rubescens_merged.fastq
```
