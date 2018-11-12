# Step 3: Human genome 38 

<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_hadoop.md><< Hadoop Setup</a>  

We will first download the BWA aligner, which will align files against our reference human genome `hg38` which is the 38 version of the human genome assembly. The downloaded files will be inserted into our HDFS. 

# Install Spark BWA: Burrows-Wheeler Aligner
# (On master node, spark1)
For detailed instruction for <a href=https://github.com/citiususc/SparkBWA>Spark BWA</a>  

if on Ubuntu:
```
# apt-get -y install gcc make git zlib-devel maven 
```
if on Centos:
```
# yum install -y gcc make git zlib-devel maven 
```
assuming you are building this in /root
```
# cd /root
# git clone https://github.com/citiususc/SparkBWA.git
# cd SparkBWA
# mvn package

[INFO] Dependency-reduced POM written at: /root/SparkBWA/dependency-reduced-pom.xml
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2:06.587s
[INFO] Finished at: Mon Nov 12 14:03:18 CST 2018
[INFO] Final Memory: 32M/241M
[INFO] ------------------------------------------------------------------------
```
**Note**  
Detailed instructions on SparkBWA is <a href=https://github.com/citiususc/SparkBWA>Here</a>. 

## Distribute SparkBWA to other nodes (Optional) 
Now, let us distribute our build to other nodes - not strictly required, but why not:
```
cd ~
# scp -r SparkBWA spark2:root/
# scp -r SparkBWA spark3:root/
...
...
# scp -r SparkBWA spark9:root/
# scp -r SparkBWA spark10:root/
```
------------
## Download two files 
```
wget ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/data/NA12750/sequence_read/ERR000589_1.filt.fastq.gz
wget ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/data/NA12750/sequence_read/ERR000589_2.filt.fastq.gz
```
#### uncompress
```
gzip -d ERR000589_1.filt.fastq.gz
gzip -d ERR000589_2.filt.fastq.gz
```
## push files to HDFS
```
hdfs dfs -copyFromLocal ERR000589_1.filt.fastq /ERR000589_1.filt.fastq
hdfs dfs -copyFromLocal ERR000589_2.filt.fastq /ERR000589_2.filt.fastq
```

## Download a reference human genome hg38 (2 hours)
You need to install `ftp` to connect to remote ftp server. 
```
# yum install ftp
```
We're downloading the reference hg38 from boradinstitute which setup ftp server. So we're connecting and downloading hg38 files over ftp. When asked password, just hit `Enter`. No need to have a password. We're still working in master node, spark1.  
For more reference, <a href=https://software.broadinstitute.org/gatk/download/bundle>Broad Institute</a>.
```
mkdir -pm 777 /Data/HumanBase
cd /Data/HumanBase
ftp ftp.broadinstitute.org
user: gsapubftp-anonymous
```
We will download the **23 files** from the Broad Institute ftp server. This will take more than an hour. Once you're in ftp mode, enter these commands below to ensure non-interactive and binary modes. When you're in `ftp>` prompt, do the following. 
```
> prompt
> binary
> cd bundle
> cd hg38
> mget *
```
Check if you have all the files in `/Data/HumanBase/`.
```
# ls -altr -h

drwxr-xr-x. 3 root root 4.0K Nov 11 12:49 ..
-rw-r--r--. 1 root root 3.0G Nov 11 13:30 dbsnp_144.hg38.vcf.gz
-rw-r--r--. 1 root root  51M Nov 11 13:30 1000G_omni2.5.hg38.vcf.gz
-rw-r--r--. 1 root root 477K Nov 11 13:30 Homo_sapiens_assembly38.fasta.64.alt
-rw-r--r--. 1 root root 849M Nov 11 13:41 Homo_sapiens_assembly38.fasta.gz
-rw-r--r--. 1 root root 1.5M Nov 11 13:41 hapmap_3.3_grch38_pop_stratified_af.vcf.gz.tbi
-rw-r--r--. 1 root root 569K Nov 11 13:41 Homo_sapiens_assembly38.dict
-rw-r--r--. 1 root root 1.8G Nov 11 14:03 1000G_phase1.snps.high_confidence.hg38.vcf.gz
-rw-r--r--. 1 root root 1.5G Nov 11 14:20 dbsnp_138.hg38.vcf.gz
-rw-r--r--. 1 root root 1.5M Nov 11 14:21 hapmap_3.3.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 412K Nov 11 14:21 Axiom_Exome_Plus.genotypes.all_populations.poly.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 1.5M Nov 11 14:21 1000G_omni2.5.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 2.4M Nov 11 14:21 dbsnp_146.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 3.2G Nov 11 14:52 dbsnp_146.hg38.vcf.gz
-rw-r--r--. 1 root root  20M Nov 11 14:52 Mills_and_1000G_gold_standard.indels.hg38.vcf.gz
-rw-r--r--. 1 root root  60M Nov 11 14:52 hapmap_3.3.hg38.vcf.gz
-rw-r--r--. 1 root root 1.5M Nov 11 14:52 Mills_and_1000G_gold_standard.indels.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 158K Nov 11 14:52 Homo_sapiens_assembly38.fasta.fai
-rw-r--r--. 1 root root 2.3M Nov 11 14:52 dbsnp_138.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 172M Nov 11 14:54 hapmap_3.3_grch38_pop_stratified_af.vcf.gz
-rw-r--r--. 1 root root 586K Nov 11 14:54 wgs_calling_regions.hg38.interval_list
-rw-r--r--. 1 root root 2.4M Nov 11 14:54 dbsnp_144.hg38.vcf.gz.tbi
-rw-r--r--. 1 root root 3.0M Nov 11 14:54 Axiom_Exome_Plus.genotypes.all_populations.poly.hg38.vcf.gz
drwxrwxrwx. 2 root root 4.0K Nov 11 14:54 .
-rw-r--r--. 1 root root 2.1M Nov 11 14:54 1000G_phase1.snps.high_confidence.hg38.vcf.gz.tbi
```

## Index the reference human genome (2 hours)

We need to index the human genome. Note that the command below will take 1-2 hours.  

if on ubuntu
```
# apt-get install -y bwa
```
if on centos
```
# yum install -y epel-release
# yum install -y bwa
# bwa index Homo_sapiens_assembly38.fasta.gz

[bwa_index] Pack FASTA... 60.64 sec
[bwa_index] Construct BWT for the packed sequence...
[BWTIncCreate] textLength=6434693834, availableWord=464768632
[BWTIncConstructFromPacked] 10 iterations done. 99999994 characters processed.
[BWTIncConstructFromPacked] 20 iterations done. 199999994 characters processed.
[BWTIncConstructFromPacked] 30 iterations done. 299999994 characters processed.
...
...
...
```
----------
# Distribute all files to other nodes
If you're not in other nodes, from the `spark1`, ssh into all other nodes, make a directory `/Data`. 
```
# ssh spark2 mkdir -m 777 /Data
# ssh spark3 mkdir -m 777 /Data
# ssh spark4 mkdir -m 777 /Data
# ssh spark5 mkdir -m 777 /Data
# rsync -av /Data/HumanBase spark2:/Data/
# rsync -av /Data/HumanBase spark3:/Data/
# rsync -av /Data/HumanBase spark4:/Data/
# rsync -av /Data/HumanBase spark5:/Data/
```
----------------
# Validation by Alignment
We will now validate our reference genome by alignment with the files downloaded. Follow the <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/execution.md>Next Step</a>.
