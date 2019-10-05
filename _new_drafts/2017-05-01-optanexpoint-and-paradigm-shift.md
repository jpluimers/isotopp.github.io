---
layout: post
status: publish
published: true
title: Optane/Xpoint and paradigm shift
author: isotopp
author_login: kris
author_email: kristian.koehntopp@gmail.com
wordpress_id: 1632
wordpress_url: http://blog.koehntopp.info/?p=1632
date: '2017-05-01 08:33:10 +0200'
date_gmt: '2017-05-01 07:33:10 +0200'
categories:
- MySQL
tags: []
---
<p>So [Optane is here](https://www.servethehome.com/intel-optane-hands-on-real-world-benchmark-and-test-results/). It's a&nbsp;bit-addressable non-volatile storage with a higher density than DRAM. It's [not as good as initially promised](https://semiaccurate.com/2016/09/12/intels-xpoint-pretty-much-broken/), yet, but it's a first iteration. It is basically very slow RAM (or very fast flash), which is bit-adressable. So you are not, like with flash, erasing 64 KB sized pages, but you are doing things to individual bits and bytes. It's also faster than flash (but slower than DRAM), about 10x faster than old Commodore 64 memory. And it's persistent, so if you power off your machine, contents are not gone. And it is very dense, denser even than the memory you currently use, because no transistors, so less space necessary per bit. This is going to change a lot of things, but not right now. We need to rethink our approach to persistence.<!--more--> Currently, all persistent media we have are&nbsp;structured in blocks - 512&nbsp;Byte or 4096 byte disk blocks, 64 KB flash pages or similar. Also, currently all persistent media we have are many orders of magnitude slower than DRAM. Clock speeds are being measured in fractions of nanoseconds (1 GHz is 1 clock cycle per ns), memory&nbsp;accesses are in two digit ns, and disk accesses with head movements are in milliseconds. Flash is inbetween, in microseconds. So a disk access is reading many bytes at once (often, internally, a full track of many blocks), but it is multiple&nbsp;million clock cycles long. Even a single flash access is taking multiple thousands of clock cycles. What we currently have are persistence systems such as databases (and filesystems) that create in-memory representations of data which are very different from on-disk representations, because there are many clock cycles to build these things and because the requirements of data being stored are very different from data being used, and it is a worthwhile optimization to convert between these two. There are also many layers of caches to compensate for the enormous performance gap between these two representation and media - database layer cache, filesystem layer cache, devicemapper translation logic, device driver request rewrite and reordering logic and finally on-disk hardware caches (where the disk hardware itself is just another embedded ARM based computer that specializes in running the actual platters/flash hardware). Most of that is becoming obsolete or even a liability, if you have persistent bit-adressable memory that is within 100x DRAM performance ('very slow persistent RAM'). You can keep the data in in-memory representation, getting rid of a lot of marshalling and serialization. You can throw away a lot of&nbsp;indirection layers and work with regular pointers or something close to pointer structures.&nbsp;You get rid of most of these caches. And you get rid of one to two entire stacks associated with munging things around these caches. It also means&nbsp;that what currently is a transaction now becomes something that involves strategic, ordered, lockless pointer twiddling of applications doing things to these in-memory structures. Much more of the database engine suddenly is within the realm of the application. And it finally means that SQL and parsing and compiling SQL takes maybe so much time relative to the actual processing of the data that it may not be a good deal any more. And finally, network communication latency is already a problem if the storage at the other end is flash - same order of magnitude for data transfer and data access here. With Optane, it may be that the actual cost is in the network communication and not so much in the data access. We might need something faster and with less overhead than TCP/IP for this. There are many changes ahead, and computer science is ill prepared to handle them. And that is already true at the performance levels of this first iteration. We might see a lot more monoliths instead of microservices: Large boxes with many cores running a workload that is accessing one shared large Xpoint store locally might have a considerable performance benefit over "I serialize everything all the time and competetively accumulate network latencies like there is no tomorrow" Microservices. Yes, modern mainframes, maybe as high level application aware storage subsystems that are being accessed remotely. We might see a second protocol besides TCP/IP for storage access over the network. We will see something that is no longer an SQL database, for sure. We will see the end of Hadoop, because Hadoop is a thing that is built around long linear reads of data stored on rotating rust, and is simply going away. It will take another two or three tech cycles, but it will happen.</p>