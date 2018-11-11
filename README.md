# Genomic Assembly

**Date**: 2018 November 11  
**Author**: Kenneth Chen

In this project, we will setup 5 spark nodes with 4CPU, 32GB RAM, 100GB disk in CentOS. On top of Spark, we will setup HDFS across all 5 nodes. Once HDFS is ready, download human reference genome `hg38` and distribute across all nodes. We will finally validate the genome with some files downloaded by alignment using spark BWA. Follow the steps shown below. 

1. <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_spark.md>Spark setup</a>
2. <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_hadoop.md>Hadoop setup</a>
3. <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_HG38.md>Human Genome in HDFS</a>
4. <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/execution.md>Alignment Validation<a>
