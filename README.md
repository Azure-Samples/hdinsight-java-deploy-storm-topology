---
services: hdinsight
platforms: dotnet
author: blackmist
---

# Programmatically deploy a topology to Apache Storm on HDInsight

An example Java application that will programmatically submit an Apache Storm topology to a Storm on HDInsight cluster. This is based on the example from [http://nishutayaltech.blogspot.in/2014/06/submitting-topology-to-remote-storm.html](http://nishutayaltech.blogspot.in/2014/06/submitting-topology-to-remote-storm.html).

## Prerequisites

* [Java](https://java.com/download): The programming language that this example is written in.

* [Maven](https://maven.apache.org/): A build management system for Java projects. Used to build the example code.

* [cURL](http://curl.haxx.se/): A cross-platform utility for working with REST APIs. Used to talk to the Ambari REST API on HDInsight.

* [jq](https://stedolan.github.io/jq/): A cross-platform utility for working with JSON documents. In this case, the data returned from Ambari through cURL.

* A Java-based Storm topology that you want to deploy. If you don't have one, consider using a basic [wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) topology.

## Build and package

1. If you are using an HDInsight 3.2 or older cluster version, edit the __pom.xml__ file and change the __version__ for the __storm-core__ dependency to 0.9.3.

        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <!-- change to the version of Storm that you are using -->
            <version>0.10.0</version>
        </dependency>

     The project defaults to 0.10.0, which is the version of Storm provided with HDInsight 3.3 and 3.4. Older cluster versions used Storm 0.9.3.

2. From the command line, change directories to the project directory and use the following to build and package the project:

        mvn package
    
    This will create a file named __SubmitToNimbus-0.0.1-SNAPSHOT.jar__ in the __target__ directory.

## Create an SSH tunnel and deploy a topology

Storm topologies are deployed by submitting the topology package to the Nimbus service, which runs on the HDInsight head nodes. This service listens on port 6627; however, this port is not exposed publicly on the internet. In order to communicate with Nimbus from outside the cluster, you can forward port 6627 on your development environment to port 6627 on the cluster using an SSH tunnel.

Use the following steps to forward port 6627 in your development environment to the cluster:

1. Use the following to find the internal fully qualified domain name (FQDN) of your head nodes:

        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'
    
    You should receive a response similar to the following:

        "hn0-mystor.n4tsyopoujmedocjui1mn2ysea.ex.internal.cloudapp.net"
        "hn1-mystor.n4tsyopoujmedocjui1mn2ysea.ex.internal.cloudapp.net"
    
    Save these values.

2. If you are using HDInsight 3.3 or 3.4, use the following command to forward your local port 6627 to __hn1__:

        ssh -p 23 -C2qTnNf -L 6627:HN1-FQDN:6627 USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    
    * Replace __HN1-FQDN__ with the value from the previous step that begins with __hn1__.

    * Replace __USERNAME__ with the SSH user name for your HDInsight cluster.

    * Replace __CLUSTERNAME__ with the name of your HDInsight cluster.

    If you secured your SSH account using a password, you will be prompted to enter it. If you used a certificate, you may need to use the `-i` parameter to specify the location of the private key.

    > [AZURE.NOTE] If you are using HDInsight 3.2, use the following command instead:
    >
    > `ssh -p 22 -C2qTnNf -L 6627:HN0-FQDN:6627 USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`
    >
    > Replace __HN0-FQDN__ with the value from the previous step that begins with __hn1__.

3. Use the following to submit the topology using the example application

        java -jar SubmitToNimbus-0.0.1-SNAPSHOT.jar <storm-topology-jar-file> <friendly-name-for-topology> <nimbus-host>

## Verify the topology was submitted

Assuming you received no errors during submission, you can view the topology using the Storm web UI for your HDInsight cluster. You can view this by pointing your browser to https://CLUSTERNAME.azurehdinsight.net/stormui. Replace __CLUSTERNAME__ with the name of your HDInsight cluster. You should see the friendly name listed as a running topology.

## Project code of conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.