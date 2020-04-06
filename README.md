# A test RNA-Seq pipeline for beginners
- jinling 2020/04/01
- original pipeline: http://www.bio-info-trainee.com/2218.html
- article: Davidovich C, Zheng L, Goodrich KJ, Cech TR. Promiscuous RNA binding by Polycomb repressive complex 2. Nat Struct Mol Biol 2013 Nov;20(11):1250-7. PMID: 24077223

## Download the human reference genome
- Download version hg19 deposited in UCSC. Code:
```
mkdir -p ~/rna-seq/data/reference && cd ~/rna-seq/data/reference
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz && tar -zxvf chromFa.tar.gz
```
- MD5 sum is provided here: http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/md5sum.txt. We can use `md5sum chromFa.tar.gz` to check whether the reference genome we downloaded is intact.
- Note that these fasta files represent seperate chromosomes. We need to merge these files (scripts/merge.sh) `bash merge.sh`. 
- To save hard-disk space, we remove these seperate fasta files `rm chr*.fa` and the original compressed file `rm chromFa.tar.gz`.

## Download raw RNA-Seq sequences
- Download raw data in .sra format from https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos1/sra-pub-run-5/SRR957[677-680]/SRR957[677-680].1
- I downloaded the raw sequences to my Windows10 system, and transferred them to my WSL (Windows Subsystem for Linux) (scripts/win-2-ubuntu.sh) `bash win-2-ubuntu.sh`.
- Here is the raw data list:

| Sample | SRA |
| :--: | :--: |
| siCtrl_BiolRep1 | SRR957677 |
| siCtrl_BiolRep2 | SRR957678 |
| siSUZ12_BiolRep1 | SRR957679 |
| siSUZ12_BiolRep2 | SRR957680 |