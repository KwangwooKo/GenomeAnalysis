# GenomeAnalysis

## 0. Intro

This is a tutorial to make DNA Genome Analysis Variant Calling Pipeline  
In this tutorial, we use **BWA2**, **GATK4** **SAMTOOLS**   


## 1. Sample preparation
### 1.1 About samples in this tutorial
data directory: NA12878 (chromosome 21)  
resource directory: reference sequence (chromosome 21)

### 1.2 Files download
```bash
git clone https://github.com/KwangwooKo/GenomeAnalysis.git
```

### 1.3 File preparation
In reference directory, ```hg38.chr21.fa.bwt.2bit.64```  : this file is zipped => need unzip

```bash
cd resource/reference
```
```
gunzip hg38.chr21.fa.bwt.2bit.64.gz
```

### 1.5 Tools installation
#### 1.5.1 BWA2

-Option 1
```
curl -L https://github.com/bwa-mem2/bwa-mem2/releases/download/v2.0pre2/bwa-mem2-2.0pre2_x64-linux.tar.bz2  | tar jxf -
```
```
cd bwa-mem2-2.0pre2_x64-linux
```
```
./bwa-mem2
```
```
Usage: bwa-mem2 <command> <arguments>
Commands:
  index         create index
  mem           alignment
  version       print version number
```

-Option 2
```bash
git clone --recursive https://github.com/bwa-mem2/bwa-mem2
```
```
cd bwa-mem2
```
```
make
```
```
./bwa-mem2
```

#### 1.5.2 Samtools

https://www.youtube.com/watch?v=8bau7KESJTo

How to check
```bash
samtools
```
```
Program: samtools (Tools for alignments in the SAM format)
Version: 1.12 (using htslib 1.12)

Usage:   samtools <command> [options]

Commands:
  -- Indexing
```

#### 1.5.3 GATK4

Download from https://gatk.broadinstitute.org/hc/en-us

```bash
wget https://github.com/broadinstitute/gatk/releases/download/4.6.0.0/gatk-4.6.0.0.zip
```
```
unzip gatk-4.6.0.0.zip
```
How to check
```bash
java -jar gatk-package-4.2.0.0-local.jar
```
```
USAGE:  <program name> [-h]

Available Programs:
--------------------------------------------------------------------------------------
Base Calling:                                    Tools that process sequencing machine data, e.g. Illumina base calls, and detect sequencing level attributes, e.g. adapters
    CheckIlluminaDirectory (Picard)              Asserts the validity for specified Illumina basecalling data.  
    CollectIlluminaBasecallingMetrics (Picard)   Collects Illumina Basecalling metrics for a sequencing run. 
...
```


## 2. Workflow

Samples -> sequencer -> FASTQ -> (reference sequence에 mapping) -> BAM -> (variant calling) -> VCF

## 3. Mapping in Reference

First, take a look at FASTQ files

```bash
cd data
```
```
zless -S sample_1.fastq.gz
```
```bash
zcat sample_1.fastq.gz | wc -l
```
```
328476
```
```
zcat sample_2.fastq.gz | wc -l
```
```
328476
```

328476 (line) / 4 = 82119 (reads)
  

Mapping reads into reference with BWA2 tools

```bash
$BWA2 mem -t 1 -R "@RG\tID:sample\tSM:sample\tPL:platform" ../resource/reference/hg38.chr21.fa sample_1.fastq.gz sample_2.fastq.gz > sample.mapped.sam
```

BWA2 mem commands produces sam files. So, it is recommend to change bam file. (sam file is too big)

```bash
$SAMTOOLS view -Sb sample.mapped.sam > sample.mapped.bam
```

How to check mapping reads
```bash
$SAMTOOLS view -h sample.mapped.bam | less -S
```

## 4. Duplication read marking
This shows how to mark duplicated reads. This is very important for variant calling process.
We will show how to process it with samtools markup.  

```bash
$SAMTOOLS fixmate -m sample.mapped.bam sample.fixmate.bam
```

```bash
$SAMTOOLS sort -o sample.fixmate.sorted.bam sample.fixmate.bam
```

```bash
$SAMTOOLS markdup sample.fixmate.sorted.bam sample.markdup.bam
```

```bash
$SAMTOOLS index sample.markdup.bam
```
