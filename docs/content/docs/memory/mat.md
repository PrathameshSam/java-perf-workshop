---
title: "Eclipse MAT"
weight: 13
description: Heap Analysis using Eclipse MAT
---

### Eclipse Memory Analyzer

[Eclipse Memory Analyzer](https://eclipse.org/mat/) is a fairly mature tool on analyzing heap dumps. It is a rich tool that includes the abilities to do OQL queries on the heap, general reporting on anti-patterns, and helpful common views (like the dominator tree). Eclipse Memory Analyzer tool is a separate install which can be a standalone install, or it can run as a plugin within Eclipse. First, install the tool if you haven't already via their [downloads page](https://eclipse.org/mat/downloads.php).

After you have it installed, go to __File__ and then __Open Heap Dump...__ and specify the `hprof` file that you wish to analyze.

💡 When Eclipse Memory Analyzer loads your `hprof` file, it will create many other index files to assist in optimally analyzing and navigating through your heap dump. It may make it easier to isolate your `prof` file in its own directory prior to opening it, based on the secondary files which are created. You can always just delete these files within Eclipse Memory Analyzer by opening the __Heap Dump History__ window, right-clicking the `hprof` which was previously loaded, and selecting __Delete Index Files__. Example listing of index files which get generated by suffix:

* `.domIn.index`
* `.domOut.index`
* `.index`
* `.o2ret.index`
* `.a2s.index`
* `.idx.index`
* `.inbound.index`
* `.o2c.index`
* `.o2hprof.index`
* `.outbound.index`
* `.threads`

#### Dominator Tree

A common first area to look at your heap within Eclipse Memory Analyzer is the dominator tree. From the dominator tree, you can organize by the retained heap size, and then begin drilling down the tree to see the contributors to the largest GC roots.

![MAT Dominator Tree](https://github.com/cchesser/java-perf-workshop/wiki/images/mat_dominator_tree.png)

#### Histogram

The histogram gives you a quick listing of all the top consumers by type. Typically this is going to provide context of the large consumers based on their "lower-level" types. For example, `char[]` is a common large contributor, which then will be followed by `String` which is a type that is predominately weighted by the size of the `char[]`.

![MAT Thread Overview](https://github.com/cchesser/java-perf-workshop/wiki/images/mat_histogram.png)

#### Thread Overview

The thread overview is a helpful view when you are looking at contributors based on the execution of code. For example, if you have a JVM which has thrown an `OutOfMemoryError`, you can tell if some type of request caused a massive spike which caused the exhaustion of memory, or if the requests may have not been abnormal, there just wasn't sufficient space to support the request.

![MAT Thread Overview](https://github.com/cchesser/java-perf-workshop/wiki/images/mat_thread_overview.png)

#### OQL

Another strong feature of Eclipse Memory Analyzer is the OQL studio to execute queries to do lookup on objects in your heap.

Lookup our `ConferenceSession` class type:

```sql
SELECT *
FROM INSTANCEOF cchesser.javaperf.workshop.data.ConferenceSession
```

💡 To execute the query, you click on the red exclamation mark (:exclamation:). Alternatively, you can press `CTRL+Enter` on Windows or `Command+Enter` on macOS.

![MAT OQL](https://github.com/cchesser/java-perf-workshop/wiki/images/mat_oql_studio.png)

By returning the raw type back, you can expand each one to further look at the contents of the object. If you wanted to just report the object with its `title` attribute and the object's retained sized, you could do this:

```sql
SELECT c.title.toString(), c.@retainedHeapSize 
FROM INSTANCEOF cchesser.javaperf.workshop.data.ConferenceSession c 
```

Now, you could then filter this by including a `WHERE` clause, where you can filter it by
the title or the abstract:

```sql
SELECT c.title.toString(), c.@retainedHeapSize FROM INSTANCEOF cchesser.javaperf.workshop.data.ConferenceSession c 
WHERE c.title.toString() LIKE ".*(J|j)ava.*" OR c.abstract.toString() LIKE ".*(J|j)ava.*"
```


Reference: [Eclipse Memory Analyzer OQL Syntax](http://help.eclipse.org/luna/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Freference%2Foqlsyntax.html)