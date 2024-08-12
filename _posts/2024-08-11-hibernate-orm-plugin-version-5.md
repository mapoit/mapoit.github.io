---
layout: post
title: Applying Hibernate ORM Plugin with Hibernate 5
subtitle: Gradle plugins remove old version so fast
comments: true
mathjax: false
author: Mapo IT
---

[Hibernate ORM plugin](https://vladmihalcea.com/maven-gradle-hibernate-enhance-plugin/) is a well-known solution for extending Hibernate's behavior through bytecode enhancement. Many articles cover how to apply it to your Gradle or Maven project.

I attempted to integrate this plugin into my project, which uses Spring Boot 2.7 and Hibernate 5.6.15.Final. However, I encountered an issue while setting up the plugin.

My project uses Gradle 7.5.1, and I tried to apply the Hibernate ORM plugin to my `build.gradle.kts` file as follows:


```
plugins {
	id("org.hibernate.orm") version "5.6.15.Final"
}
...
hibernate {
	enhance(
        closureOf<EnhanceExtension> {
            enableLazyInitialization = true
            enableDirtyTracking = true
            enableAssociationManagement = true
            enableExtendedEnhancement = false
        }
    )
}
```

However, with this configuration, the build fails with the following error:
```
Plugin [id: 'org.hibernate.orm', version: '5.6.15.Final'] was not found in any of the following sources:
```

This error occurs because the [Gradle plugin repository](https://plugins.gradle.org/plugin/org.hibernate.orm) only hosts the `org.hibernate.orm` plugin versions 6.0 and higher, so it cannot find version 5.6.15.Final.

To resolve this, we need to configure Gradle to find the plugin from the Maven repository instead of the Gradle plugin repository. This can be done by setting up a resolution strategy in the `pluginManagement` section of `settings.gradle.kts`:

```
pluginManagement {
    repositories {
        mavenLocal()
        mavenCentral()
        gradlePluginPortal()    
    }
    resolutionStrategy {
        eachPlugin {
            if (this.requested.id.id == "org.hibernate.orm") {
                useModule("org.hibernate:hibernate-gradle-plugin:${this.requested.version}")
            }
        }
    }  
}
```


By configuring this resolution strategy, Gradle will resolve the `org.hibernate.orm` plugin using the `org.hibernate:hibernate-gradle-plugin` module from the [Maven repository](https://mvnrepository.com/artifact/org.hibernate/hibernate-gradle-plugin).

While it's possible to use this plugin with the older `buildscript` and `apply plugin` method, in my project, other plugins were already configured using the `plugins` directive. Therefore, this approach helps to resolve the plugin version conflict and maintain consistency in the build script.

