|Title |  Genomic Assembly |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | Spark, Hadoop, HDFS |
|Date | 11/10/2018 |

# Genomic Assembly

## 5 Nodes privision 

Create 5 nodes with 4 CPU, 32G RAM, 100G disk with CentOS. 
```
$ slcli vs create --datacenter=hou02 --hostname=spark1 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark2 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark3 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark4 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark5 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
```

Checking all nodes privision

```
$ slcli vs list 

:..........:...............:................:...............:............:........:
:    id    :    hostname   :   primary_ip   :   backend_ip  : datacenter : action :
:..........:...............:................:...............:............:........:
: 64972694 :     spark1    : 184.173.63.164 : 10.77.147.231 :   hou02    :   -    :
: 64972722 :     spark2    : 184.173.63.165 : 10.77.147.210 :   hou02    :   -    :
: 64972734 :     spark3    : 184.173.63.162 : 10.77.147.229 :   hou02    :   -    :
: 64972750 :     spark4    : 184.173.63.166 : 10.77.147.239 :   hou02    :   -    :
: 64972762 :     spark5    :  50.23.42.89   : 10.77.147.241 :   hou02    :   -    :
:..........:...............:................:...............:............:........:
```

## Setup DNS on all 5 nodes

To easily ssh with the name instead of the IP addresses, we will setup the DNS. You need to add the spark1.mids.com as well.
```
# vi /etc/hosts

127.0.0.1       localhost.localdomain localhost
184.173.63.164  spark1.mids.com spark1
184.173.63.165  spark2.mids.com spark2
184.173.63.162  spark3.mids.com spark3
184.173.63.166  spark4.mids.com spark4
50.23.42.89     spark5.mids.com spark5
```

## Setup passwordless ssh
The idea is to `ssh` without password between nodes. spark1 must be able to `ssh spark1`, `ssh spark2` and `ssh spark3`. You already know by now that to ssh using the name requires you to set up at /etc/hosts. Without password, it requires you to setup ssh-keygen generation. Follow the steps below in spark1 node. We then scopy all ssh private and public keys to other nodes. You'll need to enter the password when asked when copying. 

Before you copy, make sure you test from spark1 to `ssh spark2`, etc. When ECDSA keys asks to connect, enter yes so that the ECDSA key will be updated in `known_hosts`. Once we have everything `id_rsa`, `id_rsa.pub`, `authorized_keys` and `known_hosts` in spark1 node, we will copy everything into all other nodes so that they can instantiate the communication without password. 

```
# ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
# chmod 600 ~/.ssh/authorized_keys

# ssh spark1 
exit
# ssh spark2
exit
# ssh spark3
exit
# ssh spark4
exit
# ssh spark5
exit

# scp ~/.ssh/* root@184.173.63.165:/root/.ssh/
# scp ~/.ssh/* root@184.173.63.162:/root/.ssh/
# scp ~/.ssh/* root@184.173.63.166:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.89:/root/.ssh/

# for i in spark1 spark2 spark3 spark4 spark5; do ssh-copy-id $i; done
```
ssh-copy-id will copy `id_rsa.pub` key to authorized_keys file (will be created) in other nodes. So when it tries to establish the connection, it will first ask the password of the node it's sshing into.

#### Check if all 5 nodes can communicate without password from spark1 node

```
# vi test.sh
```
Copy the following script
```
#!/bin/bash

# Edit node list
nodes="spark1 spark2 spark3 spark4 spark5"

# Test ssh configuration
for i in $nodes
do for j in $nodes
 do echo -n "Testing ${i} to ${j}: "
 ssh  ${i} "ssh ${j} date"
 done
done
```
Run the script
```
# chmod 755 test.sh
# ./test.sh

Testing spark1 to spark1: Sat Nov 10 20:51:19 CST 2018
Testing spark1 to spark2: Sat Nov 10 20:51:19 CST 2018
Testing spark1 to spark3: Sat Nov 10 20:51:20 CST 2018
Testing spark1 to spark4: Sat Nov 10 20:51:21 CST 2018
Testing spark1 to spark5: Sat Nov 10 20:51:22 CST 2018
Testing spark2 to spark1: Sat Nov 10 20:51:22 CST 2018
Testing spark2 to spark2: Sat Nov 10 20:51:23 CST 2018
Testing spark2 to spark3: Sat Nov 10 20:51:24 CST 2018
Testing spark2 to spark4: Sat Nov 10 20:51:25 CST 2018
Testing spark2 to spark5: Sat Nov 10 20:51:26 CST 2018
Testing spark3 to spark1: Sat Nov 10 20:51:26 CST 2018
Testing spark3 to spark2: Sat Nov 10 20:51:27 CST 2018
Testing spark3 to spark3: Sat Nov 10 20:51:28 CST 2018
Testing spark3 to spark4: Sat Nov 10 20:51:29 CST 2018
Testing spark3 to spark5: Sat Nov 10 20:51:30 CST 2018
Testing spark4 to spark1: Sat Nov 10 20:51:30 CST 2018
Testing spark4 to spark2: Sat Nov 10 20:51:31 CST 2018
Testing spark4 to spark3: Sat Nov 10 20:51:31 CST 2018
Testing spark4 to spark4: Sat Nov 10 20:51:33 CST 2018
Testing spark4 to spark5: Sat Nov 10 20:51:33 CST 2018
Testing spark5 to spark1: Sat Nov 10 20:51:34 CST 2018
Testing spark5 to spark2: Sat Nov 10 20:51:34 CST 2018
Testing spark5 to spark3: Sat Nov 10 20:51:35 CST 2018
Testing spark5 to spark4: Sat Nov 10 20:51:36 CST 2018
Testing spark5 to spark5: Sat Nov 10 20:51:37 CST 2018
```

# Install Java, SBT and Spark (on all nodes)

Note: In Ubuntu, you can download DEB packages using the `apt-get` package manager. Ubuntu also updates its applicatin packages frequently which makes the latest update easily available but it also makes the Ubuntu less stable because of the frequency of the update. Meanwhile, in CentOS, you have to use the `yum` command to download and install RPM packages from the central repository. The downside of stable CentOS is you have to install latest software manually if necessary. 

rpm = redhat package manager  

Install SBT
```
# curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
# yum install -y java-1.8.0-openjdk-devel sbt git
```
Set Jave path in `~/.bash_profile`. After you set the export path, you need to `source` it so that the path becomes activated. To test later, check the java version.
```
# echo export JAVA_HOME=\"$(readlink -f $(which java) | grep -oP '.*(?=/bin)')\" >> /root/.bash_profile
# source /root/.bash_profile

# $JAVA_HOME/bin/java -version

openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```
Install Spark
```
# curl https://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.7.tgz | tar -zx -C /usr/local --show-transformed --transform='s,/*[^/]*,spark,'
```
Set Spark path
```
# echo export SPARK_HOME=\"/usr/local/spark\" >> /root/.bash_profile
# source /root/.bash_profile
```
# Configure Spark (on spark1)

```
# cd $SPARK_HOME/conf/
# vi slaves

spark1
spark2
spark3
```

# Copy files to spark1
We will copy moby10b.txt and src directory into our spark1 node and check their directory in our node. 
```
# cd /root
# git clone https://github.com/MIDS-scaling-up/coursework.git
```
moby10b.txt and src directory
```
/root/coursework/week6/hw/apache_spark_introduction/moby10b.txt
/root/coursework/week6/hw/apache_spark_introduction/src/spark
```

# Start Spark
we will start spark from spark1 as master node. There are a few script you should be familiar with.  
```
sbin/start-master.sh - Starts a master instance on the machine the script is executed on
sbin/start-slaves.sh - Starts a slave instance on each machine specified in the conf/slaves file
sbin/start-all.sh - Starts both a master and a number of slaves as described above
sbin/stop-master.sh - Stops the master that was started via the bin/start-master.sh script
sbin/stop-slaves.sh - Stops all slave instances on the machines specified in the conf/slaves file
sbin/stop-all.sh - Stops both the master and the slaves as described above
```

Start the master and check if it's working at `http://<master_ip>:8080/` = `http://50.97.252.101:8080`
```
# $SPARK_HOME/sbin/start-master.sh

starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark-root-org.apache.spark.deploy.master.Master-1-spark1.mids.com.out
```

<p align="center">
<img src="img/spark.png" width="800"></p>
<p align="center">Figure 1. Spark</p>

Start the slaves, refresh the browser and check if all workers are now active. 
```
# $SPARK_HOME/sbin/start-slaves.sh
```

<p align="center">
<img src="img/spark_workers.png" width="800"></p>
<p align="center">Figure 2. Spark with 3 workers</p>

# Calculate Pi
```
# $SPARK_HOME/bin/run-example SparkPi
```

#### Question 1: What value of PI do you get? Why is the value not "exact"? For a hint, see `$SPARK_HOME/examples/src/main/python/pi.py` 

```
Pi is roughly 3.1411357056785283
```

## Going into Spark Shell

```
# $SPARK_HOME/bin/spark-shell

...
...
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.1.1
      /_/
         
Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_181)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```
To learn more about Apache Spark API  
https://spark.apache.org/docs/latest/quick-start.html

We're going to read `moby10b.txt` in spark and analzye afterwards. 
```
scala> val textFile = sc.textFile("/root/coursework/week6/hw/apache_spark_introduction/moby10b.txt")
```

#### Question 2: How many lines does the file have?
```
scala> textFile.count()
res3: Long = 23244
```
#### Question 3: What is the first line?
```
scala> textFile.first()
res4: String = **The Project Gutenberg Etext of Moby Dick, by Herman Melville**
```

#### Question 4: How many lines contain the text "whale"?
```
scala> val linesWithWhale = textFile.filter(line => line.contains("whale"))
scala> linesWithWhale.count()
res5: Long = 1276
```

#### Question 5: How many output files (ignore _SUCCESS file) does Spark write when the file RDD (file.saveAsTextFile( outputDirectory)) is written to the output directory?

Answer: Spark writes 2 files 
```
part-00000  part-00001  _SUCCESS
```

**Execution**

Compile the java file in the `src` directory. Use the moby10b.txt as an input and specify the output directory. You must run the command from the `src` directory. 

```
# cd /root/coursework/week6/hw/apache_spark_introduction/src/

# javac -cp .:$SPARK_HOME/jars/* spark/SparkJava8Example.java
```

##### Sample 
```
# java -cp .:$SPARK_HOME/jars/* spark.SparkJava8Example /root/coursework/week6/hw/apache_spark_introduction/moby10b.txt /<yourOutputDirectory>
```

##### Real command
```
# java -cp .:$SPARK_HOME/jars/* spark.SparkJava8Example /root/coursework/week6/hw/apache_spark_introduction/moby10b.txt /root/hw6
```

#### Question 5b: Change the line: JavaRDD file = sc.textFile(inputFile); to: JavaRDD file = sc.textFile(inputFile,1); and rerun the sample. How many outfiles are created when the RDD is saved? Explain the difference.

Only one file is created, excluding `_SUCCESS`. 
```
part-00000  _SUCCESS
```
The RDD (Resilient Distributed Dataset) by definition tries to save files in partitions. So when the file is large, it will partition into several duplicates. Here we changed the parameters into 1, so it only saved the file in one dataset. 

**Execution**

You need to change the line in SparkJava8Example.java file. 
```
# cd /root/coursework/week6/hw/apache_spark_introduction/src/spark/

# grep "sc.textFile" SparkJava8Example.java
			JavaRDD<String> file = sc.textFile(inputFile);
   
# vi SparkJava8Example.java
/sc.textFile
<press> i

sc.textFile(inputfile,1)

<press> esc
:wq!
```
You also need to remake the java class file. So remove the existing class file first. After that, go one level up to `src` directory. 
```
# rm SparkJava8Example.class
# cd ..
```
You need to use a totally different directory even if you deleted the previous files in the same directory `hw6`.  

```
# javac -cp .:$SPARK_HOME/jars/* spark/SparkJava8Example.java
# java -cp .:$SPARK_HOME/jars/* spark.SparkJava8Example /root/coursework/week6/hw/apache_spark_introduction/moby10b.txt /root/hw6_1
```

#### Question 6: The text file moby10b.txt is the The Project Gutenberg Etext of Moby Dick. This file contains a Gutenberg preamble. Write a job that removes or filters out all the lines until the text "MOBY DICK; OR THE WHALE" and write this new RDD to disk. How many lines does our filtered file have?

#### Question 7: Using your filtered RDD, count the number of words. Non-letter characters (only a-z should be included) should be removed.

#### Question 8: Find the number of times each letter is used. The results should be sorted alphabetically, from a to z.

#### Question 9: Find the number of times each letter is used. The results should be sorted from most frequent to least frequent.

#### Question 10: Rewrite the filtered file such that each line is mirrored or reversed. What are the first 20 lines of the mirrored RDD?

# Jupyter Notebook, Pyspark

In order to answer question 6 to 10, I thought of analyzing moby10b.txt in pyspark. So I need to install python in Centos OS first. First Centos also has its own software collection. To install those, I first need to install SCL release (Software Collection). 

```
# sudo yum install centos-release-scl
# sudo yum install rh-python36 bzip2 pyspark
# python --version

Python 2.7.5
```

Since python 2.7.5 is the default version, we will update the version by bash. 
```
# scl enable rh-python36 bash
# python --version

Python 3.6.3
```

### Setting up Jupyter Notebook

You need to set up the configuration file for jupyter notebook to work in your browser. Remember, you're launching jupyter-notebook from the server. But you will get access to jupyter-notebook from your local machine browser. Jupyter requires Python 2.7 or 3.5 and higher. 

If you have python2
```
# pip install --upgrade pip
# python -m pip install --upgrade pip
# python -m pip install jupyter
```
For python3
```
python3 -m pip install --upgrade pip
python3 -m pip install jupyter
```
To have a configuration file
```
# jupyter notebook --generate-config

Writing default config to: /root/.jupyter/jupyter_notebook_config.py
```
Modifying the configuration file
```
# cd /root/.jupyter/
# vi jupyter_notebook_config.py
```
Copy the following script and paste it the jupyter_notebook_config.py
```
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 8123
```
Launch jupyter-notebook
```
# jupyter-notebook
```

Go to your browser 
```
50.97.252.101:8123/?token ....
```

### Setting up Pyspark in Jupyter Notebook

I'd like to launch pyspark in jupyter notebook. There are two options to use `pyspark` in jupyter notebook.  
- Option1: When you'd call `# pyspark`, it will automatically launch jupyter-notebook and you can use `pyspark` in jupyter-notebook. It's fast.  
- Option2: You launch `# jupyter-notebook` and `import findspark` library. This is good when you want to work in different IDE. You just need to do `# pip install findspark` for later use in the jupyter notebook. 

I played around with the option1 but didn't succeed with the moby10b.txt. I detailed the procedure for future reference in optional installation below. So I tried with option2 and it worked. So Here I will continue with the option2. Checking the pyspark from earlier installation. 

https://github.com/kckenneth/Spark/blob/master/pyspark_setup.md

----------------
# Optional Installation
### I. Option1. Pyspark in Jupyter Notebook

```
# pyspark --version

Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.1.1
      /_/

Using Python version 3.6.3 (default, Mar 20 2018 13:50:41)
SparkSession available as 'spark'.
```
Once the installation is checked, we need to modify the export path of the `pyspark`. 
```
# vi /root/.bashrc
```
Copy and paste the following 
```
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
```
To activate the .bashrc
```
# source /root/.bashrc
```
Launch the pyspark
```
# pyspark

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8123/?token=d23a6df8xxxxxxxxx
```
Change the localhost to my primary IP address: `50.97.252.101`. 

### II. Anaconda (Optional Installation) 

If you're starting from scratch, you'll need to install several apps. 

1. Anaconda3 (which requires bzip2)

```
# wget https://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh
# bash Anaconda3-4.3.0-Linux-x86_64.sh
```
Accept license agreeement, yes, and yes to the last question on 'prepend the PATH'. You need to start a new terminal, log into your account. This time if you call python by `# python` python will launch from Anaconda path. 

