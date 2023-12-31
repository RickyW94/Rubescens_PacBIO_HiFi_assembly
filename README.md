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
Installation (V1.2)
```
mamba create -n inspector inspector python=2.7
conda activate inspector
```
Clone the repository and add Inspector to PATH
```
git clone https://github.com/ChongLab/Inspector.git
export PATH=$PWD/Inspector/:$PATH
```
Running Inspector

```
inspector.py \
	-c /home/jon/Desktop/UCDavis_PacBIO_Aristotle/RUN/19-final/final.p_ctg.fasta \
	-r /home/jon/Desktop/UCDavis_PacBIO_Aristotle/out.fastq \
	-o /home/jon/Desktop/UCDavis_PacBIO_Aristotle/QC/inspector/orub_pbipa_inspector \
	-t 40 \
	-d hifi
```

## Blobtoolkit
Running blobtoolkit
```
docker run -it --rm --name btk \
  -v /home/jon/Desktop/UCDavis_PacBIO_Aristotle/QC/blobtoolkit:/blobtoolkit/datasets \
  -v /home/jon/Desktop/UCDavis_PacBIO_Aristotle:/blobtoolkit/data \
  genomehubs/blobtoolkit:latest \
  blobtools create \
  --fasta data/RUN/19-final/final.p_ctg.fasta \
  datasets/blobtoolkit/orub_pbipa
```
```
docker run -d --rm --name btk \
	-v /home/jon/Desktop/UCDavis_PacBIO_Aristotle/QC/blobtoolkit/blobtoolkit:/blobtoolkit/datasets \
	-p 8000:8000 -p 8080:8080 \
	-e VIEWER=true \
	genomehubs/blobtoolkit:latest
```


Installing Blobtoolkit
```
pip install blobtoolkit
```

To make the output viewable through a browser we'll use firefox with geckodriver

```
mamba install -c conda-forge firefox geckodriver
```
# Assembling with Hifiasm
Install instructions
```
mamba create -n hifiasm -c bioconda hifiasm
```
Running HiFiasm
```
hifiasm \
	-o O_Rubescens_merged_hifiasm \
	-t 40 \
	/home/jon/Desktop/UCDavis_PacBIO_Aristotle/out.fastq
```
Output will be in .gfa format. Convert to .fastq using gfatools version .4-r214-dirty
```
mamba create -n gfatools -c bioconda gfatools
conda activate gfatools
```
Running gfatools
```
gfatools gfa2fa O_Rubescens_merged_hifiasm.bp.p_ctg.gfa > O_rubescens_hifiasm_homo.fasta
```
# Assembling with Flye
Install using conda
```
conda create -n flye -c bioconda flye
conda activate flye
```
Running flye
```
flye \
	--pacbio-hifi \
	/media/data/rwright/UCDavis_PacBIO_Aristotle/out.fastq.gz \
	--out-dir /media/data/rwright/UCDavis_PacBIO_Aristotle/flye_assembly \
	--scaffold \
	--threads 15
```
