# A test RNA-Seq pipeline for beginners
- jinling 2020/04/01
- original pipeline: http://www.bio-info-trainee.com/2218.html
- article: Davidovich C, Zheng L, Goodrich KJ, Cech TR. Promiscuous RNA binding by Polycomb repressive complex 2. Nat Struct Mol Biol 2013 Nov;20(11):1250-7. PMID: 24077223

## **Tools**
- fastq-dump 2.8.0
- fastqc 0.11.9
- hisat2 2.2.0
- samtools 1.7
- htseq 0.11.3
## **Download the human reference genome**
- Download version hg19 deposited in UCSC. Code:
```
mkdir -p ~/rna-seq/data/reference && cd ~/rna-seq/data/reference
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz && tar -zxvf chromFa.tar.gz
```
- MD5 sum is provided here: http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/md5sum.txt. We can use `md5sum chromFa.tar.gz` to check whether the reference genome we downloaded is intact.
- Note that these fasta files represent seperate chromosomes. We need to merge these files (scripts/merge.sh) `bash merge.sh`. 
```
#!/bin/bash
# Filename: merge.sh
# Function:
#   Merge the seperate fasta files in human reference gonome hg19.
#   $(seq 1 22), X, Y, M represent chromosomes.
# 2020/04/06  jinling

reference=../data/reference

for i in $(seq 1 22) X Y M
do
    cat $reference/chr$i.fa >> $reference/hg19.fa
done
```
- To save hard-disk space, we remove these seperate fasta files `rm chr*.fa` and the original compressed file `rm chromFa.tar.gz`.

## **Download raw RNA-Seq sequences**
- Download raw data in .sra format from https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos1/sra-pub-run-5/SRR957[677-680]/SRR957[677-680].1
- I downloaded the raw sequences to my Windows10 system, and transferred them to my WSL (Windows Subsystem for Linux) (scripts/win-2-ubuntu.sh) `bash win-2-ubuntu.sh`.
```
#!/bin/bash
# Filename: win-2-ubuntu.sh
# Function:
#   Move .sra files downloaded in Windows10 system to WSL.
# 2020/04/06  jinling

source=/mnt/c/Users/jinling/Downloads
target=../data/raw

for i in $(seq 677 680)
do
    cp $source/SRR957$i.1 $target/SRR957$i.1.sra
done
```
- Here is the raw data list:

| Sample | SRA |
| :--: | :--: |
| siCtrl_BiolRep1 | SRR957677 |
| siCtrl_BiolRep2 | SRR957678 |
| siSUZ12_BiolRep1 | SRR957679 |
| siSUZ12_BiolRep2 | SRR957680 |
## **Convert .sra format to .fastq format**
- Use `fastq-dump` program to do the conversion `bash sra-2-fastq.sh` (scripts/sra-2-fastq.sh)
```
#!/bin/bash
# Filename: sra-2-fastq.sh
# Function:
#   Convert RNA-Seq raw data in .sra format to .fastq format.
# 2020/04/07  jinling

path=../data/raw

ls $path/*.sra | {
while read id
do
    fastq-dump --split-3 $id -O $path
done
}
```
- By default, the output of `fastq-dump` will be in the `.` directory. We use `-O` to define the output path. 
- This procedure takes about 8 minutes.
- To save hard-disk space, we remove .sra files `rm *.sra`.
## **Check the quality of RNA-Seq sequences**
- We created the quality reports using the program `fastqc`.
```
ls *.fastq | xargs fastqc -o ../../analysis/fastqc/ -t 8
```
- `-o` defines the output path; `-t` defines the number of threads.
- The procedure takes a few minutes.
- We move the quality check results from WSL to Windows10 to check them on the browser `cp *.html /mnt/c/Users/jinling/Downloads/`.
## **Filter bad quality reads and remove adaptors**
- *Here we skip the step which trims low quality bases and adaptor sequences.*
## **Mapping reads to the reference genome**
### *Build or download the reference genome index*
- Here I build it myself.
```
hisat2-build -p 4 ../reference/hg19.fa genome
```
```
wget https://cloud.biohpc.swmed.edu/index.php/s/hg19/download 
```
- `-p` defines the number of threads.
- This procedure takes 41min47s.
### *Mapping reads*
- Mapping reads to the reference genome index `bash mapping.sh` (scripts/mapping.sh)
```
#!/bin/bash
# Filename: mapping.sh
# Function:
#   Mapping RNA-Seq reads to the reference genome.
# 2020/04/09  jinling

raw=../data/raw
index=../data/index/genome
align=../analysis/align

hisat2 -p 5 -x $index -U $raw/SRR957677.1.fastq -S $align/siCtrl_1.sam 2> $align/siCtrl_1.log
hisat2 -p 5 -x $index -U $raw/SRR957678.1.fastq -S $align/siCtrl_2.sam 2> $align/siCtrl_2.log
hisat2 -p 5 -x $index -U $raw/SRR957679.1.fastq -S $align/siSUZ12_1.sam 2> $align/siSUZ12_1.log
hisat2 -p 5 -x $index -U $raw/SRR957680.1.fastq -S $align/siSUZ12_2.sam 2> $align/siSUZ12_2.log
```
- This procedure takes 17min5s.
### *Convert .sam format to .bam format*
- The program `samtools sort` will **convert** .sam format to .bam format (to save hard-disk space), as well as **sort** the aligned reads.
- `bash sam-2-bam.sh` (scripts/sam-2-bam.sh)
```
#!/bin/bash
# Filename: sam-2-bam.sh
# Function:
#   Convert .sam format to .bam format.
# 2020/04/10  jinling

path=../analysis/align

ls $path/*.sam | {
while read id
do
    nohup samtools sort -n -@ 5 \
    -o $path/$(basename $id ".sam").sort.bam $id &
done
}
```
- `-n` indicates sorting by read name.
- This procedure takes about 10min.
- Remove .sam files to save hard-disk space `rm *.sam`.
## **Counting reads**
- The program `htseq-count` counts reads aligned to specific genes.
- `bash count.sh`. (scripts/count.sh)
```
#!/bin/bash
# Filename: count.sh
# Function:
#   Counting reads which mapped to specific genes.
# 2020/04/10  jinling

input=../analysis/align
annotation=../data/annotation/gencode.v19.annotation.gtf.gz
output=../analysis/count

ls $input/*.sort.bam | {
while read id
do
    nohup samtools view $id |\
    htseq-count -f sam -s no -i gene_name - $annotation \
    1>$output/$(basename $id ".sort.bam").genecounts \
    2>$output/$(basename $id ".sort.bam").htseq.log &
done
}
```
- This procedure takes ~ 40min.
## **Differential gene expression analysis**
- This procedure is executed in **R**.
- Move read count file to Windows10 system `cp *.genecounts /mnt/c/Users/jinling/Downloads`.