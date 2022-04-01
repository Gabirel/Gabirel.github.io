---
layout: post
lang: en
title: The Template of Gradle(Kotlin and Java)
categories: gradle kotlin
tags: gradle java kotlin
date: 2018-04-18
---

<p class="intro"><span class="dropcap">T</span>his is the template showing how to combine Java and Kotlin into gradle file.</p>



```gradle
buildscript {
    ext.kotlin_version = '1.2.30'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

group 'com.godeep'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'kotlin'
apply plugin: 'application'

sourceCompatibility = 1.9

mainClassName = 'Main'

sourceSets {
    main.java.srcDirs += "src/main"
    test.java.srcDirs += "src/test"
}

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "com.jfoenix:jfoenix:9.0.3"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```



<center><blockquote>The End</blockquote></center>