---
layout: post
title: "Linux的Cached Memory"
description: ""
category: [基础架构]
tags: [缓存]
---
{% include JB/setup %}

## 文件Cache

-------------------------------------------

在 Linux 操作系统中，当应用程序需要读取文件中的数据时，操作系统先分配一些内存，将数据从存储设备读入到这些内存中，然后再将数据分发给应用程序；当需要往文件中写数据时，操作系统先分配内存接收用户数据，然后再将数据从内存写到磁盘上。

文件 Cache 管理指的就是对这些由操作系统分配，并用来存储文件数据的内存的管理。 Cache 管理的优劣通过两个指标衡量：一是 Cache 命中率，Cache 命中时数据可以直接从内存中获取，不再需要访问低速外设，因而可以显著提高性能；二是有效 Cache 的比率，有效 Cache 是指真正会被访问到的 Cache 项，如果有效 Cache 的比率偏低，则相当部分磁盘带宽会被浪费到读取无用 Cache 上，而且无用 Cache 会间接导致系统内存紧张，最后可能会严重影响性能。

文件 Cache 是文件数据在内存中的副本，因此文件 Cache 管理与内存管理系统和文件系统都相关：一方面文件 Cache 作为物理内存的一部分，需要参与物理内存的分配回收过程，另一方面文件 Cache 中的数据来源于存储设备上的文件，需要通过文件系统与存储设备进行读写交互。从操作系统的角度考虑，文件 Cache 可以看做是内存管理系统与文件系统之间的联系纽带。因此，文件 Cache 管理是操作系统的一个重要组成部分，它的性能直接影响着文件系统和内存管理系统的性能。

图 1 描述了 Linux 操作系统中文件 Cache 管理与内存管理以及文件系统的关系示意图。从图中可以看到，在 Linux 中，具体文件系统，如 ext2/ext3、jfs、ntfs 等，负责在文件 Cache和存储设备之间交换数据，位于具体文件系统之上的虚拟文件系统VFS负责在应用程序和文件 Cache 之间通过 read/write 等接口交换数据，而内存管理系统负责文件 Cache 的分配和回收，同时虚拟内存管理系统(VMM)则允许应用程序和文件 Cache 之间通过 memory map的方式交换数据。可见，在 Linux 系统中，文件 Cache 是内存管理系统、文件系统以及应用程序之间的一个联系枢纽。

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/cache.jpg)

## 文件Cache数据结构

-------------------------------------------

在 Linux 的实现中，文件 Cache 分为两个层面，一是 Page Cache，另一个 Buffer Cache，每一个 Page Cache 包含若干 Buffer Cache。内存管理系统和 VFS 只与 Page Cache 交互，内存管理系统负责维护每项 Page Cache 的分配和回收，同时在使用 memory map 方式访问时负责建立映射；VFS 负责 Page Cache 与用户空间的数据交换。而具体文件系统则一般只与 Buffer Cache 交互，它们负责在外围存储设备和 Buffer Cache 之间交换数据。Page Cache、Buffer Cache、文件以及磁盘之间的关系如图 2 所示，Page 结构和 buffer_head 数据结构的关系如图 3 所示。在上述两个图中，假定了 Page 的大小是 4K，磁盘块的大小是 1K。本文所讲述的，主要是指对 Page Cache 的管理。

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/cache1.jpg)

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/cache2.jpg)

在 Linux 内核中，文件的每个数据块最多只能对应一个 Page Cache 项，它通过两个数据结构来管理这些 Cache 项，一个是 radix tree，另一个是双向链表。Radix tree 是一种搜索树，Linux 内核利用这个数据结构来通过文件内偏移快速定位 Cache 项，图 4 是 radix tree的一个示意图，该 radix tree 的分叉为4(22)，树高为4，用来快速定位8位文件内偏移。Linux(2.6.7) 内核中的分叉为 64(26)，树高为 6(64位系统)或者 11(32位系统)，用来快速定位 32 位或者 64 位偏移，radix tree 中的每一个叶子节点指向文件内相应偏移所对应的Cache项。

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/cache3.gif)

另一个数据结构是双向链表，Linux内核为每一片物理内存区域(zone)维护active_list和inactive_list两个双向链表，这两个list主要用来实现物理内存的回收。这两个链表上除了文件Cache之外，还包括其它匿名(Anonymous)内存，如进程堆栈等。

## 文件Cache预读和替换

--------------------------------------------

Linux内核中文件预读算法的具体过程是这样的：对于每个文件的第一个读请求，系统读入所请求的页面并读入紧随其后的少数几个页面(不少于一个页面，通常是三个页面)，这时的预读称为`同步预读`。对于第二次读请求，如果所读页面不在Cache中，即不在前次预读的group中，则表明文件访问不是顺序访问，系统继续采用同步预读；如果所读页面在Cache中，则表明前次预读命中，操作系统把预读group扩大一倍，并让底层文件系统读入group中剩下尚不在Cache中的文件数据块，这时的预读称为`异步预读`。无论第二次读请求是否命中，系统都要更新当前预读group的大小。

此外，系统中定义了一个window，它包括前一次预读的group和本次预读的group。任何接下来的读请求都会处于两种情况之一：第一种情况是所请求的页面处于预读window中，这时继续进行异步预读并更新相应的window和group；第二种情况是所请求的页面处于预读window之外，这时系统就要进行同步预读并重置相应的window和group。图5是Linux内核预读机制的一个示意图，其中a是某次读操作之前的情况，b是读操作所请求页面不在window中的情况，而c是读操作所请求页面在window中的情况。

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/cache4.gif)

Linux内核中文件Cache替换的具体过程是这样的：刚刚分配的Cache项链入到inactive_list头部，并将其状态设置为active，当内存不够需要回收Cache时，系统首先从尾部开始反向扫描active_list并将状态不是referenced的项链入到inactive_list的头部，然后系统反向扫描inactive_list，如果所扫描的项的处于合适的状态就回收该项，直到回收了足够数目的Cache项。

## Linux Cache查看

--------------------------------------------

使用命令 cat /proc/meminfo

	MemTotal:       65951816 kB
	MemFree:         5619132 kB
	Buffers:          973232 kB
	Cached:         42700380 kB
	SwapCached:            0 kB
	Active:         40573736 kB
	Inactive:       15884684 kB
	Active(anon):   12211624 kB
	Inactive(anon):  1134312 kB
	Active(file):   28362112 kB
	Inactive(file): 14750372 kB
	Unevictable:        1892 kB
	Mlocked:               0 kB
	SwapTotal:             0 kB
	SwapFree:              0 kB
	Dirty:             37940 kB
	Readahead	        0 kB
	Writeback:             0 kB
	AnonPages:      12786760 kB
	Mapped:           317028 kB
	Shmem:            559224 kB
	Slab:            2973088 kB
	SReclaimable:    2860064 kB
	SUnreclaim:       113024 kB
	KernelStack:       14592 kB
	PageTables:       164732 kB
	NFS_Unstable:          0 kB
	Bounce:                0 kB
	WritebackTmp:          0 kB
	CommitLimit:    32975908 kB
	Committed_AS:   36035680 kB
	VmallocTotal:   34359738367 kB
	VmallocUsed:      384232 kB
	VmallocChunk:   34323892400 kB
	HugePages_Total:       0
	HugePages_Free:        0
	HugePages_Rsvd:        0
	HugePages_Surp:        0
	Hugepagesize:       2048 kB
	DirectMap4k:        6152 kB
	DirectMap2M:    67082240 kB
	
通过free -g(-m)命令观察系统的内存使用情况：

    	        total       used       free     shared    buffers     cached
	Mem:            62         57          5          0          0         40
	-/+ buffers/cache:         15         47
	Swap:            0          0          0
	
其中第一行用全局角度描述系统使用的内存状况：

	total——总物理内存 used——已使用内存，一般情况这个值会比较大，total=used+free
	used 已经使用的内存数  used=buffers+cached (maybe add shared also) 
	free——完全未被使用的内存 shared——应用程序共享内存 
	buffers——缓存，主要用于目录方面,inode值等（ls大目录可看到这个值增加） 
	cached——缓存，用于已打开的文件 

第二行描述应用程序的内存使用： 

	前个值表示-buffers/cache——应用程序使用的内存大小，used减去缓存值 
	后个值表示+buffers/cache——所有可供应用程序使用的内存大小，free加上缓存值 
	  
	-buffers/cache=used-buffers-cached 
	+buffers/cache=free+buffers+cached 
	
第三行表示：

	swap的使用：used——已使用 free——未使用
	
## 手动释放Cache（不建议）

------------------------------------

/proc 是一个虚拟文件系统,我们可以通过对它的读写操作做为与kernel实体间进行通信的一种手段.也就是说可以通过修改/proc中的文件,来对当前 kernel的行为做出调整.那么我们可以通过调整/proc/sys/vm/drop_caches来释放内存.操作如下:

	首先查看/proc/sys/vm/drop_caches的值,默认为0：
	# cat /proc/sys/vm/drop_caches
	#	0

	然后同步一下数据：	
	#sync
	(执行sync命令(描述:sync命令运行sync子例程。如果必须停止系统，则运行sync命令以确保文件系统的完整性。
	sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的i-node、已延迟的块I/O和读写映射文件)
	
	
	最后修改/proc/sys/vm/drop_caches：
	
	To free  pagecache,  
	use  echo 1 > /proc/sys/vm/drop_caches; 
	to free dentries and inodes, 
	use echo 2 > /proc/sys/vm/drop_caches;
	to free pagecache, dentries  and  inodes,  
	use echo 3 > /proc/sys/vm/drop_caches.

