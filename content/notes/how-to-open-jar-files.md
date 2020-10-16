---
menu: JAR Files
title: JAR File - What It Is & How to Open One
bookToc: false
---

# JAR File - What It Is & How to Open One

A JAR (**J**ava **AR**chive) file is a package file format, based on the ZIP file format, used to aggregate many Java class files, metadata and resources (text, images, etc.) into one file for distribution.[^jar-spec]

The contents of a JAR file can be extracted using any standard decompression tool, or the `jar` command line utility.

### unzip
```sh
unzip -d foo foo.jar
```
{{<details "Parameters">}}
- `-d <dir>`: extract files into dir.
{{</details>}}

### jar
```sh
jar -xf foo.jar
```
{{<details "Parameters">}}
- `-x`: extract named (or all) files from archive.
- `-f <file>`: specify archive file name.
{{</details>}}

## Decompile Class Files

Java class files are compiled files containing Java bytecode that can be executed by a Java Virtual Machine (JVM). To decompile and view the source code in plain text any Java decompiler can be used.
[JAD](https://varaneckas.com/jad/) was the go-to for many years but only supports Java 1.4 or below.
The [Java-Decompiler Project](https://java-decompiler.github.io/), [Procyon](https://bitbucket.org/mstrobel/procyon/wiki/Java%20Decompiler), and online tools like [DevToolZone's Java Decompiler](https://devtoolzone.com/decompiler/java) are good alternatives.

### procyon

```sh
procyon foo.class
```

[^jar-spec]: “JAR File Specification.” JAVA SE Documentation, https://docs.oracle.com/javase/6/docs/technotes/guides/jar/jar.html#JARIndex.
