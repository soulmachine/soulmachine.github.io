---
layout: post
title: "Java Code Style and Static Analysis"
date: 2014-12-15 11:42:09 -0800
comments: true
categories: 
---

##1. Checkstyle

[CheckStyle](http://checkstyle.sourceforge.net/) is a development tool to help programmers write Java code that adheres to a coding standard. It automates the process of checking Java code to spare humans of this boring (but important) task. This makes it ideal for projects that want to enforce a coding standard.

Which Java code style to choose? Google has published a few coding standards, include [Google Java Style](http://google-styleguide.googlecode.com/svn/trunk/javaguide.html). Aditionally, there are xml configuration files for Eclipse and IntelliJ in the SVN repository, <https://code.google.com/p/google-styleguide/source/browse/trunk>. 

Checkstyle Eclipse plugin has already used Google Java style by default, checkt it by clicking "Window->Preferences->Checkstyle->Global Check Configurations", or at <https://github.com/checkstyle/checkstyle/blob/master/google_checks.xml>.


###1.1 Checkstyle Eclipse Plugin

To check your coding style automatically in Eclipse, just install the Checkstyle Eclpise plugin. Launch Eclipse, click Menu "Help -> Install New Software", input 
<http://eclipse-cs.sf.net/update/>, then click "Next" to install the plugin.

###1.2 Checkstyle Maven Plugin

Add the following lines to pom.xml to enable Checkstyle:
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.13</version>
                <dependencies>
                    <dependency>
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>6.1.1</version>
                    </dependency>
                </dependencies>
                <executions>
                    <execution>
                        <id>checkstyle</id>
                        <phase>validate</phase>
                        <configuration>
                            <configLocation>google_checks.xml</configLocation>
                            <encoding>UTF-8</encoding>
                            <consoleOutput>true</consoleOutput>
                            <failsOnError>true</failsOnError>
                            <includeTestSourceDirectory>true</includeTestSourceDirectory>
                        </configuration>
                        <goals>
                            <goal>checkstyle</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.13</version>
                <configuration>
                    <configLocation>https://raw.githubusercontent.com/checkstyle/checkstyle/master/google_checks.xml</configLocation>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jxr-plugin</artifactId>
                <version>2.5</version>
            </plugin>
        </plugins>
    </reporting>

<!-- more -->

##1.3 Maven Eclipse Plugin

To format source code automatically according to Google Java style, import the file [eclipse-java-google-style.xml]( https://google-styleguide.googlecode.com/svn/trunk/eclipse-java-google-style.xml) into Eclipse by clicking menu "Window->Preferences->Java->Code Style->Formatter->Import".

You can add the code style automatically via Maven, add the following lines to pom.xml:

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-eclipse-plugin</artifactId>
        <version>2.9</version>
        <configuration>
            <downloadSources>true</downloadSources>
            <downloadJavadocs>true</downloadJavadocs>
            <workspace>${basedir}</workspace>
            <workspaceCodeStylesURL>https://google-styleguide.googlecode.com/svn/trunk/eclipse-java-google-style.xml</workspaceCodeStylesURL>
        </configuration>
    </plugin>


##2 PMD

Checking code style is not enough, there are many static anaylysis tools which can analyze your code and find potential bugs, for example PMD, findbugs, etc. The difference between them is <http://www.sw-engineering-candies.com/blog-1/comparison-of-findbugs-pmd-and-checkstyle>.

[PMD](http://pmd.sourceforge.net/) is a source code analyzer. It finds common programming flaws like unused variables, empty catch blocks, unnecessary object creation, and so forth.

###2.1 PMD Eclipse Plugin

PMD Eclipse plugin update site: 
<http://sourceforge.net/projects/pmd/files/pmd-eclipse/update-site/>.

###2.2 Maven PMD Plugin

To enable automatically static analysis during compilation, add the following lines to pom.xml:

    <build>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-pmd-plugin</artifactId>
            <version>3.3</version>
            <executions>
                <execution>
                    <phase>validate</phase>
                    <goals>
                        <goal>check</goal>
                        <goal>cpd-check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </build>
    <reporting>
        <plugins>
            <plugin>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>3.3</version>
                <reportSets>
                    <reportSet>
                        <reports>
                            <report>pmd</report>
                            <report>cpd</report>
                        </reports>
                    </reportSet>
                </reportSets>
            </plugin>
        </plugins>
    </reporting>


##3. FindBugs

###3.1 FindBugs Eclipse Plugin

Eclipse Update Site: <http://findbugs.cs.umd.edu/eclipse>

###3.2 Maven FindBugs Plugin

Official site: <http://mojo.codehaus.org/findbugs-maven-plugin/>

To configure your build to fail if any errors are found in the FindBugs report, add the following lines to pom.xml.
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>findbugs-maven-plugin</artifactId>
                <version>3.0.1-SNAPSHOT</version>
                <configuration>
                    <effort>Max</effort>
                    <threshold>Low</threshold>
                    <xmlOutput>true</xmlOutput>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

Reference: <http://mojo.codehaus.org/findbugs-maven-plugin/examples/violationChecking.html>

