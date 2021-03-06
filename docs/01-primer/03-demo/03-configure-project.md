---
layout: default
title: Configure project
parent: Demo
grand_parent: Primer
nav_order: 3
permalink: docs/primer/demo/configure/
---

# Configure project
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Configure project

1. Extract the downloaded zip file

   ```bash
   $ unzip contact-us.zip

   Archive:  contact-us.zip
      creating: contact-us/
     inflating: contact-us/settings.gradle
      creating: contact-us/gradle/
      creating: contact-us/gradle/wrapper/
     inflating: contact-us/gradle/wrapper/gradle-wrapper.properties
     inflating: contact-us/gradle/wrapper/gradle-wrapper.jar
     inflating: contact-us/gradlew
     inflating: contact-us/gradlew.bat
     inflating: contact-us/build.gradle
      creating: contact-us/src/
      creating: contact-us/src/main/
      creating: contact-us/src/main/java/
      creating: contact-us/src/main/java/demo/
      creating: contact-us/src/main/java/demo/boot/
     inflating: contact-us/src/main/java/demo/boot/ContactUsApplication.java
      creating: contact-us/src/main/resources/
     inflating: contact-us/src/main/resources/application.properties
      creating: contact-us/src/main/resources/templates/
      creating: contact-us/src/main/resources/static/
      creating: contact-us/src/test/
      creating: contact-us/src/test/java/
      creating: contact-us/src/test/java/demo/
      creating: contact-us/src/test/java/demo/boot/
     inflating: contact-us/src/test/java/demo/boot/ContactUsApplicationTests.java
     inflating: contact-us/HELP.md
     inflating: contact-us/.gitignore
   ```

   The directory structure of the project

   ```bash
   $ tree contact-us
   contact-us
   ├── HELP.md
   ├── build.gradle
   ├── gradle
   │   └── wrapper
   │       ├── gradle-wrapper.jar
   │       └── gradle-wrapper.properties
   ├── gradlew
   ├── gradlew.bat
   ├── settings.gradle
   └── src
      ├── main
      │   ├── java
      │   │   └── demo
      │   │       └── boot
      │   │           └── ContactUsApplication.java
      │   └── resources
      │       ├── application.properties
      │       ├── static
      │       └── templates
      └── test
          └── java
              └── demo
                  └── boot
                      └── ContactUsApplicationTests.java

   14 directories, 10 files
   ```

1. Navigate in the project's directory

   ```bash
   $ cd contact-us
   ```

   All commands are executed from within the project directory.

1. Delete the unnecessary files and folders

   ```bash
   $ rm -rf src/main/resources/templates/
   $ rm -rf src/main/resources/static/
   $ rm HELP.md
   ```

1. (_Optional_) Change the empty file's `src/main/resources/application.properties` extension to `yaml` (or `yml`), `src/main/resources/application.yaml`

   ```bash
   $ mv src/main/resources/application.properties src/main/resources/application.yaml
   ```

   {% include custom/note.html details="This is a matter of preference as the application can be configured either using <code>.properties</code> or <code>.yaml</code> files alike" %}

   Note that the examples shown in this demo make use of `yaml`.

1. Open the project in the IDE

   ```bash
   $ idea .
   ```

1. Configure Gradle

   Update file: `build.gradle`

   ```groovy
   plugins {
     id 'java'

     id 'org.springframework.boot' version '2.3.0.RELEASE'
     id 'io.spring.dependency-management' version '1.0.9.RELEASE'
   }

   java {
     sourceCompatibility = JavaVersion.VERSION_14
     targetCompatibility = JavaVersion.VERSION_14
   }

   repositories {
     mavenCentral()
     jcenter()
   }

   configurations {
     compileOnly {
      extendsFrom annotationProcessor
     }
   }

   dependencies {
     /* Lombok */
     compileOnly 'org.projectlombok:lombok'
     annotationProcessor 'org.projectlombok:lombok'

     /* Spring */
     implementation 'org.springframework.boot:spring-boot-starter-web'
     testImplementation('org.springframework.boot:spring-boot-starter-test') {
      exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
     }
   }

   test {
     useJUnitPlatform()
     testLogging {
      events = ['FAILED', 'PASSED', 'SKIPPED', 'STANDARD_OUT']
     }
   }
   ```

1. (_Optional_) include the [task-tree plugin](https://plugins.gradle.org/plugin/com.dorongold.task-tree)

   Update file: `build.gradle`

   ```groovy
   plugins {
     id 'com.dorongold.task-tree' version '1.5'
   }
   ```

   This will allow us to see which task depends on what.

1. Build the application

   ```bash
   $ ./gradlew clean build

   ...
   BUILD SUCCESSFUL in 4s
   6 actionable tasks: 5 executed, 1 up-to-date
   ```

   Should build without errors
