<!--

********************************************************************************

WARNING:

    DO NOT EDIT "openjdk/README.md"

    IT IS AUTO-GENERATED

    (from the other files in "openjdk/" combined with a set of templates)

********************************************************************************

-->

# Supported tags and respective `Dockerfile` links

## Simple Tags

-	[`11.0.3-jdk-stretch`, `11.0.3-stretch`, `11.0-jdk-stretch`, `11.0-stretch`, `11-jdk-stretch`, `11-stretch` (*11/jdk/Dockerfile*)](https://github.com/docker-library/openjdk/blob/1733ab9061cf1a076dc0566e40edb600fc434db4/11/jdk/Dockerfile)
-	[`11.0.3-jdk-slim-stretch`, `11.0.3-slim-stretch`, `11.0-jdk-slim-stretch`, `11.0-slim-stretch`, `11-jdk-slim-stretch`, `11-slim-stretch`, `11.0.3-jdk-slim`, `11.0.3-slim`, `11.0-jdk-slim`, `11.0-slim`, `11-jdk-slim`, `11-slim` (*11/jdk/slim/Dockerfile*)](https://github.com/docker-library/openjdk/blob/1733ab9061cf1a076dc0566e40edb600fc434db4/11/jdk/slim/Dockerfile)

## Shared Tags

-	`11.0.3-jdk`, `11.0.3`, `11.0-jdk`, `11.0`, `11-jdk`, `11`:
	-	[`11.0.3-jdk-stretch` (*11/jdk/Dockerfile*)](https://github.com/docker-library/openjdk/blob/1733ab9061cf1a076dc0566e40edb600fc434db4/11/jdk/Dockerfile)

[![Build Status](https://doi-janky.infosiftr.net/job/multiarch/job/arm64v8/job/openjdk/badge/icon) (`arm64v8/openjdk` build job)](https://doi-janky.infosiftr.net/job/multiarch/job/arm64v8/job/openjdk/)

# Quick reference

-	**Where to get help**:  
	[the Docker Community Forums](https://forums.docker.com/), [the Docker Community Slack](https://blog.docker.com/2016/11/introducing-docker-community-directory-docker-community-slack/), or [Stack Overflow](https://stackoverflow.com/search?tab=newest&q=docker)

-	**Where to file issues**:  
	[https://github.com/docker-library/openjdk/issues](https://github.com/docker-library/openjdk/issues)

-	**Maintained by**:  
	[the Docker Community](https://github.com/docker-library/openjdk)

-	**Supported architectures**: ([more info](https://github.com/docker-library/official-images#architectures-other-than-amd64))  
	[`amd64`](https://hub.docker.com/r/amd64/openjdk/), [`arm64v8`](https://hub.docker.com/r/arm64v8/openjdk/), [`windows-amd64`](https://hub.docker.com/r/winamd64/openjdk/)

-	**Published image artifact details**:  
	[repo-info repo's `repos/openjdk/` directory](https://github.com/docker-library/repo-info/blob/master/repos/openjdk) ([history](https://github.com/docker-library/repo-info/commits/master/repos/openjdk))  
	(image metadata, transfer size, etc)

-	**Image updates**:  
	[official-images PRs with label `library/openjdk`](https://github.com/docker-library/official-images/pulls?q=label%3Alibrary%2Fopenjdk)  
	[official-images repo's `library/openjdk` file](https://github.com/docker-library/official-images/blob/master/library/openjdk) ([history](https://github.com/docker-library/official-images/commits/master/library/openjdk))

-	**Source of this description**:  
	[docs repo's `openjdk/` directory](https://github.com/docker-library/docs/tree/master/openjdk) ([history](https://github.com/docker-library/docs/commits/master/openjdk))

-	**Supported Docker versions**:  
	[the latest release](https://github.com/docker/docker-ce/releases/latest) (down to 1.6 on a best-effort basis)

# What is OpenJDK?

OpenJDK (Open Java Development Kit) is a free and open source implementation of the Java Platform, Standard Edition (Java SE). OpenJDK is the official reference implementation of Java SE since version 7.

> [wikipedia.org/wiki/OpenJDK](http://en.wikipedia.org/wiki/OpenJDK)

Java is a registered trademark of Oracle and/or its affiliates.

![logo](https://raw.githubusercontent.com/docker-library/docs/a3439b66b7980d1811f6b3835a3c541747172970/openjdk/logo.png)

# How to use this image

## Start a Java instance in your app

The most straightforward way to use this image is to use a Java container as both the build and runtime environment. In your `Dockerfile`, writing something along the lines of the following will compile and run your project:

```dockerfile
FROM arm64v8/openjdk:7
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac Main.java
CMD ["java", "Main"]
```

You can then run and build the Docker image:

```console
$ docker build -t my-java-app .
$ docker run -it --rm --name my-running-app my-java-app
```

## Compile your app inside the Docker container

There may be occasions where it is not appropriate to run your app inside a container. To compile, but not run your app inside the Docker instance, you can write something like:

```console
$ docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp arm64v8/openjdk:7 javac Main.java
```

This will add your current directory as a volume to the container, set the working directory to the volume, and run the command `javac Main.java` which will tell Java to compile the code in `Main.java` and output the Java class file to `Main.class`.

## Make JVM respect CPU and RAM limits

On startup JVM tries to detect the number of available CPU cores and the amount of RAM to adjust its internal parameters (like the number of garbage collector threads to spawn) accordingly. When container is run with limited CPU/RAM, standard system API, used by JVM for probing, will return host-wide values. This can cause excessive CPU usage and memory allocation errors with older versions of JVM.

Inside Linux containers, OpenJDK versions 8 and later can correctly detect container-limited number of CPU cores and available RAM. In OpenJDK 11 this is turned on by default. In versions 8, 9, and 10 you have to enable the detection of container-limited amount of RAM using the following options:

```console
$ java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap ...
```

Inside Windows Server (non-Hyper-V) containers, limit for number of available CPU cores does not work (is ignored by Host Compute Service). To set such limit manually, JVM can be started the following way:

```console
$ start /b /wait /affinity 0x3 path/to/java.exe ...
```

In this example CPU affinity hex mask `0x3` will limit JVM to 2 CPU cores.

RAM limit is supported by Windows Server containers, but currently JVM cannot detect it. To prevent excessive memory allocations, `-XX:MaxRAM=...` option must be specified with the value that is not bigger than a containers RAM limit.

## Environment variables with periods in their names

Some shells (notably, [the BusyBox `/bin/sh` included in Alpine Linux](https://github.com/docker-library/openjdk/issues/135)) do not support environment variables with periods in the names (which are technically not POSIX compliant), and thus *strip* them instead of passing them through (as Bash does). If your application requires environment variables of this form, either use `CMD ["java", ...]` directly (no shell), or (install and) use Bash explicitly instead of `/bin/sh`.

# License

View [license information](http://openjdk.java.net/legal/gplv2+ce.html) for the software contained in this image.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

Some additional license information which was able to be auto-detected might be found in [the `repo-info` repository's `openjdk/` directory](https://github.com/docker-library/repo-info/tree/master/repos/openjdk).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
