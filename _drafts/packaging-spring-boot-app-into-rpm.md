---
layout: post
title: Packaging a Spring Boot application into an RPM with Gradle
categories: tech
tags: spring-boot gradle
---

Some things I learnt from using the
[Gradle Linux Packaging Plugin](https://github.com/nebula-plugins/gradle-ospackage-plugin).

* TOC
{:toc}

## Background

Recently I was working with a client where we were developing new
[Spring Boot](https://projects.spring.io/spring-boot/)
[microservices](https://en.wikipedia.org/wiki/Microservices)
to be deployed to [Amazon Web Services](https://aws.amazon.com/)
VMs. The services were being built using [Gradle](https://gradle.org/)
and needed to be packaged as [RPMs](https://en.wikipedia.org/wiki/Rpm_(software))
for deployment by [AWS CloudFormation](https://aws.amazon.com/cloudformation/) scripts.

I started with a Gradle configuration copied from somebody else’s project and
cut it down as I figured out what worked and what didn’t, and what was actually
needed.

I am guilty of not reading all the [plugin
documentation](https://github.com/nebula-plugins/gradle-ospackage-plugin/wiki)
and, by trial and error, found out some things.

## Target environment

* Linux environment with RPM package system (RedHat, Centos).
* Supervisor (`supervisord`) installed.

## Contents of `package.gradle`

Here is `package.gradle` as I described in [Gradle build file organisation and reuse]({% post_url
2017-07-12-gradle-build-file-organisation %}) with explanatory comments.

```groovy
// Package a Spring Boot application in an RPM.

buildscript {
    ext {
        osPackageVersion = '4.4.0'
    }
    repositories {
        mavenCentral()
        maven { url 'https://plugins.gradle.org/m2/' }
    }
    dependencies {
        classpath "com.netflix.nebula:gradle-ospackage-plugin:${osPackageVersion}"
    }
}
apply plugin: com.netflix.gradle.plugins.application.OspackageApplicationPlugin

// This setting is supposed to cause creation of a single über-JAR, but I found it makes
// no difference.
springBoot {
    executable = true
}

// The JARs and scripts will be copied into /opt/${packageName}.
ospackage {
    // The docs say this is set already but in the ospackage closure it is not
    // set, so get it (again) from project name.
    packageName = project.name

    // Remove illegal chars from rpm version.
    version = "${project.version}".replaceAll('(~|-)', '_')

    os = LINUX
    type = BINARY
    arch = NOARCH

    preInstall file('scripts/rpm/preInstall.sh')
    postInstall file('scripts/rpm/postInstall.sh')
    preUninstall file('scripts/rpm/preUninstall.sh')
    postUninstall file('scripts/rpm/postUninstall.sh')

    // Copy the config files
    from('install/unix/conf') {
        fileType CONFIG | NOREPLACE
        fileMode 0754
        into "/opt/${packageName}/conf"
    }
}

buildRpm {
    user "${packageName}"
    permissionGroup "${packageName}"

    // Creates an empty log directory
    directory("/var/log/${packageName}", 0755)

    /* Symlink the supervisord configuration into the directory where
       it will be picked up */
    link("/etc/supervisor/${packageName}.ini",
            "/opt/${packageName}/conf/${packageName}.ini")
}
```

## Default settings

* The RPM that is constructed puts files into a directory tree starting at
  `/opt/${project.name}`
