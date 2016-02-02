    Title: Continuous Integration With Flex
    Date: 2008-01-16T00:00:00
    Tags: agile, ci, flex, flexunit, java, tdd, ria
Earlier today I had posed a question to a mailing list in the .NET community asking about Continuous Integration with Flex in the .NET world. After a couple of answers from people who obviously did not understand the question, because they just told me to google CruiseControl.NET, someone with some knowledge of TDD and Agile practices stepped up and pointed out the obvious point I was trying to make. **There currently is no real good way to automate your FlexUnit tests in such a way that a CI server like CC.NET or HudsonCI would know whether or not all of the tests for your Actionscript classes passed or failed.**

<!-- more -->

~~So I’ve decided to start a Google Code project called agile-flex, where a couple of other developers and I will attempt to build some agile tools for the Flex framework, starting with a test runner that will help enable continuous integration for Java, .NET, or even just plain old Actionscript. The runner will likely be based off an article I found from Aaron Spjut here. In a nutshell we will create a test runner in Adobe AIR that will generate XML output similar to JUnit and NUnit for the CI server to be able to interpret. This will also enable the generation of report artifacts using the JUnit Report tasks or even a custom XSLT if desired. I’ll post more details as the project continues.~~

UPDATE… The Flex-Mojos project now fulfills this need, so I’ve deleted the Google Code Project that we started for this.
