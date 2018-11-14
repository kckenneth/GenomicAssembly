|Title |  Genomic Assembly |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | Spark, Hadoop, HDFS |
|Date | 11/10/2018 |

# Genomic Assembly

<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/README.md><< Introduction</a>
**== 1. Spark setup**    
<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_hadoop.md>>> 2. Hadoop setup</a>      
<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_HG38.md>>> 3. Human Genome in HDFS</a>    
<a href=https://github.com/kckenneth/GenomicAssembly/blob/master/execution.md>>> 4. Alignment</a>  

## 10 Nodes privision 

Create 10 nodes with 4 CPU, 32G RAM, 100G disk with CentOS. 
```
$ slcli vs create --datacenter=hou02 --hostname=spark1 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark2 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark3 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark4 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark5 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark6 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark7 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark8 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark9 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark10 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
```

Checking all nodes provision

```
$ slcli vs list 

:..........:...............:................:...............:............:........:
:    id    :    hostname   :   primary_ip   :   backend_ip  : datacenter : action :
:..........:...............:................:...............:............:........:
: 65044368 :     spark1    :  50.23.42.89   : 10.77.147.231 :   hou02    :   -    :
: 65044492 :    spark10    :  50.23.42.83   : 10.77.147.230 :   hou02    :   -    :
: 65044382 :     spark2    : 184.173.63.164 : 10.77.147.210 :   hou02    :   -    :
: 65044390 :     spark3    :  50.23.42.86   : 10.77.147.239 :   hou02    :   -    :
: 65044396 :     spark4    : 184.173.63.166 : 10.77.147.229 :   hou02    :   -    :
: 65044402 :     spark5    :  50.23.42.87   : 10.77.147.241 :   hou02    :   -    :
: 65044426 :     spark6    : 184.173.63.165 : 10.77.147.214 :   hou02    :   -    :
: 65044444 :     spark7    :  50.23.42.84   : 10.77.147.217 :   hou02    :   -    :
: 65044454 :     spark8    :  50.23.42.85   : 10.77.147.219 :   hou02    :   -    :
: 65044476 :     spark9    :  50.23.42.82   : 10.77.147.224 :   hou02    :   -    :
:..........:...............:................:...............:............:........:
```

## Setup DNS on all nodes

To easily ssh with the name instead of the IP addresses, we will setup the DNS. You need to add the spark1.mids.com as well. It's important you'd use the private IP address. I tried using the public and it gives me an error by the time I got into yarn resource manager. 
```
# vi /etc/hosts

127.0.0.1      localhost.localdomain localhost
10.77.147.231  spark1.mids.com spark1
10.77.147.210  spark2.mids.com spark2
10.77.147.239  spark3.mids.com spark3
10.77.147.229  spark4.mids.com spark4
10.77.147.241  spark5.mids.com spark5
10.77.147.214  spark6.mids.com spark6
10.77.147.217  spark7.mids.com spark7
10.77.147.219  spark8.mids.com spark8
10.77.147.224  spark9.mids.com spark9
10.77.147.230  spark10.mids.com spark10
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
...
...
# ssh spark10
exit

# scp ~/.ssh/* root@50.23.42.89:/root/.ssh/
# scp ~/.ssh/* root@184.173.63.164:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.86:/root/.ssh/
# scp ~/.ssh/* root@184.173.63.166:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.87:/root/.ssh/
# scp ~/.ssh/* root@184.173.63.165:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.84:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.85:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.82:/root/.ssh/
# scp ~/.ssh/* root@50.23.42.83:/root/.ssh/

# for i in spark1 spark2 spark3 spark4 spark5; do ssh-copy-id $i; done
```
ssh-copy-id will copy `id_rsa.pub` key to authorized_keys file (will be created) in other nodes. So when it tries to establish the connection, it will first ask the password of the node it's sshing into.

#### Check if all nodes can communicate without password from spark1 node

```
# vi test.sh
```
Copy the following script
```
#!/bin/bash

# Edit node list
nodes="spark1 spark2 spark3 spark4 spark5 spark6 spark7 spark8 spark9 spark10"

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

Testing spark1 to spark1: Mon Nov 12 10:29:04 CST 2018
Testing spark1 to spark2: Mon Nov 12 10:29:04 CST 2018
...
...
Testing spark10 to spark9: Mon Nov 12 10:30:21 CST 2018
Testing spark10 to spark10: Mon Nov 12 10:30:22 CST 2018
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

openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
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
spark4
spark5
spark6
spark7
spark8
spark9
spark10
```

# Setup Hadoop

Next step, we will set up hadoop in all nodes. Go <a href=https://github.com/kckenneth/GenomicAssembly/blob/master/setup_hadoop.md>Here</a>
