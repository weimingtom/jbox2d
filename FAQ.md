**NOTE:** If your question has to do with the engine behavior itself, please also look [here](http://www.box2d.org/forum/viewtopic.php?f=9&t=558).  Most of these have to do specifically with the jbox2d distribution.

**NOTE #2:** Try running with the `-ea` flag to enable assertions and cause the engine to crash when methods are called incorrectly (ex: calling `destroyBody()` inside of a contact listener).



# Building #
## What is the project structure?  What's with all of these sub-projects? ##
JBox2D uses Maven for building.  Most IDE's support maven, eclipse-specific instructions are in the getting started wiki page, but basically you go to 'import' then 'import maven projects'.  Navigate to the jbox2d folder, and import all of the projects.

## How do I customize the settings? ##
Most of the engine-specific settings are `static final` for performance.  If you need to customize these settings (we recommend you don't), build from the source and look at the Settings.java file in the common package of the library.

## I don't care about having my own build, where are the jars? ##
Take a look in the `target` folder of each project, there should be jars there.

## I'm have trouble with `protoc` failing when I build the serialization project, what is that?  Does that matter? ##
That project requires the [protocol buffer](http://code.google.com/p/protobuf/) library to be installed.  This is only required if you change things in the .proto file for proto serialization.  Otherwise, the sources are already generated and in source control.

# Learning #
## How does the engine work?  How do I get it to do stuff I want? ##
Please look at the [Testbed](Testbed.md) wiki page.

# Running #
## I want to see some examples.  How do I run the testbed? ##
Grab the lastest version and navigate to the `jbox2d-testbed` folder.  This is the testbed project.  There should be an executable jar in the `target` folder, with the suffix "with-dependencies".  Double click that one, or run it from the command line with the command `java -jar jbox2d-<version>-with-dependencies.jar`

## I want to make something with this engine.  What's the best way to start? ##
It is HIGHLY recommended to start out by making a testbed test.  The purpose of the testbed is to provide all of the graphics, animation, and settings, so you only have to work with the physics engine itself.  Take a look at the [Testbed](Testbed.md) page for more info.

## What is the SLF4J error?  Static logger? ##
This does **not** mean anything is wrong.  It is just letting you know that the library uses [SLF4J](http://www.slf4j.org/) to log messages, and there isn't a logger binding available.  If you wish to see error messages, include a logging jar and config file (See the website for details).

## I'm getting `null` when I try to create bodies, or calling `destroyBody()` has no effect ##
This is because the world is currently locked, which happens when `step()`.  This means you're probably inside of a contact listener callback.  You're going to have to somehow buffer your decisions or contact collisions so you can process them outside of the engine.  See the box2d manual for more info.

# Bugs #
## I think I found a bug, what do I do? ##
Please report all bugs in the issues tab.  We greatly appreciate if you could:
  * Make a testbed test of the bug.
  * Give the smallest example possible that demonstrates the bug.

This allows us to easily run the test and see the bug, and ensures we fix it as quickly as possible.

# Contributing #
## I want to help out, what do I do? ##
First of all, that's awesome! We'd love your help. Please contact Daniel Murphy (danielmurphy161 at gmail).

## I want to support the project! ##
Awesome!  We'll love you forever!  See the [webpage](http://jbox2d.org) for donation links.