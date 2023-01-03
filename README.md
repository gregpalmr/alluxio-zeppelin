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

If you want to run the Zeppelin server in a Docker container or in a Kubernetes pod, you can build the Docker image file using the following instructions.

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

TBD

### Step 5. Build a stand-alone Linux build
     
If you want to run the Zeppelin server on Linux, you can build the release using these instructions.
     
     ALLUXIO_VERSION="2.9.0"
     ZEPPELIN_VERSION="0.10.0"
     ZEPPELIN_HOME="/opt/zeppelin"
     
     # Clone the Zeppelin git repo
     git clone --single-branch --branch branch-0.10 https://github.com/apache/zeppelin.git
     cd zeppelin
     
     # Change the Alluxio POM file to use the specified version of Alluxio source
     sed -i "s/<alluxio.version>.*<\/alluxio.version>/<alluxio.version>${ALLUXIO_VERSION}<\/alluxio.version\>/" alluxio/pom.xml
     
     # Configure NPM and Bower
     echo "unsafe-perm=true" > ~/.npmrc 
     echo '{ "allow_root": true }' > ~/.bowerrc
     
     # Build the Zeppelin server from source
     mvn -B clean package -DskipTests -Pbuild-distr -Pspark-3.0 -Pscala-2.12 -Phadoop2
     
     # Move the distro artifacts to /opt/zeppelin
     mv zeppelin-distribution/target/zeppelin-*/zeppelin-* ${ZEPPELIN_HOME}/ 
     
     # Fetch the Alluxio assembly jar file required by the Zeppelin Alluxuio interpreter
     wget https://downloads.alluxio.io/downloads/files/${ALLUXIO_VERSION}/alluxio-${ALLUXIO_VERSION}-bin.tar.gz 
     tar xvfz alluxio-${ALLUXIO_VERSION}-bin.tar.gz 
     cp alluxio-${ALLUXIO_VERSION}/assembly/alluxio-client-*.jar ${ZEPPELIN_HOME}/interpreter/alluxio/ 
     
     # Clean up the build environment
     cd .. 
     rm -rf ~/.m2 
     rm -rf zeppelin 
     rm -rf alluxio-${ALLUXIO_VERSION}*
     
### Step 6. Run a stand-alone Linux build

To run the Zeppelin server on Linux, you must first configure the Zeppelin Alluxio interpreter. Modify the interpreter configuration file using the vi editor:
     
     vi /opt/zeppelin/interpreter/alluxio/interpreter-setting.json
     
If you are running with a single Alluxio master node, change the contents to point to your Alluxio master node, like this:
     
     [
       {
         "group": "alluxio",
         "name": "alluxio",
         "className": "org.apache.zeppelin.alluxio.AlluxioInterpreter",
         "properties": {
           "alluxio.master.hostname": {
             "envName": "ALLUXIO_MASTER_HOSTNAME",
             "propertyName": "alluxio.master.hostname",
             "defaultValue": "<my_alluxio_master_server_ hostname>",
             "description": "Alluxio non-HA - master hostname",
             "type": "string"
           },
           "alluxio.master.port": {
             "envName": "ALLUXIO_MASTER_PORT",
             "propertyName": "alluxio.master.port",
             "defaultValue": "19998",
             "description": "Alluxio non-HA - master port",
             "type": "number"
           }
         },
         "editor": {
           "editOnDblClick": false,
           "completionSupport": true
         }
       }
     ]

If you are running Alluxio in HA mode, using the built-in Raft service, you can configure the Alluxio interpreter like this:
     
     [
       {
         "group": "alluxio",
         "name": "alluxio",
         "className": "org.apache.zeppelin.alluxio.AlluxioInterpreter",
         "properties": {
           "alluxio.master.hostname": {
             "envName": "ALLUXIO_MASTER_EMBEDDED_JOURNAL_ADDRESSES",
             "propertyName": "alluxio.master.embedded.journal.address",
             "defaultValue": "<master_hostname_1>:19200,<master_hostname_2>:19200,<master_hostname_3>:19200",
             "description": "Alluxio HA - Alluxio hostnames and ports",
             "type": "string"
           },
         "editor": {
           "editOnDblClick": false,
           "completionSupport": true
         }
       }
     ]

If you are running Alluxio in HA mode, using the Zookeeper service, you can configure the Alluxio interpreter like this:
     
     [
       {
         "group": "alluxio",
         "name": "alluxio",
         "className": "org.apache.zeppelin.alluxio.AlluxioInterpreter",
         "properties": {
           "alluxio.master.hostname": {
             "envName": "ALLUXIO_ZOOKEEPER_ENABLED",
             "propertyName": "alluxio.zookeeper.enabled",
             "defaultValue": "true",
             "description": "Alluxio master hostname",
             "type": "string"
           },
           "alluxio.master.hostname": {
             "envName": "ALLUXIO_ZOOKEEPER_ADDRESS",
             "propertyName": "alluxio.zookeeper.address",
             "defaultValue": "<zk1_hostname>:2181, <zk2_hostname>=zk1:2181, <zk3_hostname>:2181",
             "description": "Alluxio HA - Zookeeper hostnames and ports",
             "type": "string"
           },
         "editor": {
           "editOnDblClick": false,
           "completionSupport": true
         }
       }
     ]
     
With the Alluxio interpreter configured, you can start the Zeppelin server using the supplied start up script:
     
     export USE_HADOOP=false
     /opt/zeppelin/bin/zeppelin-daemon.sh start
     
Later, you can stop the Zeppelin server using the command:
     
     /opt/zeppelin/bin/zeppelin-daemon.sh stop

### Step 7. Use the Zeppelin Alluxio interpreter

To use the Zeppelin Alluxi interpreter, start a new Zeppelin notebook session by pointing your Web browser to the Zeppelin server, like this:
     
     http://<zeppelin_server_address>8080

Then, create a new notebook and use the Alluxio interpreter by running the following paragraphs:

     %alluxio
     help
     Commands list:
	[help] - List all available commands.
	[cat <path>] - Prints the file's contents to the console.
	[chgrp [-R] <group> <path>] - Changes the group of a file or directory specified by args. Specify -R to change the group recursively.
	[chmod -R <mode> <path>] - Changes the permission of a file or directory specified by args. Specify -R to change the permission recursively.
	[chown -R <owner> <path>] - Changes the owner of a file or directory specified by args. Specify -R to change the owner recursively.
	[copyFromLocal <src> <remoteDst>] - Copies a file or a directory from local filesystem to Alluxio filesystem.
	[copyToLocal <src> <localDst>] - Copies a file or a directory from the Alluxio filesystem to the local filesystem.
	[count <path>] - Displays the number of files and directories matching the specified prefix.
	[createLineage <inputFile1,...> <outputFile1,...> [<cmd_arg1> <cmd_arg2> ...]] - Creates a lineage.
	[deleteLineage <lineageId> <cascade(true|false)>] - Deletes a lineage. If cascade is specified as true, dependent lineages will also be deleted.
	[du <path>] - Displays the size of the specified file or directory.
	[fileInfo <path>] - Displays all block info for the specified file.
	[free <file path|folder path>] - Removes the file or directory(recursively) from Alluxio memory space.
	[getCapacityBytes] - Gets the capacity of the Alluxio file system.
	[getUsedBytes] - Gets number of bytes used in the Alluxio file system.
	[listLineages] - Lists all lineages.
	[load <path>] - Loads a file or directory in Alluxio space, makes it resident in memory.
	[loadMetadata <path>] - Loads metadata for the given Alluxio path from the under file system.
	[location <path>] - Displays the list of hosts storing the specified file.
	[ls [-R] <path>] - Displays information for all files and directories directly under the specified path. Specify -R to display files and directories recursively.
     ...

     %alluxio
     ls /   
     /tmp
     /user
     ...
     
### Summary

This git repo provides instructions for building an Apache Zeppelin server that integrates with Alluxio 2.9.0. If you have any questions or comments, please send them to greg.palmer@alluxio.com

