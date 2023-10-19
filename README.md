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
  -j 70 \ # number of threads
  m84066_230908_205816_s3.hifi_reads.default.bam \
  m84066_230914_182946_s2.hifi_reads.default.bam
```
