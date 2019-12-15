# Inspector Pom - a command line Maven pom.xml inspector tool

> Because nobody likes to squint at oodles of XML just to check a version
> number.

Inspector Pom is a lightweight developer tool to extract and display data from
Maven POM files as plain text. It is meant for quick inspection of `pom.xml`-s
in a terminal. By default, the tool works without Maven, simply extracting and
reformatting relevant parts of POM xml files. In this mode, output is almost
instant. It does its own XML-parsing using `xsltproc`.

There are also options for resolving dependencies transitively before displaying
POM information, and even have Maven generate the entire effective POM and
display data from it. These two modes are a bit slower, since they invoke Maven
in the background before extracting and displaying information. But they can be
incredibly useful.

![Screenshot](https://stegard.net/dl/pom-screenshot.png)


## Requirements

1. A plain Bourne-compatible shell (`/bin/sh`). 

2. xsltproc for XML-parsing

3. Maven (`mvn`) if you want to use options for resolving dependencies or
   generate effective POMs.


## Installation

    git clone https://github.com/oyvindstegard/pom
    cd pom
    chmod +x pom
    cp pom <to somewhere in your PATH>


## Usage examples

Simply invoke `pom` in a project root directory or any subdir, and it searches
for the nearest `pom.xml` file (upwards) and displays essential data as nicely
formatted plain text:

```
$ pwd
/home/oyvind/dev/kafka-sandbox
$ pom
POM file         ~/dev/kafka-sandbox/pom.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT
Packaging        jar

Build plugins:
  org.apache.maven.plugins:maven-compiler-plugin:3.8.1
  org.apache.maven.plugins:maven-shade-plugin:3.2.1
  org.apache.maven.plugins:maven-surefire-plugin:3.0.0-M3

Properties:
  jackson.version = 2.10.0
  java.version = 11
  junit.version = 5.5.2
  kafka.version = 2.3.1
  project.build.sourceEncoding = UTF-8
  slf4j.version = 1.7.28

Direct dependencies:
  com.fasterxml.jackson.core:jackson-annotations:${jackson.version}
  com.fasterxml.jackson.core:jackson-core:${jackson.version}
  com.fasterxml.jackson.core:jackson-databind:${jackson.version}
  com.fasterxml.jackson.datatype:jackson-datatype-jdk8:${jackson.version}
  com.fasterxml.jackson.datatype:jackson-datatype-jsr310:${jackson.version}
  org.apache.kafka:kafka-clients:${kafka.version}
  org.slf4j:slf4j-api:${slf4j.version}
  org.slf4j:slf4j-simple:${slf4j.version}

Direct test dependencies:
  org.junit.jupiter:junit-jupiter-api:${junit.version}
  org.junit.jupiter:junit-jupiter-engine:${junit.version}
```

Maven is not invoked in this case, and so the command is quick. If terminal is
wide enough, the listings will be formatted in two columns for vertical space
efficiency. Test scoped dependencies are showed separate from dependencies in
other scopes.

### Resolving dependencies before display

```
$ pom -r
Invoking Maven to resolve dependencies ..
POM file         ~/dev/kafka-sandbox/pom.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT
Packaging        jar

Build plugins:
  org.apache.maven.plugins:maven-compiler-plugin:3.8.1
  org.apache.maven.plugins:maven-shade-plugin:3.2.1
  org.apache.maven.plugins:maven-surefire-plugin:3.0.0-M3

Properties:
  jackson.version = 2.10.0
  java.version = 11
  junit.version = 5.5.2
  kafka.version = 2.3.1
  project.build.sourceEncoding = UTF-8
  slf4j.version = 1.7.28

Resolved dependencies:
  com.fasterxml.jackson.core:jackson-annotations:2.10.0
  com.fasterxml.jackson.core:jackson-core:2.10.0
  com.fasterxml.jackson.core:jackson-databind:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jdk8:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.10.0
  com.github.luben:zstd-jni:1.4.0-1
  org.apache.kafka:kafka-clients:2.3.1
  org.lz4:lz4-java:1.6.0
  org.slf4j:slf4j-api:1.7.28
  org.slf4j:slf4j-simple:1.7.28
  org.xerial.snappy:snappy-java:1.1.7.3

Resolved test dependencies:
  org.apiguardian:apiguardian-api:1.1.0
  org.junit.jupiter:junit-jupiter-api:5.5.2
  org.junit.jupiter:junit-jupiter-engine:5.5.2
  org.junit.platform:junit-platform-commons:1.5.2
  org.junit.platform:junit-platform-engine:1.5.2
  org.opentest4j:opentest4j:1.2.0
```

Observe that the dependencies have been resolved, including version numbers that
used property variables and transitive dependencies. This is handy to have a
quick look at what the POM actually pulls in.

### Resolving entire effective POM before display

```
$ pom -e
Invoking Maven to generate effective POM ..


POM file         /tmp/effective-pom.nnKUUP.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT

Defined repository URLs:
  https://repo.maven.apache.org/maven2
  https://repo.maven.apache.org/maven2  (Plugins)

Build plugins:
  maven-clean-plugin:2.5       maven-resources-plugin:2.6
  maven-compiler-plugin:3.8.1  maven-shade-plugin:3.2.1
  maven-deploy-plugin:2.7      maven-site-plugin:3.3
  maven-install-plugin:2.4     maven-surefire-plugin:3.0.0-M3
  maven-jar-plugin:2.4

Properties:
  jackson.version = 2.10.0
  java.version = 11
  junit.version = 5.5.2
  kafka.version = 2.3.1
  project.build.sourceEncoding = UTF-8
  slf4j.version = 1.7.28

Direct dependencies:
  com.fasterxml.jackson.core:jackson-annotations:2.10.0
  com.fasterxml.jackson.core:jackson-core:2.10.0
  com.fasterxml.jackson.core:jackson-databind:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jdk8:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.10.0
  org.apache.kafka:kafka-clients:2.3.1
  org.slf4j:slf4j-api:1.7.28
  org.slf4j:slf4j-simple:1.7.28

Direct test dependencies:
  org.junit.jupiter:junit-jupiter-api:5.5.2
  org.junit.jupiter:junit-jupiter-engine:5.5.2

```

Notice that version numbers of dependencies have been resolved, and default
plugins show up, even though they may not be directly defined in the POM file
under inspection. This is because Maven generates an effective POM file which
contains everything explicitly.

### Both resolving dependencies and generating effecting POM

You can combine options `-r` and `-e` to both resolve all dependencies
transitively and generate an effective POM. This gives a very good overview of
the POM model that is actually used during builds.

```
$ pom -re
Invoking Maven to resolve dependencies ..
Invoking Maven to generate effective POM ..


POM file         /tmp/effective-pom.1s0eQR.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT

Defined repository URLs:
  https://repo.maven.apache.org/maven2
  https://repo.maven.apache.org/maven2  (Plugins)

Build plugins:
  maven-clean-plugin:2.5       maven-resources-plugin:2.6
  maven-compiler-plugin:3.8.1  maven-shade-plugin:3.2.1
  maven-deploy-plugin:2.7      maven-site-plugin:3.3
  maven-install-plugin:2.4     maven-surefire-plugin:3.0.0-M3
  maven-jar-plugin:2.4         

Properties:
  jackson.version = 2.10.0
  java.version = 11
  junit.version = 5.5.2
  kafka.version = 2.3.1
  project.build.sourceEncoding = UTF-8
  slf4j.version = 1.7.28

Resolved dependencies:
  com.fasterxml.jackson.core:jackson-annotations:2.10.0
  com.fasterxml.jackson.core:jackson-core:2.10.0
  com.fasterxml.jackson.core:jackson-databind:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jdk8:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.10.0
  com.github.luben:zstd-jni:1.4.0-1
  org.apache.kafka:kafka-clients:2.3.1
  org.lz4:lz4-java:1.6.0
  org.slf4j:slf4j-api:1.7.28
  org.slf4j:slf4j-simple:1.7.28
  org.xerial.snappy:snappy-java:1.1.7.3

Resolved test dependencies:
  org.apiguardian:apiguardian-api:1.1.0
  org.junit.jupiter:junit-jupiter-api:5.5.2
  org.junit.jupiter:junit-jupiter-engine:5.5.2
  org.junit.platform:junit-platform-commons:1.5.2
  org.junit.platform:junit-platform-engine:1.5.2
  org.opentest4j:opentest4j:1.2.0

```

### Using shortcuts to only display desired parts of a POM

You can limit the output by adding filters (or shortcuts) to pom commands:

```
$ pom basic props
POM file         ~/dev/kafka-sandbox/pom.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT
Packaging        jar

Properties:
  jackson.version = 2.10.0
  java.version = 11
  junit.version = 5.5.2
  kafka.version = 2.3.1
  project.build.sourceEncoding = UTF-8
  slf4j.version = 1.7.28
```

Combine `pom` with `grep` for very exact extractions of information. (And let
Inspector Pom worry about parsing the XML, which can be a pain to grep
directly.)

### Colorized output

There is experimental support for colorized output to improve readability. This
deemphasizes group-ids and highlights version numbers and section headings. Use
option `-c` to enable.

To always use colorized output, add this to `~/.bashrc` or similar:

    alias pom='pom -c'


## Command help

```
$ pom -h
Inspector Pom 0.9
This command quickly extracts and displays essential data from a Maven POM file.
The output is compact and readable plain text, optionally colorized.

Usage: pom [options] [pomfile/dir] [shortcut [..]]

By default, the first POM file found in either the current directory or any
parent directory is processed, and all information is displayed. If path to an
existing directory or file is supplied, then that is used as basis for POM file
resolving instead.

If the POM file directly refers to a parent POM by path, the parent POM is also
processed and displayed.

Shortcuts can be used to filter the types of information displayed about the POM
file.

Options:
-r      Invoke Maven to resolve all regular dependencies before display.
-e      Invoke Maven to generate effective POM and display that instead.
        Can be combined with '-r' to show both all resolved transitive 
        dependencies and the effective POM.
-P <id> Show data for given POM profile.
        This will typically not show global data common to all profiles.
-n      Avoid traversing parent POM files with local path reference.
-1      Always use single-column listings.
-c      Enable experimental colorized output.
-h      Show this help.
-v      Show version of this utility.

Shortcuts/filters:
ba[sic]        Show basic project info.
re[pos]        Show defined repositories.
di[st]         Show distribution repository.
pl[ugins]      Show build plugins.
de[ps]         Show direct dependencies.
ex[clusions]   Show transitive exclusions for direct dependencies.
pr[ops]        Show properties.
mo[dules]      Show modules.
all            Show everything (default).
```

## Contributing

Pull requests and issue reports are welcome. The tool has mostly been tested on
Linux-like environments, but should also work fine for MacOSX/BSD.
