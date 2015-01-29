---
layout: post
title: "Create a Hadoop Compilation and Development Environment"
date: 2015-01-29 11:56:41 -0800
comments: true
categories: Hadoop
---

If you want to read the source code of Hadoop and dig into the internals of Hadoop, then you need to know how to compile the source code and use IDEs (such as Eclipse or IntelliJ Idea) to open the projects. This blog will introduce you how to create a Hadoop build and development environment.

**Environment**: CentOS 6.6,  Oracle JDK 1.7.0_75

## 1. Install Oracle JDK 7

    Download jdk-7u75-linux-x64.rpm
    sudo yum localinstall -y ./jdk-7u75-linux-x64.rpm

For now if you use JDK 8 to compile the source code of Hadoop, it will fail because the javadoc  in Java 8 is considerably more strict than the one in earlier version, see more detailes [here](http://blog.joda.org/2014/02/turning-off-doclint-in-jdk-8-javadoc.html)

## 2. Install Maven

    wget http://mirrors.gigenet.com/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz
    sudo tar -zxf apache-maven-3.2.5-bin.tar.gz -C /opt
    sudo vim /etc/profile
    export M2_HOME=/opt/apache-maven-3.2.5
    export PATH=$M2_HOME/bin:$PATH

## 3. Install FindBugs(Optional)

    wget http://iweb.dl.sourceforge.net/project/findbugs/findbugs/3.0.0/findbugs-3.0.0.tar.gz
    sudo tar -zxf findbugs-3.0.0.tar.gz -C /opt
    sudo vim /etc/profile
    export FB_HOME=/opt/findbugs-3.0.0
    export PATH=$FB_HOME/bin:$PATH

If you want to run `mvn compile findbugs:findbugs` then you need to install FindBugs.

<!-- more -->

## 4. Install build tools

    sudo yum install -y gcc-c++ cmake autoconf automake

## 5. Install native libraries packages

    sudo yum -y install  gcc-c++ lzo-devel  zlib-devel libtool openssl-devel

Optional:

    sudo yum install -y snappy-devel
    sudo yum install -y svn

During the process of compilation, it will throw out some warning messages if there is no `svn` command.

## 5. Install Protobuf

    wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
    tar -zxf protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0
    ./configure
    make
    sudo make install
    rm -rf ./protobuf-2.5.0
    rm ./protobuf-2.5.0.tar.gz

**NOTE**: The Protobuf version must be exactly 2.5.0, or the compilation will fail.

## 6. Download the source code

    git clone git@github.com:apache/hadoop.git

## 7. Compile

Create binary distribution with native code and without documentation.

    mvn clean package -Pdist,native -Dtar -DskipTests -Drequire.snappy -Drequire.openssl

## 8. Install IntelliJ Idea

    wget http://download-cf.jetbrains.com/idea/ideaIC-14.0.3.tar.gz
    sudo tar -zxf ideaIC-14.0.3.tar.gz -C /opt
    Launch IntelliJ Idea, Create Desktop Entry

## 9. Open projects in IntelliJ Idea

First you need to run

    mvn install -DskipTests

Since some submodules of Hadoop depend on other submodules,  so you need to run `mvn install -DskipTests` to copy jars of Hadoop submodules to local `$HOME/.m2` , so that Maven won't download them from Internet. If you omit this step, Maven will try to download them from public Maven repository and will fail, because the newest version of Hadoop jars are not available in public Maven repository yet.


Then click `File->Open` to open the `pom.xml` in the root directory of Hadoop repo.

**Note**: Don't use `mvn idea:idea` to generate IntelliJ Idea projects first , IntelliJ Idea can handle pom.xml quite well, besides, the ["Maven IDEA Plugin"](http://maven.apache.org/plugins/maven-idea-plugin/) has already RETIRED.

## 10. Increase the memory for IntelliJ Idea(Optional)

There are many classes in Hadoop source code, if your IntelliJ hangs and doesn't respond from time to time, then you can try to give more memory to IntelliJ Idea by modifying the file `idea64.vmoptions` in the installation directory. Example below:

    -Xms128m
    -Xmx4096m
    -XX:MaxPermSize=1024m

## Reference

1. [HowToContribute](http://wiki.apache.org/hadoop/HowToContribute)
1. [BUILDING.txt](https://github.com/apache/hadoop/blob/trunk/BUILDING.txt)
1. [Create a Hadoop Build and Development Environment](http://vichargrave.com/create-a-hadoop-build-and-development-environment-for-hadoop/)
1. [How-to: Create an IntelliJ IDEA Project for Apache Hadoop](http://blog.cloudera.com/blog/2014/06/how-to-create-an-intellij-idea-project-for-apache-hadoop/)

In my next blog, I'll explain how to [Debug Hadoop Applications with IntelliJ](http://www.soulmachine.me/blog/2015/01/30/debug-hadoop-applications-with-intellij/)
