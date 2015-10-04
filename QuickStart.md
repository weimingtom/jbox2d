

# Introduction #

JBox2D is a Java rigid body physics engine library.  It does not include any graphics support, you are expected to hook it up to your rendering library yourself (though the testbed includes a sample implementation using Java2D graphics, it is not intended to be a production-ready implementation).

This page will show you how to generally use/build jbox2d in any environment.

# Importing JBox2D #
## Jar Releases ##
The jar releases have all dependencies included, so you just need to include them as an external library in your project.  This is a common option with most IDE's, and usually consists of:
  1. Placing the jar in your project
  1. Right clicking and choosing to include the jar in the build path.

If you need more help with this, please see help sources for your IDE.  If you're not using an IDE, then you shouldn't need help with this.

## Maven ##
JBox2D is released in Maven (or will be soon), so you should see it listed as an available dependency.

# Building JBox2D #
## Maven ##
The most recent versions of JBox2D lean on Maven to help handle dependency management, and if your IDE supports Maven, then this should be pretty easy.  To quickly get a project running you:


  1. Check out the project through Subversion:
    1. For 2.2.1.1 tag:  `svn checkout http://jbox2d.googlecode.com/svn/tags/jbox2d-2.2.1.1/ jbox2d-read-only`
    1. For trunk `svn checkout http://jbox2d.googlecode.com/svn/trunk jbox2d-read-only`
    1. **The trunk is recommended for now, as it represents the 2.2.1 release branch.**
  1. Import to your IDE as a Maven project (using the pom.xml descriptor in the root folder)
  1. Run the `org.jbox2d.testbed.framework.TestbedMain` class

Note: the trunk is pretty stable at this time.

If you're planning on doing some more in-depth customization, we would suggest adding each module project independently (jbox2d-library, jbox2d-serialization, jbox2d-testbed) and putting all of those projects in your workspace.

**Note:**
  * You might receive errors about the jbox2d parent POM is not found.  In this case you'll have to run 'mvn install" on the root project (checked out like above).
  * If you're changing the protobuffer definition for serialization, you'll need the protobuffer library installed on your machine.  Download it [here](http://code.google.com/apis/protocolbuffers/docs/overview.html).

## Non Maven ##
You'll have to include the dependencies yourself if you don't want to use Maven.  Fortunately, there are only a couple:

  1. Standard Logging Facade for Java ([SLF4J](http://www.slf4j.org/))
  1. [Protobuffers](http://code.google.com/p/protobuf/downloads/list) for serialization (you might have to build it)

Include those and you should be good!

## Quick IDE Setups ##
### Eclipse (tested with [Eclipse Helios](http://www.eclipse.org/downloads/packages/release/helios/sr2)) ###

  1. Check out the project through Subversion (you can also do this within Eclipse using the [Subclipse](http://subclipse.tigris.org/) plugin if you don't like the command line):
> > `svn checkout http://jbox2d.googlecode.com/svn/trunk/ jbox2d-read-only`
  1. Make sure you have the [Maven Integration for Eclipse](http://maven.apache.org/eclipse-plugin.html) plugin installed
  1. In Eclipse, go to _File_->_Import_ and select _Maven_->_Existing Maven Projects_
  1. Navigate to the directory that you checked out from Subversion (jbox2d-read-only if you cut and pasted the example above) and hit _Next_
  1. Select both of the projects, and add them to your working sets if desired
  1. Finish the import
  1. In Package Explorer, find the jbox2d-testbed project, navigate to _src/main/java_ and find the _org.jbox2d.testbed.framework_ package - right click on TestbedMain.java and run it as a Java application.
  1. If the JBox2D testbed runs, congratulations, you've successfully set up the project!  You can add the jbox2d-library project as a reference to other projects to use it, or you can browse through the jbox2d-testbed project to see how things are done (navigate to the org.jbox2d.testbed.tests directory to see code examples)

**NOTE** make sure the src/generated folder is included if you're using the serialization library
### NetBeans ###
R.A. Nagy made a blog post [here](http://www.soft9000.com/blog9000/index.php?entry=Angry-Buddha-Springs?) about importing/building with NetBeans. Thanks R.A. Nagy!

# Using JBox2D #
**Please** see the [Testbed](Testbed.md) wiki article.