[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/big-data-europe/Lobby)

# Changes

Version 2.0.0 introduces uses wait_for_it script for the cluster startup

# Hadoop Docker

## Supported Hadoop Versions
See repository branches for supported hadoop versions

## Quick Start

To deploy an example HDFS cluster, run:
```
  docker-compose up
```

# note
docker-compose.yml misses port mapping for all services, but namenode. Manually added RM 8080
See docker-compose.yml



Run example wordcount job:
```
  make wordcount
```
## note ghidosan 2020.01.29: command fails with error:
quote... docker run --network docker-hadoop_default --env-file hadoop.env bde2020/hadoop-base:2020.01.29-18.27 hdfs dfs -mkdir -p /input/
Unable to find image 'bde2020/hadoop-base:2020.01.29-18.27' locally
docker: Error response from daemon: manifest for bde2020/hadoop-base:2020.01.29-18.27 not found: manifest unknown: manifest unknown." ...unquote

bde2020/hadoop-base is a docker image (see https://hub.docker.com/r/bde2020/hadoop-base), but if I try to pull it it return 'alredy exists'

#FIX
Run '~/Documents/GitHub/docker-hadoop$ make -f Makefile': it basically update Debian and download / extract hadoop 3.1 under /opt/hadoop on container 
then run 'make wordcount'

quote...
~/Documents/GitHub/docker-hadoop$ make -f Makefile 
docker build -t bde2020/hadoop-base:2020.01.29-18.27 ./base
Sending build context to Docker daemon  8.192kB
Step 1/21 : FROM debian:9
9: Pulling from library/debian
146bd6a88618: Pull complete 
Digest: sha256:85c4668abb4f26e913152ba8fd04fca5f1c2345d3e2653855e6bb0acf461ed50
Status: Downloaded newer image for debian:9
-- omitted ---
Step 5/21 : ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
--- omitted --
Step 6/21 : RUN curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS
--- omitted ---
Step 7/21 : RUN gpg --import KEYS
--- omitted ---
Step 8/21 : ENV HADOOP_VERSION 3.1.3
--- omitted ---
Step 9/21 : ENV HADOOP_URL https://www.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz
--- omitted ----
Step 10/21 : RUN set -x     && curl -fSL "$HADOOP_URL" -o /tmp/hadoop.tar.gz     && curl -fSL "$HADOOP_URL.asc" -o /tmp/hadoop.tar.gz.asc     && gpg --verify /tmp/hadoop.tar.gz.asc     && tar -xvf /tmp/hadoop.tar.gz -C /opt/     && rm /tmp/hadoop.tar.gz*
*--- omitted ---
Step 11/21 : RUN ln -s /opt/hadoop-$HADOOP_VERSION/etc/hadoop /etc/hadoop
--- omitted ---
Step 12/21 : RUN mkdir /opt/hadoop-$HADOOP_VERSION/logs
Step 13/21 : RUN mkdir /hadoop-data
Step 14/21 : ENV HADOOP_PREFIX=/opt/hadoop-$HADOOP_VERSION
Step 15/21 : ENV HADOOP_CONF_DIR=/etc/hadoop
Step 16/21 : ENV MULTIHOMED_NETWORK=1
Step 17/21 : ENV USER=root
Step 18/21 : ENV PATH $HADOOP_PREFIX/bin/:$PATH
Step 19/21 : ADD entrypoint.sh /entrypoint.sh
Step 20/21 : RUN chmod a+x /entrypoint.sh
Step 21/21 : ENTRYPOINT ["/entrypoint.sh"]
--- omitted----
...unquote





















===================================================================================

Or deploy in swarm:
```
docker stack deploy -c docker-compose-v3.yml hadoop
```

`docker-compose` creates a docker network that can be found by running `docker network list`, e.g. `dockerhadoop_default`.

Run `docker network inspect` on the network (e.g. `dockerhadoop_default`) to find the IP the hadoop interfaces are published on. Access these interfaces with the following URLs:

* Namenode: http://<dockerhadoop_IP_address>:9870/dfshealth.html#tab-overview
* History server: http://<dockerhadoop_IP_address>:8188/applicationhistory
* Datanode: http://<dockerhadoop_IP_address>:9864/
* Nodemanager: http://<dockerhadoop_IP_address>:8042/node
* Resource manager: http://<dockerhadoop_IP_address>:8088/

## Configure Environment Variables

The configuration parameters can be specified in the hadoop.env file or as environmental variables for specific services (e.g. namenode, datanode etc.):
```
  CORE_CONF_fs_defaultFS=hdfs://namenode:8020
```

CORE_CONF corresponds to core-site.xml. fs_defaultFS=hdfs://namenode:8020 will be transformed into:
```
  <property><name>fs.defaultFS</name><value>hdfs://namenode:8020</value></property>
```
To define dash inside a configuration parameter, use triple underscore, such as YARN_CONF_yarn_log___aggregation___enable=true (yarn-site.xml):
```
  <property><name>yarn.log-aggregation-enable</name><value>true</value></property>
```

The available configurations are:
* /etc/hadoop/core-site.xml CORE_CONF
* /etc/hadoop/hdfs-site.xml HDFS_CONF
* /etc/hadoop/yarn-site.xml YARN_CONF
* /etc/hadoop/httpfs-site.xml HTTPFS_CONF
* /etc/hadoop/kms-site.xml KMS_CONF
* /etc/hadoop/mapred-site.xml  MAPRED_CONF

If you need to extend some other configuration file, refer to base/entrypoint.sh bash script.
