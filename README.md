*We can help you get Chronicle up and running in your organisation, we suggest you invite us in for consultancy, charged on an ad-hoc basis, we can discuss the best options tailored to your individual requirements. - [Contact Us](sales@chronicle.software)*

*Or you may already be using Chronicle and just want some help - [find out more..](http://chronicle.software/support/)*

# Chronicle Queue

Inter Process Communication ( IPC ) with sub millisecond latency and able to store every message:
![Chronicle](http://chronicle.software/wp-content/uploads/2014/07/ChronicleQueue_200px.png)


Releases are available on maven central as

 ```xml
<dependency>
  <groupId>net.openhft</groupId>
  <artifactId>chronicle-queue</artifactId>
  <version><!--replace with the latest version, see below--></version>
</dependency>
```
Click here to get the [Latest Version Number](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22net.openhft%22%20AND%20a%3A%22chronicle-queue%22)

Snapshots are available on [OSS sonatype](https://oss.sonatype.org/content/repositories/snapshots/net/openhft/chronicle-queue)



### Contents
* [Overview](https://github.com/OpenHFT/Chronicle-Queue#overview)
* [Building Blocks](https://github.com/OpenHFT/Chronicle-Queue#building-blocks)
* [Chronicle Queue V4](https://github.com/OpenHFT/Chronicle-Queue#chronicle-queue-v3)
  * [Single Chronicle Queue](https://github.com/OpenHFT/Chronicle-Queue#vanilla-chronicle)
  * [Getting Started](https://github.com/OpenHFT/Chronicle-Queue#getting-started)
  * [Replication](https://github.com/OpenHFT/Chronicle-Queue#replication)
* [Support](https://github.com/OpenHFT/Chronicle-Queue#support)
* [JavaDoc](http://openhft.github.io/Chronicle-Queue/apidocs/)

## Overview
Chronicle Queue is a Java project focused on building a persisted low latency messaging framework for high performance and critical applications.

![](http://chronicle.software/wp-content/uploads/2014/07/Chronicle-diagram_005.jpg)

At first glance Chronicle Queue can be seen as **yet another queue implementation** but it has major design choices that should be emphasised.

Using non-heap storage options (RandomAccessFile) Queue provides a processing environment where applications do not suffer from Garbage Collection. While implementing high performance and memory-intensive applications (you heard the fancy term "bigdata"?) in Java; one of the biggest problems is Garbage Collection. Garbage Collection (GC) may slow down your critical operations non-deterministically at any time. In order to avoid non-determinism and escape from GC delays off-heap memory solutions are ideal. The main idea is to manage your memory manually so it does not suffer from GC. Chronicle behaves like a management interface over off-heap memory so you can build your own solutions over it.
Queue uses RandomAccessFiles while managing memory and this choice brings lots of possibilities. RandomAccessFiles permit non-sequential, or random, access to a file's contents. To access a file randomly, you open the file, seek a particular location, and read from or write to that file. RandomAccessFiles can be seen as "large" C-type byte arrays that you can access at any random index "directly" using pointers. File portions can be used as ByteBuffers if the portion is mapped into memory.

This memory mapped file is also used for exceptionally fast interprocess communication (IPC) without affecting your system performance. There is no Garbage Collection (GC) as everything is done off heap.

![](http://chronicle.software/wp-content/uploads/2014/07/Screen-Shot-2014-09-30-at-11.24.53.png)

### Use cases

- Log everything logging. (Fast enough to record everything)
- Event sourcing.
- Log aggregation.
- Stream processing
- Commit log.
- Metrics

## Building Blocks
Chronicle Queue is the main interface for management and can be seen as the Collection class of Chronicle environment. You will reserve a portion of memory and then put/fetch/update records using the Chronicle interface.

Chronicle has three main concepts:
  * Tailer (sequential and random reads, forward and backwards)
  * Appender (sequential writes, append to the end only).

An Excerpt is the main data container in a Chronicle Queue, each Chronicle is composed of Excerpts. Putting data to a chronicle means starting a new Excerpt, writing data into it and finishing the Excerpt at the end.
A Tailer is an Excerpt optimized for sequential reads.
An Appender is something like Iterator in Chronicle environment. You add data appending the current chronicle.

## Chronicle Queue V4

Current version of Chronicle-Queue (V4) contains the SingleChronicleQueue implementation.

### Single Chronicle Queue
This queue is a designed to support:
 - rolling files on a daily, weekly or hourly basis.
 - concurrent writers on the same machine.
 - concurrent readers on the same machine or across multiple machines via TCP replication (With Chronicle Queue Enterprise).
 - zero copy serialization and deserialization.
 - millions of writes/reads per second on commodity hardware. <br/>(~5 M messages / second for 96 byte messages on a i7-4790)

The directory structure is as follows.

```
base-directory /
   {cycle-name}.cq4       - The default format is yyyyMMdd for daily rolling.
```

The format consists of Size Prefixed Bytes which are formatted using BinaryWire or TextWire.  The ChronicleQueue.dump() method can be used to dump the raw contents as a String.

## Getting Started

### Chronicle Construction
Creating an instance of Chronicle is a little more complex than just calling a constructor.
To create an instance you have to use the ChronicleQueueBuilder.

```java
String basePath = System.getProperty("java.io.tmpdir") + "/getting-started"
ChronicleQueue queue = ChronicleQueueBuilder.single("queue-dir").build();
```

In this example we have created an IndexedChronicle which creates two RandomAccessFiles one for indexes and one for data having names relatively:

${java.io.tmpdir}/getting-started/{today}.cq4

### Writing
```java
// Obtain an ExcerptAppender
ExcerptAppender appender = queue.createAppender();

// write - {msg: TestMessage}
appender.writeDocument(w -> w.write(() -> "msg").text("TestMessage"));

// write - TestMessage
appender.writeText("TestMessage");
```

### Reading
```java
ExcerptTailer tailer = queue.createTailer();

tailer.readDocument(w -> System.out.println("msg: " + w.read(()->"msg").text()));

assertEquals("TestMessage", tailer.readText());
```

### Cleanup

Chronicle Queue stores its data off heap and it is recommended that you call close() 
once you have finished working with Chronicle-Queue to free resources, however no data will be lost of you dod not do this.

```java
queue.close();
```

### Putting it all together
```java
try (ChronicleQueue queue = ChronicleQueueBuilder.single("queue-dir").build()) {
    // Obtain an ExcerptAppender
    ExcerptAppender appender = queue.createAppender();

    // write - {msg: TestMessage}
    appender.writeDocument(w -> w.write(() -> "msg").text("TestMessage"));

    // write - TestMessage
    appender.writeText("TestMessage");

    ExcerptTailer tailer = queue.createTailer();

    tailer.readDocument(w -> System.out.println("msg: " + w.read(()->"msg").text()));

    assertEquals("TestMessage", tailer.readText());
}
```

## Replication

Chronicle Queue Enterprise supports TCP replication with optional filtering so only the required record or even fields are transmitted. This improves performances and reduce bandwitdh requirements.

![](http://chronicle.software/wp-content/uploads/2014/07/Screen-Shot-2015-01-16-at-15.06.49.png)

##  Support
* [Chronicle FAQ](https://github.com/OpenHFT/Chronicle-Queue/blob/master/docs/FAQ.adoc)
* [Chronicle support on StackOverflow](http://stackoverflow.com/tags/chronicle/info)
* [Chronicle support on Google Groups](https://groups.google.com/forum/?hl=en-GB#!forum/java-chronicle)
* [Development Tasks - JIRA] (https://higherfrequencytrading.atlassian.net/browse/CHRON)
