# Genome Analysis Tutorial

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

1. Before installing samtools, check if it is installed.
```bash
which samtools
```
if you see directory paths, it means that samtools is installed.

2. Samtools website
https://www.htslib.org/download/
copy link address of download (samtools-1.20)

```bash
wget https://github.com/samtools/samtools/releases/download/1.20/samtools-1.20.tar.bz2
```
3. unzip it
```bash
tar xvf samtools-1.20.tar.bz2
```
Check the directory of samtools-1.20


4. Installing dependencies

Slightly tricky
change to administrator mode
```bash
sudo su
```
In the directory of samtools-1.20
```bash
ls configure
```
```bash
./configure
```
Check what is not installed.
For example, if gcc is not installed,

```bash
apt-get update
```
```bash
apt-get install gcc
```
check the configure again
```bash
./configure
```
Repeat this step until all dependecies are installed (should be no error).
Read notes carefully.

After installing all dependencies, install make
```bash
apt-get install make
```
cf. make
The commands make and make install are commonly used in the process of building and installing software from source code.
When you run the make command, it reads the instructions from a file called Makefile (or makefile), which defines how to compile and link the program. 
The Makefile contains rules and dependencies that dictate how the source files should be compiled into executable binaries or other target files.

```bash
make
```
cf. make install
After the program has been successfully compiled, you need to install it so that it can be used system-wide. The make install command reads another set of instructions from the Makefile, which typically involves copying the compiled binaries, configuration files, and other necessary files to the appropriate directories in the system (such as /usr/local/bin, /usr/local/lib, etc.).

```bash
make install
```
```bash
which samtools
```

To exit root (administrator mode)
```bash
exit
```
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
#### 1.5.4 setting

```bash
nano ~/.bashrc
```
export SAMTOOLS=~/bioinfo_tutorials/tools/samtools-1.20/samtools
export BWA2=~/bioinfo_tutorials/tools/bwa-mem2-2.0pre2_x64-linux/bwa-mem2
export GATK4=~/bio~/bioinfo_tutorials/tools/gatk-4.6.0.0/gatk-package-4.6.0.0-l>


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

Making index file
```bash
$BWA2 index hg38.chr21.fa
```
  
Mapping reads into reference with BWA2 tools

```bash
$BWA2 mem -t 8 -R "@RG\tID:sample\tSM:sample\tPL:platform" ../resource/reference/hg38.chr21.fa sample_1.fastq.gz sample_2.fastq.gz > sample.mapped.sam
```
(comments)
mem : BWA-MEM algorithm for alignment
-t -8 : 8 number of thread
-R : This option adds read group information to each read in the output SAM file. This information is critical for downstream analysis tools like GATK.
@RG : start of a read group
\t : represent a tab character
ID:sample : specifies the read group ID
SM:sample : specifies the sample name
PL:platform : specifies the sequencing platform (ex illumina)
">" : redirects the output to a file.


BWA2 mem commands produces sam files. So, it is recommend to change bam file. (sam file is too big)

```bash
$SAMTOOLS view -Sb sample.mapped.sam > sample.mapped.bam
```
(comments)
view : used to view, filter, and convert alignment files.
-S: indicates that the input is in SAM format. 
-b: specifies that the output should be in BAM format. 

How to check mapping reads
```bash
$SAMTOOLS view -h sample.mapped.bam | less -S
```
(comments)
-h : includes the header information from the BAM file. 
The pipe (|) is used to pass the output of one command as input to another command.
less : terminal pager program used to view the contents of a file one screen at a time.
-S: tells less to disable line wrapping. This means lines that are too long to fit on the screen will be truncated, 
and you can scroll horizontally to view the entire line. This is particularly useful for viewing SAM files where each line can be very long.


## 4. Duplication read marking
This shows how to mark duplicated reads. This is very important for variant calling process.
We will show how to process it with samtools markup.  

```bash
$SAMTOOLS fixmate -m sample.mapped.bam sample.fixmate.bam
```
(comments)
fixmate: corrects and updates mate-pair information in a BAM file
-m: Adds MC and MQ tags for mate coordinates and mate mapping quality, which are useful for subsequent duplicate marking steps.

```bash
$SAMTOOLS sort -o sample.fixmate.sorted.bam sample.fixmate.bam
```
(comments)
sort : sorts a BAM file by coordinates or by read names. Sorting is essential for many downstream applications, such as indexing and variant calling.
-o: Specifies the output file.

```bash
$SAMTOOLS markdup sample.fixmate.sorted.bam sample.markdup.bam
```
(comments)
markup : identifies and marks duplicate reads in a BAM file

```bash
$SAMTOOLS index sample.markdup.bam
```
(comments)
index : creates an index for a sorted BAM file. 

flag information in BAM file
https://broadinstitute.github.io/picard/explain-flags.html

In this website, 1024 : read is PCR or optical duplicate
To show duplicated reads only, 
```bash
$SAMTOOLS view -f 1024 sample.markdup.bam
```
ex) the first duplicated read's flag is 1153. So, if 1153 is entered at explain-flags site, it will show that read is PCR or optical duplicate.  

Other method  
```bash
$SAMTOOLS tview sample.markdup.bam
```
In the new window, push '/' key and then input ```chr21:5012650```
Each line is read.

cf. counting total lines of a bam file
```bash
$SAMTOOLS view sample.markup.bam | wc -l
```
So, to view only duplicated read
```bash
$SAMTOOLS view -f 1024 sample.markup.bam | wc -l
```

## 5. Variant calling

Reading: chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://samtools.github.io/hts-specs/VCFv4.2.pdf
Lastly, we will process to find the way of variant calling
```bash
java -jar $GATK4 BaseRecalibrator -I sample.markdup.bam -R ../resource/reference/hg38.chr21.fa --known-sites ../resource/knownsites/hg38_v0_Homo_sapiens_assembly38.known_indels.chr21.vcf.gz -L chr21 -O sample.recal_data.table
```

```bash
java -jar $GATK4 ApplyBQSR -R ../resource/reference/hg38.chr21.fa -I sample.markdup.bam --bqsr-recal-file sample.recal_data.table -L chr21 -O sample.recal.bam
```

```bash
java -jar $GATK4 HaplotypeCaller -R ../resource/reference/hg38.chr21.fa -I sample.recal.bam -L chr21 -O sample.g.vcf -ERC GVCF
```

```bash
java -jar $GATK4 GenotypeGVCFs -R ../resource/reference/hg38.chr21.fa -V sample.g.vcf -L chr21 -O sample.vcf
```

```bash
less -S sample.vcf
```

