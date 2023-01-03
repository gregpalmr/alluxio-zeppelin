# alluxio-zeppelin
Build an Apache Zeppelin server that integrates with the Alluxio data orchestration platform

## Introduction

The Apache Zeppelin is a multi-language notebook for working with your data lake data. It supports Spark Scala, SparkSQL, PySpark, SparkR and other integrations. For more information about Apache Zeppelin, see:

- https://zeppelin.apache.org/docs/0.10.0
     
Zeppelin supports custom interpreters that allow you to interact with various sub-systems, including interacting with the Alluxio data orchestration platform. Alluxio allows users to bring data closer to compute workloads across clusters, regions, clouds, and countries. For more information about Alluxio, see:

- https://www.alluxio.io/data-orchestration

This git repo provides instructions for building an Apache Zeppelin server that can interact with Alluxio using the Alluxio interpreter.

## Zeppelin Alluxio Interpreter

The Alluxio interpreter that is bundled with the Zeppelin server allows users to run all the Alluxio filesystem commands that are available through Alluxio's command line interface program (CLI). Here are the Alluxio CLI commands that can be executed with the Zeppelin Alluxio interpreter:

|Operation                |Syntax                                           |Description                                              |
|-------------------------|-------------------------------------------------|---------------------------------------------------------|
|cat	|cat "path"	|Print the content of the file to the console.|
|chgrp	|chgrp "group" "path"	|Change the group of the directory or file.|
|chmod	|chmod "permission" "path"	|Change the permission of the directory or file.|
|chown	|chown "owner" "path"	|Change the owner of the directory or file.|
|copyFromLocal	|copyFromLocal "source path" "remote path"	|Copy the specified file specified by "source path" to the path specified by "remote path". This command will fail if "remote path" already exists.|
|copyToLocal	|copyToLocal "remote path" "local path"	|Copy the specified file from the path specified by "remote path" to a local destination.|
|count	|count "path"	|Display the number of folders and files matching the specified prefix in "path".|
|du	|du "path"	|Display the size of a file or a directory specified by the input path.|
|fileInfo	|fileInfo "path"	|Print the information of the blocks of a specified file.|
|free	|free "path"	|Free a file or all files under a directory from Alluxio. If the file/directory is also in under storage, it will still be available there.|
|getCapacityBytes	|getCapacityBytes	|Get the capacity of the AlluxioFS.|
|getUsedBytes	|getUsedBytes	|Get number of bytes used in the AlluxioFS.|
|load	|load "path"	|Load the data of a file or a directory from under storage into Alluxio.|
|loadMetadata	|loadMetadata "path"	|Load the metadata of a file or a directory from under storage into Alluxio.|
|location	|location "path"	|Display a list of hosts that have the file data.|
|ls	|ls "path"	|List all the files and directories directly under the given path with information such as size.|
|mkdir	|mkdir "path1" ... "pathn"	|Create directory(ies) under the given paths, along with any necessary parent directories. Multiple paths separated by spaces or tabs. This command will fail if any of the given paths already exist.|
|mount	|mount "path" "uri"	|Mount the underlying file system path "uri" into the Alluxio namespace as "path". The "path" is assumed not to exist and is created by the operation. No data or metadata is loaded from under storage into Alluxio. After a path is mounted, operations on objects under the mounted path are mirror to the mounted under storage.|
|mv	|mv "source" "destination"	|Move a file or directory specified by "source" to a new location "destination". This command will fail if "destination" already exists.|
|persist	|persist "path"	|Persist a file or directory currently stored only in Alluxio to the underlying file system.|
|pin	|pin "path"	|Pin the given file to avoid evicting it from memory. If the given path is a directory, it recursively pins all the files contained and any new files created within this directory.|
|report	|report "path"	|Report to the master that a file is lost.|
|rm	|rm "path"	|Remove a file. This command will fail if the given path is a directory rather than a file.|
|setTtl	|setTtl "time"	|Set the TTL (time to live) in milliseconds to a file.|
|tail	|tail "path"	|Print the last 1KB of the specified file to the console.|
|touch	|touch "path"	|Create a 0-byte file at the specified location.|
|unmount	|unmount "path"	|Unmount the underlying file system path mounted in the Alluxio namespace as "path". Alluxio objects under "path" are removed from Alluxio, but they still exist in the previously mounted under storage.|
|unpin	|unpin "path"	|Unpin the given file to allow Alluxio to evict this file again. If the given path is a directory, it recursively unpins all files contained and any new files created within this directory.|
|unsetTtl	|unsetTtl	|Remove the TTL (time to live) setting from a file.|

For more information about he Zeppelin Alluxio interpreter, see:

- https://zeppelin.apache.org/docs/0.10.0/interpreter/alluxio.html
     
## Build a Docker image

If you want to run the Zeppelin server in a Docker container or in a Kubernetes Pod, you can build the Docker image file using the following instructions.

### Step 1. Make sure you have Docker desktop installed and configured.

- For Windows computers, follow the instructions found here: https://docs.docker.com/desktop/install/windows-install

- For MacOS computers, follow the instructions found here: https://docs.docker.com/desktop/install/mac-install

- For Linux computers, follow the instructions here: https://docs.docker.com/desktop/install/linux-install/

### Step 2. Download the Docker build (Dockerfile) script

Download the Docker build script named Dockerfile.zeppelin from this git repo. Either copy and paste the contents into a file on your computer, or use the following WGET or CURL command:

     wget https://raw.githubusercontent.com/gregpalmr/alluxio-zeppelin/main/Dockerfile.zeppelin
     
     curl -O https://raw.githubusercontent.com/gregpalmr/alluxio-zeppelin/main/Dockerfile.zeppelin
     
### Step 3. Build the Docker image

Run the following Docker build command to build a Zeppelin Docker image that includes the correct integration with Alluxio.

     docker build -f Dockerfile.zeppelin -t alluxio/zeppelin:0.10.0_2.9.0 ./

If you want to rebuild the image again, without starting from the beginning, you can run the command with the --no-cache option:

     docker build --no-cache -f Dockerfile.zeppelin -t alluxio/zeppelin:0.10.0_2.9.0 ./
     
### Step 4. Run the Docker image

**Run Manually**

To run the Docker image manually, use the following commands and supply the correct hostname or IP address of the Alluxio master node(s).

When running against a non-HA configured Alluxio cluster, use this command:

     docker run -p 8080:8080 --rm \
                 -v $PWD/tmp-zeppelin-logs:/opt/zeppelin/logs \
                 -v $PWD/tmp-zeppelin-notebooks:/opt/zeppelin/notebook \
                 -e ZEPPELIN_IN_DOCKER=true \
                 -e ALLUXIO_MASTER_HOSTNAME="<alluxio_master_ip_addr>" \
                 -e ALLUXIO_MASTER_PORT="19998" \
                 -e SPARK_MASTER="spark://<spark_master_ip_addr>:7077" \
                 --name alluxio-zeppelin \
                 alluxio/zeppelin:0.10.0_2.9.0
 
 When running against a Raft-based HA configured Alluxio cluster, use this command:
 
     docker run -p 8080:8080 --rm \
                 -v $PWD/tmp-zeppelin-logs:/opt/zeppelin/logs \
                 -v $PWD/tmp-zeppelin-notebooks:/opt/zeppelin/notebook \
                 -e ZEPPELIN_IN_DOCKER=true \
                 -e ALLUXIO_MASTER_EMBEDDED_JOURNAL_ADDRESSES=<master_hostname_1>:19200,<master_hostname_2>:19200,<master_hostname_3>:19200 \
                 -e SPARK_MASTER="spark://<spark_master_ip_addr>:7077" \
                 --name alluxio-zeppelin \
                 alluxio/zeppelin:0.10.0_2.9.0
                 
When running against a Zookeeper-based HA configured Alluxio cluster, use this command:

     docker run -p 8080:8080 --rm \
                 -v $PWD/tmp-zeppelin-logs:/opt/zeppelin/logs \
                 -v $PWD/tmp-zeppelin-notebooks:/opt/zeppelin/notebook \
                 -e ZEPPELIN_IN_DOCKER=true \
                 -e ALLUXIO_ZOOKEEPER_ENABLED=true \
                 -e ALLUXIO_ZOOKEEPER_ADDRESS=<zk1_hostname>:2181, <zk2_hostname>=zk1:2181, <zk3_hostname>:2181 \
                 -e SPARK_MASTER="spark://<spark_master_ip_addr>:7077" \
                 --name alluxio-zeppelin \
                 alluxio/zeppelin:0.10.0_2.9.0
                 
**Run in a Kubernetes Pod**

<TBD>

### Summary

This git repo provided instructions for building an Apache Zeppelin server that integrates with Alluxio 2.9.0. If you have any questions or comments, please send them to greg.palmer@alluxio.com

