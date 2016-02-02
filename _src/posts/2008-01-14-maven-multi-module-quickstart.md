    Title: Maven Multi-Module Quickstart
    Date: 2008-01-14T00:00:00
    Tags: java, maven

Recently I've had lots of questions about how to create multi-module projects, so when I discovered this technique, I thought I'd write this up. This technique exploits a little known feature of the `archetype:create` plugin, and the Maven site archetype to kickstart your project. Creating a multi-module project has many benefits, one of them being the ability to build every artifact in a project with one simple `mvn compile` command. Another benefit is that if you are using either the maven `eclipse:eclipse` plugin or the `idea:idea` plugin, you can enter this command at the root of the project and it will generate all of the project files for all of the contained modules.

<!-- more -->

First generate the top level project using the `maven-archetype-site-simple` archetype using the following command,

```
mvn archetype:create
 -DgroupId=[Java:the project's group id]
 -DartifactId=[Java:the project's artifact id]
 -DarchetypeArtifactId=maven-archetype-site-simple

```

this will generate a Maven project with the following directory structure.

![Maven Site Simple Folder](/img/maven-archetype-site-simple.png "folder structure")

The project that is generated is the minimum project setup to generate site documentation. The `index.apt` file is the main index page for the site, and is written in the Almost Plain Text format, which is a wiki like format. You can also generate a more complete site project using the `maven-archetype-site` archetype like this,

```
mvn archetype:create
 -DgroupId=[Java:the project's group id]
 -DartifactId=[Java:the project's artifact id]
 -DarchetypeArtifactId=maven-archetype-site
```

This will generate the following project structure.

![Maven Site Folder Structure](/img/maven-archetype-site.png "site folder structure")

After you have generated the site project, edit the `pom.xml` created from the site archetype plugin. Make sure the the packaging type is set to `pom` like the following.

``` xml
<project>
    <modelversion>4.0.0</modelversion>
    <groupid>com.pillartechnology</groupid>
    <artifactid>sampleProject</artifactid>
    <version>1.0-SNAPSHOT</version>
<packaging>pom</packaging>
...
</project>
```

By setting the packaging type to `pom`, any projects you generate from the root of the project directory will insert itself into the project by creating an entry into the modules section of the `pom.xml` for the site. In the root directory of your project that you created above, type in the following command,

```
mvn archetype:create
 -DgroupId=[Java:the module's group id]
 -DartifactId=[Java:the module's artifact id]
```

If you now edit the `pom.xml` for the main project, you should see an entry towards the bottom of the file like the following.

``` xml
...
<modules>
    <module>sampleModule</module>
</modules>
...
```
