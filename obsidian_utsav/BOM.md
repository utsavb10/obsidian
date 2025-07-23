
To showcase importance of bom when you have a library. The library is a collection of multiple modules which maybe intertwined with each other.

For this demo, I have created a resource-library as our library.
resourceful-library will consist of three modules, uno, dos and tres.
We also have a business-service which would independent modules of this library as per need.


Commit Showcase order

Open m2 folder as well

```
lib, create awesome library with uno, dos, tres modules
service, create business service with uno and dos dependency
```

Till now, we only see first commits in uno and duo libs and our business-service using it plain and simple by declaring their versions. 
Run main to show it is working

```
lib, Library-Uno: add abbreviation functionality and version change ....
service, Update duo dependency to v1.1 and resolve the code accordingly in Main
```

Run main to show it is working

imagine this work was done by dev-1 up till now
now dev-2 is asked to use the lib-tres changes into our business service

```
service, Use tres library to call abbreviate function for my minions' address
```

the code looks good, on service
run> mvn clean verify 
should work
dev-2 puts this commit into master, dev-2 is super-confident on this *untested code* straight goes a deployment
....... and disaster happens in prod

Run the main file

```
Naaruhodo
Exception in thread "main" java.lang.NoSuchMethodError: 'java.lang.String com.nocom.uno.LangUtil.abbreviate(java.lang.String)'
	at com.nocom.tres.TresUtil.addressModifier(TresUtil.java:18)
	at com.nocom.business.Main.main(Main.java:30)

Process finished with exit code 1
```

What happened ?

The TresUtil required method named addressModifier but it was not available at runtime.

Why ? 

run> mvn dependency:tree -Dverbose=true

Should get something like this below,
```
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ business-service ---
[WARNING] The artifact xml-apis:xml-apis:jar:2.0.2 has been relocated to xml-apis:xml-apis:jar:1.0.b2
[INFO] com.nocom:business-service:jar:1.0-SNAPSHOT
[INFO] +- com.nocom:library-dos:jar:1.1-SNAPSHOT:compile
[INFO] |  \- com.google.guava:guava:jar:32.0.0-jre:compile
[INFO] |     +- com.google.guava:failureaccess:jar:1.0.1:compile
[INFO] |     +- com.google.guava:listenablefuture:jar:9999.0-empty-to-avoid-conflict-with-guava:compile
[INFO] |     +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] |     +- org.checkerframework:checker-qual:jar:3.33.0:compile
[INFO] |     +- com.google.errorprone:error_prone_annotations:jar:2.18.0:compile
[INFO] |     \- com.google.j2objc:j2objc-annotations:jar:2.8:compile
[INFO] +- com.nocom:library-uno:jar:1.0-SNAPSHOT:compile
[INFO] |  \- org.apache.commons:commons-lang3:jar:3.17.0:compile
[INFO] \- com.nocom:library-tres:jar:1.0-SNAPSHOT:compile
[INFO]    \- (com.nocom:library-uno:jar:1.1-SNAPSHOT:compile - omitted for conflict with 1.0-SNAPSHOT)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------

```

Observe the omitted for conflict with 1.0-SNAPSHOT at last

Our given version was available at compile time but not at runtime since we declared to use version 1.0 of library-uno in our business-service pom.

Explain how maven picks up dependencies if not earlier.

now, hotfix,

```
service, dev-2 deploys hotfix
```

Is it enough ?
Can we somehow not care about the service's internal dependency matrix while using the library in our business-service.

Yes, time to use a bill of materials pom.

First step is to govern the module version information through the parent pom within the library.
Since we're only correcting pom versioning, I'm only going to change the patch version of the library modules but a minor version change for the parent pom.

```
lib, Govern module versions from the parent pom
```

This change has simplified the internal dependency reference within the modules but not much has changed wrt business-service. We still have to upgrade the version of each dependency.

```
service, upgrade each library version in service
```

Run main, should work.

Now we move towards the 2nd phase towards building the BOM

*Use parent as the pom*

Make sure this packaging is of type pom
and dependencyManagement section declares all the modules

```
lib, Parent declared as BOM
```

It seems like a major change for parent, so I bumped the major version of it and also bumped up the patch version of the modules just to make sure that I do not create multiple copies of the same module and version combination because of SNAPSHOT and also to show a clear distinction in the service for this demo.

Before we got to service changes, notice library-tres pom , see how we removed the version from dependency of library-uno and it is directly referred from the parent dependencyManagement lookup.

We need something like this on service side as well, moving on,

```
service, use resourceful library bom
```

Run the service, and it should work ....

This is a delight. We are in a state where we can publish a BOM and just modify the BOM version on the service. We just have to make sure as developers that we publish BOM with managed dependencies that are compatible with each other.

Is it everything we need here though ? 

1. If you imagine your life as a developer of this library, you would imagine yourself making change to the PARENT version each time you contribute to at least one module in this library. Apart from inconvenience of changing the parent version to each module, this methodology can also cause breakages if you forget bumping the parent version in even a single module.
2. Another concern comes related to the transitive dependencies, if your project(business-service) now needs to import guava as a dependency, it can also get referred from the same BOM whether we want it or not and may cause inconvenience for the developer who wants to use a specific version of the dependency in question.

Let us see how we can improve on this, 

```
lib, Create a library-parent at the same level to other modules
service, use bom version 3.0
```

If we open the maven tab on the right side of intellij. We can see the project structure is now a bit more hierarchial.

resourceful-library-bom
		library-parent
			library-uno
			library-dos
			library-tres

A slightly complex structure to maintain and understand.

**Important:** It so happens that [project aggregation and project inheritance](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Inheritance_vs_Project_Aggregation) are _two separate concepts_! Here the aggregated POMs declare the aggregate POM as their parent, but this need not be the case: a POM (resourceful-library-bom) may aggregate other POMs (uno,dos,tres) that do not in turn declare the aggregator POM as their parent. We'll revisit this distinction shortly and explain its implications.

```
lib, Resolve complex structure
service, use bom version 4.0
```

The structure now comes up like this,

resourceful-library-bom
		library-parent
library-uno
library-dos
library-tres

The good thing about this change is that we can leave the library-parent version as it is and take advantage of this version being a SNAPSHOT. We do not need to change the version property for each module when we make a change. 

The only con is that we would still like to change to BOM version accordingly and this will require a change into the library-parent pom's parent tag as well. 

Although, you can evade this using flatten-maven-plugin.
This tool is a bloody hack !

```
lib, Try building bom with flatten-maven-plugin to ease the parent properties
service, use bom version 5.0
```
