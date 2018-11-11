# Human genome 38

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

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2:17.446s
[INFO] Finished at: Sun Nov 11 09:47:14 CST 2018
[INFO] Final Memory: 32M/241M
[INFO] ------------------------------------------------------------------------
```
------------

```
wget ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/data/NA12750/sequence_read/ERR000589_1.filt.fastq.gz
wget ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/phase3/data/NA12750/sequence_read/ERR000589_2.filt.fastq.gz
# uncompress
gzip -d ERR000589_1.filt.fastq.gz
gzip -d ERR000589_2.filt.fastq.gz
# push to HDFS
hdfs dfs -copyFromLocal ERR000589_1.filt.fastq /ERR000589_1.filt.fastq
hdfs dfs -copyFromLocal ERR000589_2.filt.fastq /ERR000589_2.filt.fastq
```
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
Once you're in ftp mode, enter these commands below to ensure non-interactive and binary modes. When you're in `ftp>` prompt, do the following. 
```
prompt
binary
cd bundle
cd hg38
mget *
```
