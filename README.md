## Build Instructions

1. git clone this repository:
`git clone https://github.com/lixuanbin/compile-openjdk-in-docker.git`
2. get in a specific directory, for example:
`cd ubuntu1404_openjdk8`
3. build the Docker image:
`docker build -t ubuntu1404_openjdk8:version3 .`
4. make local directory and run the image:
`mkdir -p ~/docker_share/ubuntu1404_openjdk8 && docker run -v ~/docker_share/ubuntu1404_openjdk8:/app -ti --entrypoint /bin/sh ubuntu1404_openjdk8:version3`
5. [inside container] copy the source and binaries to the mounted volume to persist your work:
`cp -r /opt/openjdk /app/ && cd /app/openjdk/openjdk8`
6. [inside container] alter the source codes and re-compile it, or you can use gdb to debug the source codes, have fun :)

## Using Fine-Grained Make Targets
The default behavior for make is to create consistent and correct output, at the expense of build speed, if necessary.

If you are prepared to take some risk of an incorrect build, and know enough of the system to understand how things build and interact, you can speed up the build process considerably by instructing make to only build a portion of the product.

* Building Individual Modules

The safe way to use fine-grained make targets is to use the module specific make targets. All source code in JDK 9 is organized so it belongs to a module, e.g. java.base or jdk.jdwp.agent. You can build only a specific module, by giving it as make target: make jdk.jdwp.agent. If the specified module depends on other modules (e.g. java.base), those modules will be built first.

You can also specify a set of modules, just as you can always specify a set of make targets: `make jdk.crypto.cryptoki jdk.crypto.ec jdk.crypto.mscapi jdk.crypto.ucrypto`

* Building Individual Module Phases

The build process for each module is divided into separate phases. Not all modules need all phases. Which are needed depends on what kind of source code and other artifact the module consists of. The phases are:

>gensrc (Generate source code to compile)
gendata (Generate non-source code artifacts)
copy (Copy resource artifacts)
java (Compile Java code)
launchers (Compile native executables)
libs (Compile native libraries)
rmic (Run the rmic tool)
You can build only a single phase for a module by using the notation $MODULE-$PHASE. For instance, to build the gensrc phase for java.base, use make java.base-gensrc.

Note that some phases may depend on others, e.g. java depends on gensrc (if present). Make will build all needed prerequisites before building the requested phase.

* Skipping the Dependency Check

When using an iterative development style with frequent quick rebuilds, the dependency check made by make can take up a significant portion of the time spent on the rebuild. In such cases, it can be useful to bypass the dependency check in make.

Note that if used incorrectly, this can lead to a broken build!

To achieve this, append -only to the build target. For instance, make jdk.jdwp.agent-java-only will only build the java phase of the jdk.jdwp.agent module. If the required dependencies are not present, the build can fail. On the other hand, the execution time measures in milliseconds.

A useful pattern is to build the first time normally (e.g. make jdk.jdwp.agent) and then on subsequent builds, use the -only make target.

* Rebuilding Part of java.base (JDK_FILTER)

If you are modifying files in java.base, which is the by far largest module in OpenJDK, then you need to rebuild all those files whenever a single file has changed. (This inefficiency will hopefully be addressed in JDK 10.)

As a hack, you can use the make control variable JDK_FILTER to specify a pattern that will be used to limit the set of files being recompiled. For instance, make java.base JDK_FILTER=javax/crypto (or, to combine methods, make java.base-java-only JDK_FILTER=javax/crypto) will limit the compilation to files in the javax.crypto package.

## Build Output Structure
The build output for a configuration will end up in build/<configuration name>, which we refer to as $BUILD in this document. The $BUILD directory contains the following important directories:

>buildtools/
configure-support/
hotspot/
images/
jdk/
make-support/
support/
test-results/
test-support/

This is what they are used for:

* images: This is the directory were the output of the *-image make targets end up. For instance, make jdk-image ends up in images/jdk.

* jdk: This is the "exploded image". After make jdk, you will be able to launch the newly built JDK by running $BUILD/jdk/bin/java.

* test-results: This directory contains the results from running tests.

* support: This is an area for intermediate files needed during the build, e.g. generated source code, object files and class files. Some noteworthy directories in support is gensrc, which contains the generated source code, and the modules_* directories, which contains the files in a per-module hierarchy that will later be collapsed into the jdk directory of the exploded image.

* buildtools: This is an area for tools compiled for the build platform that are used during the rest of the build.

* hotspot: This is an area for intermediate files needed when building hotspot.

* configure-support, make-support and test-support: These directories contain files that are needed by the build system for configure, make and for running tests.

## References
* [OpenJDK 8 build readme](https://hg.openjdk.java.net/jdk8/jdk8/raw-file/tip/README-builds.html)

* [OpenJDK 9 build readme](https://hg.openjdk.java.net/jdk-updates/jdk9u/raw-file/tip/common/doc/building.html)

* [Supported build platforms](https://wiki.openjdk.java.net/display/Build/Supported+Build+Platforms)

* [AdoptOpenJDK Dockerfile](https://github.com/AdoptOpenJDK/openjdk-build/blob/master/docker/jdk8/x86_64/ubuntu/Dockerfile)

* [os not supported error while building hotspot](https://stackoverflow.com/questions/14285820/os-not-supported-error-while-building-hotspot)

* [Compile JDK8 error.''Could not find freetype''](https://stackoverflow.com/questions/52377684/compile-jdk8-error-could-not-find-freetype)

* [Debugging OpenJDK 8 with NetBeans on Ubuntu](https://marcin-chwedczuk.github.io/debugging-openjdk8-with-netbeans-on-ubuntu)

* [GDB: Debug native part of java application (C/C++ libraries and JDK)](https://medium.com/@pirogov.alexey/gdb-debug-native-part-of-java-application-c-c-libraries-and-jdk-6593af3b4f3f)