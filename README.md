[![Build Status](https://travis-ci.org/Dumb-Code/GradleHook.svg?branch=master)](https://travis-ci.org/Dumb-Code/GradleHook)
[![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=net.dumbcode.gradlehook&metric=alert_status)](https://sonarcloud.io/dashboard?id=net.dumbcode.gradlehook)

# Gradle-Hook
Adds the `postRequest` task which simply posts a POST request along with the specified builds. Additional fields for the request can be specified. The request uses the user agent `Mozilla/5.0` and has the content-type of `multipart/form-data`

# Applying the Plugin
Using the [plugins DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block):
```gradle
plugins {
  id "net.dumbcode.gradlehook" version "1.2.0"
}
```
Using [legacy plugin application](https://docs.gradle.org/current/userguide/plugins.html#sec:old_plugin_application):
```gradle
buildscript {
  repositories {
    maven {
      url "https://plugins.gradle.org/m2/"
    }
  }
  dependencies {
    classpath "net.dumbcode.gradlehook:GradleHook:1.2.0"
  }
}

apply plugin: "net.dumbcode.gradlehook"

```

# Simple Plugin
The bare minimum of a plugin
```gradle
gradlehook {
    urlToken "http://example.com/webhook" //You will want to keep this url private. It should really be in a gradle.properties file.
    addArtifact jar
}
```
## Additional Tasks
You can apply multiple tasks to be sent over. In this senario 2 files would be sent accross, the 
```gradle
task sourcesJar(type: Jar, dependsOn: classes) {
    classifier = "sources"
    from sourceSets.main.allSource
}

artifacts {
    archives sourcesJar
}

gradlehook {
    urlToken "http://example.com/webhook"
    addArtifact jar
    addArtifact sourcesJar
}
```

## Fields
When sending the request, you might want to add additional data. This can be done with the `addField` method.
For example, sending a webhook to a discord server would be:
```gradle
gradlehook {
    urlToken 'https://discordapp.com/api/webhooks/012345678912345678/foobar' //Would go in a gradle.properties, or a file that isn't commited to git
    
    addField 'payload_json', '{ "embeds": [{ "timestamp": "{{datetime}}" }] }'
    
    addArtifact jar
}
```

## Field Placeholders
The fields are able to have placeholders, as shown in the above example. These placeholders mean the following:
 - `{{version}}` -> project version
 - `{{name}}` -> project name
 - `{{group}}` -> project group
 - `{{datetime}}` -> the current time in UTC, in ISO-8601 format 
 
 ## Message First Option
 In some senarios, you want the text message to sent as a seperate webhook before the build webhooks. The following would mean a webhook with the field "id" would be sent, then once an HTTP_OK response code is sent, the artifacts are sent over in a webhook. 
```gradle
gradlehook {
    urlToken "http://example.com/webhook"
    addField 'id', 'user2201'
    addArtifact jar
    messageFirst
}
```
