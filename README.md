# pom - Maven shallow pom.xml inspector

> Because nobody likes to squint at oodles of XML just to check a version
> number.

Lightweight command line tool to extract and display data from Maven POM files
as plain text. Meant for quick inspection of pom.xml-s in a terminal. By
default, the tool works without Maven, simply extracting and reformatting
relevant parts of POM xml files. In this mode, output is almost always instant.

However, there are also options for resolving dependencies transitively before
displaying POM information, and even have Maven generate the entire effective
POM and display data from it. These two modes are a bit slower, since they
invoke Maven in the background before extracting and displaying information.


## Requirements

1. A plain Bourne-compatible shell (`/bin/sh`), bash is not required.

2. xsltproc for XML-parsing

3. Maven (`mvn`) if you want to use options for resolving dependencies or
   effective POM.


## Examples

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
  org.junit.jupiter:junit-jupiter-api:${junit.version}
  org.junit.jupiter:junit-jupiter-engine:${junit.version}
  org.slf4j:slf4j-api:${slf4j.version}
  org.slf4j:slf4j-simple:${slf4j.version}
```

If terminal is wide enough, the information will be formatted in 2 columns for
vertical space efficiency.

### Resolving dependencies before display

```
$ pom -R
Invoking Maven to resolve dependencies ..
POM file         ~/dev/kafka-sandbox/pom.xml
GroupId          no.nav.kafka
ArtifactId       kafka-sandbox
Version          1.0-SNAPSHOT

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

All resolved dependencies:
  com.fasterxml.jackson.core:jackson-annotations:2.10.0
  com.fasterxml.jackson.core:jackson-core:2.10.0
  com.fasterxml.jackson.core:jackson-databind:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jdk8:2.10.0
  com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.10.0
  com.github.luben:zstd-jni:1.4.0-1
  org.apache.kafka:kafka-clients:2.3.1
  org.apiguardian:apiguardian-api:1.1.0
  org.junit.jupiter:junit-jupiter-api:5.5.2
  org.junit.jupiter:junit-jupiter-engine:5.5.2
  org.junit.platform:junit-platform-commons:1.5.2
  org.junit.platform:junit-platform-engine:1.5.2
  org.lz4:lz4-java:1.6.0
  org.opentest4j:opentest4j:1.2.0
  org.slf4j:slf4j-api:1.7.28
  org.slf4j:slf4j-simple:1.7.28
  org.xerial.snappy:snappy-java:1.1.7.3
```

Observe that the dependencies have been resolved, including transitive
dependencies ! This is handy to have a quick look at what the POM pulls in.

### Resolving entire effective POM before display




## Command help

```
$ pom -h
Maven POM file inspector 0.7
This command quickly extracts and displays essential data from a Maven POM file.
The output is compact and readable plain text.

Usage: pom [options] [pomfile/dir] [shortcut [..]]

By default, the first POM file found in either the current directory
or any parent directory is processed, and all information is displayed. If path
to an existing directory or file is supplied, then that is used as basis for POM
file resolving instead.

If the POM file directly refers to a parent POM by path, the parent POM is also
processed and displayed.

Shortcuts can be used to filter the types of information displayed about the POM
file.

Options:
-1, --single       Always use single-column listings.
-P, --profile <id> Show data for given POM profile.
                   This will not show global data common to all profiles.
-E, --effective    Invoke Maven to generate effective POM and display that instead.
-R, --resolve      Invoke Maven to resolve all regular dependencies before display.
-N, --noparents    Avoid traversing parent POM files with local path reference.
-v, --version      Show version of this utility.
-h, --help         Show this help.

Shortcuts/filters:
ba[sic]        Show basic project info.
re[pos]        Show defined repositories.
di[st]         Show distribution repository.
pl[ugins]      Show build plugins.
de[ps]         Show direct dependencies.
ex[clusions]   Show transitive exclusions for direct dependencies.
pr[ops]        Show properties.
all            Show everything (DEFAULT).
```
