# Human genome 38

This is the 38 verion of the human genome. We will download two zipped files and align to our standard human genome 38. Make sure you get the raw reads from the 1000 Genomes Project and insert into HDFS. These will be the ones we will be aligning.

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
