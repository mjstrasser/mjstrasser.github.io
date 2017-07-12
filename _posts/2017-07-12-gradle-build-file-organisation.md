---
layout: post
title: Gradle build file organisation and reuse
categories: tech
---

Some ideas on how to organise and reuse Gradle build files in a multi-project build.

## Introduction

Our project was to create a set of related microservices in Java using Spring Boot,
built as a set of Gradle projects all under a single root project.

We wanted to reuse Gradle build scripts as much as possible and reduce the amount of
copy-and-paste between service projects.

## General layout

```text
(root)
  |
  +- build.gradle
  +- settings.gradle
  +- gradle/
  |    +- code-standards.gradle
  |    +- idea.gradle
  |    +- package.gradle
  |    +- spring-actuator-info.gradle
  |    +- spring-boot.gradle
  |    +- spring-rest-docs.gradle
  |
  +-- microservice-a/
  |     +- build.gradle
  |     +- gradle.properties
  |     +- src/
  |
  +-- microservice-b/
        +- build.gradle
        +- gradle.properties
        +- src/
```

## Gradle scripts

### Project root

The project root contains `settings.gradle` with:

```groovy
rootProject.name = 'api-microservices'

include 'microservice-a', 'microservice-b'
```

There can only be one `settings.gradle` file and it must be in the root directory.  

The root `build.gradle` contains information in common with all projects:

```groovy
// Common build versions for Spring Boot, Cloud and Prometheus, and Boot plugin.
buildscript {
    ext {
        springBootVersion = '1.5.4.RELEASE'
        springCloudVersion = 'Dalston.SR1'
        springMetricsVersion = '0.5.1.RELEASE'
        prometheusVersion = '0.0.23'
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

// External plugins cannot be declared in an external build script but will be available to
// all child projects.
plugins {
    id 'org.asciidoctor.convert' version '1.5.3'
    // Plugin to generate git.properties on the classpath that will be included
    // in the Actuator info response.
    id 'com.gorylenko.gradle-git-properties' version '1.4.17'
}

group 'com.example'

// These things are applied to all projects.
allprojects {

    apply plugin: 'java'
    sourceCompatibility = 1.8

    repositories {
        maven {
            // Search local repo first.
            url 'http://repo.example.com/public-remote-virtual'
            mavenCentral()
        }
    }
}
```

If you don’t want all your services to use the same version of Spring Boot (etc.) you will need a different
configuration.

### Service projects

The `build.gradle` for each service project is now simplified to start with:

```groovy
apply from: '../gradle/spring-boot.gradle'
apply from: '../gradle/idea.gradle'
apply from: '../gradle/code-standards.gradle'
apply from: '../gradle/spring-rest-docs.gradle'
apply from: '../gradle/spring-actuator-info.gradle'
apply from: '../gradle/package.gradle'
```

Other, project-specific declarations follow. For example, to use Redis:

```groovy
dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-redis')
}
```

Each project contains a `gradle.properties` file containing the current version of the service.
It is separate from `build.gradle` so it can be read and written by the
 [Gradle Release Plugin](https://github.com/researchgate/gradle-release).

## Common, reusable scripts

Here is a brief description of the reusable scripts that we created.

| File    | Description         |
|---------|---------------------|
| `spring-boot.gradle` | Apply Spring Boot plugin and specify Spring and other dependencies for all projects. |
| `idea.gradle` | Apply and configure the IntelliJ IDEA plugin. |
| `code-standards.gradle` | Apply and configure Checkstyle and Jacoco plugins. |
| `spring-rest-docs.gradle` | Apply and configure the AsciiDoctor plugin; also include the generated doc in the assembled JAR. |
| `spring-actuator-info.gradle` | Apply and configure the Gradle Git Properties plugin; Specify that Spring Boot generates `build-info.properties` that will returned by the Actuator `info` endpoint. |
| `package.gradle` | Apply and configure the Gradle Linux Packaging plugin to package the service as an RPM. |

## Postscript

We also used the [Gradle Release Plugin](https://github.com/researchgate/gradle-release)
but I was not able (in the time available) to get it to work from
an external script using `apply from`. This meant that each service build script contained the same big
block of code (not shown here). 

Perhaps I will solve that one in the future.
