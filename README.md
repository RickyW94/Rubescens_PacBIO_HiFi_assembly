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

# Running Inspector on Genomics Computer
Install as above

Then run
```
inspector.py \
	-c /media/data/rwright/UCDavis_PacBIO_Aristotle/ipa_assembly/final.p_ctg.fasta \
	-r /media/data/rwright/UCDavis_PacBIO_Aristotle/out.fastq.gz \
	-o /media/data/rwright/UCDavis_PacBIO_Aristotle/Inspector/orub_pbipa_inspector \
	-t 15 \
	-d hifi
```
Inspector failed to map any reads to the genome. Moving on to CRAQ for genome assessment and NextPolish2.
# Running CRAQ and NextPolish2
While Inspector could do the polishing for us, CRAQ will just get us an idea of how much polishing we'll need to do with NextPolish2
## Install CRAQ
```
git clone https://github.com/JiaoLaboratory/CRAQ.git 
```
## Run CRAQ
Running CRAQ without short reads
```
perl ./CRAQ/bin/craq -g ipa_assembly/final.p_ctg.fasta -sms out.fastq.gz -x map-hifi
```

## Installing NextPolish
```
wget https://github.com/Nextomics/NextPolish/releases/download/v1.4.1/NextPolish.tgz
```

```
pip install paralleltask
tar -vxzf NextPolish.tgz && cd NextPolish && make
```
## NextPolish2 dependencies and prep (partially deprecated, actually using NextPolish, not NextPolish2)
Nextpolish2 is dependent on short read data, which we don't have. However, the first steps for Nextpolish2 are the same, so we can salvage the alignment generated from the steps below below.
Map raw reads to assembly using minimap2 and samtools
```
minimap2 -ax map-hifi -t 15 ./ipa_assembly/final.p_ctg.fasta out.fastq.gz|samtools sort -o hifi.map.sort.bam -
```
Index the bam output
```
samtools index hifi.map.sort.bam
```
In order to proceed from this point using NextPolish, we must pull the requisite steps from the bash script in step 4 of the [NextPolish tutorial](https://nextpolish.readthedocs.io/en/latest/TUTORIAL.html).
This script creates an alignment using minimap2 then runs nextpolish, then repeats once by default. We will have to do that all manually twice.
Unfortunately, it doesn't specify which version of python to use, so for now we'll assume python3 is adequate.
```
ls `pwd`/hifi.map.sort.bam > hifi.map.sort.bam.fofn #Generates a list (of length 1 in this case) of files for nextpolish
python NextPolish/lib/nextpolish2.py -g ./ipa_assembly/final.p_ctg.fasta -l hifi.map.sort.bam.fofn -r hifi -p 15 -sp -o assembly.nextpolish.fa
```
Nextpolish complains about too many fragments mapped to the genome and suggests using asm20 instead of map-hifi in minimap2's settings. Since the alignment takes so long, I'm going to proceed as though this is the next round of polishing, starting by aligning the reads to the already polished genome.
```
minimap2 -ax asm20 -t 15 assembly.nextpolish.fa out.fastq.gz|samtools sort -o asm20.map.sort.bam -
```
Index the bam file
```
samtools index asm20.map.sort.bam
```
Now perform a second round of polishing on the alignment output 'asm20.map.sort.bam'
```
ls `pwd`/asm20.map.sort.bam > asm20.map.sort.bam.fofn
python NextPolish/lib/nextpolish2.py -g ./assembly.nextpolish.fa -l asm20.map.sort.bam.fofn -r hifi -p 15 -sp -o assembly_2.nextpolish.fa
```
The prior output was named 'assembly.nextpolish.fa', the output of this second round will be named 'assembly_2.nextpolish.fa'

# Busco
## Install Busco
```
mamba create -n busco -c conda-forge -c bioconda busco=5.6.1
```
```
conda activate busco
```
## Run Busco
First we need to replace all of the '/' in the IPA output with '_'. I used nano to find and replace all '/' in the headers.
Use the mollusc database

Run for each version. First is 'media/data/rwright/UCDavis_PacBIO_Aristotle/ipa_assembly/final.p_ctg.fasta'. Second is 'media/data/rwright/UCDavis_PacBIO_Aristotle/assembly.nextpolish.fa'. Third is 'media/data/rwright/UCDavis_PacBIO_Aristotle/assembly_2.nextpolish.fa'
```
busco -i final.p_ctg.fasta -l mollusca_odb10 -o final.p_ctg_busco -m genome -c 15 -f
```
