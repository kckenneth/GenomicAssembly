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
The idea is to `ssh` without password between nodes. spark1 must be able to `ssh spark1`, `ssh spark2` and `ssh spark3`. You already know by now that to ssh using the name requires you to set up at /etc/hosts. Without password, it requires you to setup ssh-keygen generation.

```
# ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
# chmod 600 ~/.ssh/authorized_keys

# scp ~/.ssh/* root@50.97.252.103:/root/.ssh/
# scp ~/.ssh/* root@50.97.252.102:/root/.ssh/

# for i in spark1 spark2 spark3; do ssh-copy-id $i; done
```
ssh-copy-id will copy id_rsa.pub key to authorized_keys file (will be created) in other nodes. So when it tries to establish the connection, it will first ask the password of the node it's sshing into.
