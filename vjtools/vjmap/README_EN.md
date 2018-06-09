# VJMap

VJMap is jmap with **per GC generation (Eden, Survivor, OldGen)** object stats printing. It is built with advanced
 techniques to disclose situations like memory leaks and fast-growing tenured objects.

# 1. Introduction

You may print stats about objects with the stock jmap via `jmap -histo PID` but only by viewing the heap as a whole. 
However, additional information like OldGen object counting and survivor object age counting can play a vital 
role in troubleshooting. VJMap is built to make such information available.

Initially inspired by [tbjmap](https://github.com/alibaba/TBJMap), JDK8 compatibility was added as well as query on aged 
survivor objects.

**[Note]**: Does not work with G1. Use it with CMS and ParallelGC only. 

# 2. Getting Started

[donload from Maven Central](http://repo1.maven.org/maven2/com/vip/vjtools/vjmap/1.0.0/vjmap-1.0.0.zip) - 27k

**[Important]**: VJMap DOES cause stop-of-the-world of the target app. Make sure the target app is isolated from user 
access before you start using VJMap in production.

Run VJMap under **the same user who started the target process**. If access errors are still met, try again with 
root user.

VJMap may take quite some time to finish. Use `kill <PID_OF_VJMap>` to allow for a graceful exit. If `kill -9 <PID_OF_VJMap>` 
is mistakenly issued to the VJMap process, the target app will end up in blocked state, in which case you will have to 
execute `kill -18 <PID_OF_TARGET_APP>` TWICE to awaken the target app.

## 2.1 Commands

```
// Prints object stats of all the heap, ordered by their respective size in total.
./vjmap.sh -all PID > /tmp/histo.log

// Prints oldgen object stats, ordered by size in OldGen. Only CMS is supported for this option. 
./vjmap.sh -old PID > /tmp/histo-old.log


// Prints survivor objects over the age of 3.
./vjmap.sh -sur PID > /tmp/histo-sur.log


// Prints survivor objects over the age of 10, as desinated by the argument -sur:minage=10
// When the promotion threshold -XX:MaxTenuringThreshold is lifted, objects with a high age value will be bound 
for the CMS oldgen
./vjmap.sh -sur:minage=10 PID > /tmp/histo-sur.log
```

> PID is the process ID of target java application

## 2.2 Display Larger Objects, Leaving Smaller Ones Out

```
// Shows objects with sizes over 1KB over the whole heap
./vjmap.sh -all:minsize=1024 PID > /tmp/histo.log

// shows objects with sizes over 1KB in OldGen specifically 
./vjmap.sh -old:minsize=1024 PID > /tmp/histo-old.log

// shows objects with sizes over 1KB in survivor space 
./vjmap.sh -sur:minsize=1024 PID > /tmp/histo-sur.log
```

## 2.3 Order by Classname and Filter by Size for Periodic Comparisons

```
./vjmap.sh -all:minsize=1024,byname PID > /tmp/histo.log

./vjmap.sh -old:minsize=1024,byname PID > /tmp/histo-old.log

./vjmap.sh -sur:minsize=1024,byname PID > /tmp/histo-sur.log
```

# 3.Outputs

## 3.1 Count Survivor Objects over the Age of 3.

```
Survivor Object Histogram:

 #num  #count     #bytes #Class description
-----------------------------------------------------------------------------------
   1:      37         1k io.netty.buffer.PoolThreadCache$MemoryRegionCache$Entry
   2:       2         64 java.util.concurrent.locks.AbstractQueuedSynchronizer$Node
Total: 39/    1k over age 2

Heap traversal took 1.3 seconds.
```

# 4. Enhancements over TBJMap
* Added JDK8 Support.
* Added Display: survivor objects over the specified age.
* Performance Boost: by accessing Survivor and OldGen directly instead of by accessing the whole heap with Heap Visitor callbacks.
* New config Arg: order objects by size and leave out small ones.
* New Config Arg: order objects by name for periodic comparison.
* Reading Friendliness: output by the unit of (k, m, g) and fix alignment, order objects in OldGen by size in OldGen view by default. 