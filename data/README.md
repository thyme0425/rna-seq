# Data sources
2020/04/06
## Human reference genome (/reference)
- Version hg19 deposited in UCSC
- Downloaded on 2020/04/06 12:47
```
mkdir -p ~/rna-seq/data/reference && cd ~/rna-seq/data/reference
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz && tar -zxvf chromFa.tar.gz
```
- SHA-1 sum: eae3720102935210a10c184d68e3024ce05b18a0   `chromFa.tar.gz`
## Raw RNA-Seq sequences (/raw)
- Download from NCBI-SRA: https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos1/sra-pub-run-5/SRR957[677-680]/SRR957[677-680].1
- Downloaded on 2020/04/06 12:47
- SHA-1 sum:
e59e2b6925d683b2527001af65b7fbb53d812a92  `SRR957677.1.sra`
b8b822a23e93a856f9a4c18f0d93b57d9626a541  `SRR957678.1.sra`
762aad4621615168719142d2f3a3015ffe7cafe6  `SRR957679.1.sra`
3930cebe53a786efc66f27e5e8b50f564e30b27d  `SRR957680.1.sra`