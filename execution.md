# Validation [Running alignment]

<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_HG38.md><< Human Genome setup</a>  

Make sure that BWA Spark is correctly installed and that the command below runs to the end. If you are successful, you will see a file called FullOutput.sam (4.67 GB) created in the /user/hadoop/OUTPUT_DIR directory of your HDFS. Set the num-executors to the number of nodes in your cluster. Set executor-cores to the number of cores in each VM in your cluster -- and if you see out of memory errors, go down. The --index parameter should contain the base name of your reference genome files. Make sure the path to the SparkBWA jar file is correct in the line below.

```
# $SPARK_HOME/spark-submit --class com.github.sparkbwa.SparkBWA --master yarn --driver-memory 1500m --executor-memory 15g --executor-cores 4 --verbose --num-executors 10 /root/SparkBWA/target/SparkBWA-0.2.jar -m -r -p --index /Data/HumanBase/Homo_sapiens_assembly38.fasta.gz -n 32  -w "-R @RG\tID:foo\tLB:bar\tPL:illumina\tPU:illumina\tSM:ERR000589" /ERR000589_1.filt.fastq /ERR000589_2.filt.fastq /Data/HumanBase/
```
