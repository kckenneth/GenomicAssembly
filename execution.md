# Validation [Running alignment]

<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_HG38.md><< Human Genome setup</a>  

### Setting the Yarn in Spark environment
Before we validate our alignment, we need to first understand the spark and yarn. Basically, Spark is a built-in RDD we installed. On top of the spark local file system, we installed Hadoop distributed file system (HDFS). Now we're going to submit our job from spark using `spark-submit` using `YARN` (Yet Another Resource Navigator or Negotiator). Yarn is a resource manager for HDFS, not for spark. So we're crossing the two systems here. Spark must know where is the resource manager `Yarn` for HDFS, which is basically a configuration file. So we need to set the hadoop configuration path in spark environment. 

```
# cd /usr/local/spark/conf
# vi spark-env.sh.template
```
Copy and paste the following anywhere. There are many commented description.
```
export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop/"
```
Rename the spark environment file
```
# mv spark-env.sh.template spark-env.sh
```

### Valide the alignment

Make sure that BWA Spark is correctly installed and that the command below runs to the end. If you are successful, you will see a file called FullOutput.sam (4.67 GB) created in the /user/hadoop/OUTPUT_DIR directory of your HDFS. Set the num-executors to the number of nodes in your cluster. Set executor-cores to the number of cores in each VM in your cluster -- and if you see out of memory errors, go down. The --index parameter should contain the base name of your reference genome files. Make sure the path to the SparkBWA jar file is correct in the line below.

```
# $SPARK_HOME/bin/spark-submit --class com.github.sparkbwa.SparkBWA --master yarn --driver-memory 1500m --executor-memory 15g --executor-cores 4 --verbose --num-executors 10 /root/SparkBWA/target/SparkBWA-0.2.jar -m -r -p --index /Data/HumanBase/Homo_sapiens_assembly38.fasta.gz -n 32  -w "-R @RG\tID:foo\tLB:bar\tPL:illumina\tPU:illumina\tSM:ERR000589" /ERR000589_1.filt.fastq /ERR000589_2.filt.fastq /Data/HumanBase/
```

```
	 client token: N/A
	 diagnostics: AM container is launched, waiting for AM container to Register with RM
	 ApplicationMaster host: N/A
	 ApplicationMaster RPC port: -1
	 queue: default
	 start time: 1542144791497
	 final status: UNDEFINED
	 tracking URL: http://spark1:8088/proxy/application_1542062922100_20899/
	 user: root
```
There would be a bunch of logs showing up. One of the log I copied and pasted here. You can go to the following link in your browser to check the status of your job. 
```
http://50.23.42.89:8088/proxy/application_1542062922100_20899/
```



18/11/13 15:44:15 INFO spark.SparkContext: Successfully stopped SparkContext
18/11/13 15:44:15 INFO util.ShutdownHookManager: Shutdown hook called
18/11/13 15:44:15 INFO util.ShutdownHookManager: Deleting directory /tmp/spark-e1e5a3d3-b39e-4f4f-8070-64175cb2916a


### To move the output from HDFS to local filesystem
```
# hdfs dfs -copyToLocal Output_ERR000589/* ./
```
