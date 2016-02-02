    Title: Analyzing Dependencies With The Maven Site Plugin
    Date: 2008-01-16T00:00:01
    Tags: java, maven

Now as most people will tell you I’m kind of a geek when it comes to Maven. It’s really a nice tool and it makes dependency management in large projects almost a no brainer…almost. The story I’m about to share is true, only the names have been changed to protect the innocent…

<!-- more -->

A couple of weeks ago we were having some clashes in one of our applications due to an older version of log4j showing up on the classpath. So what could have been like finding a needle in a haystack was actually fairly simple to track down, if you only know where to look. Thankfully, some of my ranting and raving about Maven has rubbed off at this client. They took some of my suggestions as far as Maven-izing their new projects and have even seen the benefits of using the site plugin to generate project documentation. In this case, the site plugin is what ended up saving us. In the site documentation that is generated there are a couple of reports called, oddly enough, `Dependencies` and `Dependency Convergence`.

## Dependency Convergence
The Dependency Convergence report is especially nice when dealing with large multi-module projects. At this client it is normal for each project to contain anywhere from 3 – 7 modules and maintaining a pom.xml for each one can sometimes lead to versions getting out of sync between modules. This report analyzes all of the dependency versions that appear in all of the modules for the project, and compares them to the versions of the dependencies defined for this particular module.

At the top of the report you’ll see the summary. In this specific report you’ll see that this module has quite a few mismatched versions being defined as dependencies.

![Dependency Convergence Summary](/img/dependency-stats1.png)

The rest of the report consists of an itemized list of every dependency that is defined in the module and this is where you can see what the discrepancies are, if any.

![Dependency Convergence](/img/dependency-convergence.png)

## Dependencies
The Dependencies report is the one that actually saved us some serious investigation. We had just gone through and removed any explicit dependencies on log4j, since we were using commons-logging and Weblogic provides log4j for us. Somehow log4j kept showing up on the classpath, and it was an older version which had a method with a different signature than what was provided by Weblogic, so it was still causing problems. So along comes the dependency report to the rescue. The top of the report lists out what dependencies are defined and in what scope, followed by any transitive dependencies. A little further down the report is a section that actually gives a tree-view of your dependencies so you can see the hierarchy. Here is where we discovered that displaytag was the culprit.

![Dependency Tree](/img/dependency-hierarchy.png)
