---
title: JDK性能分析工具
date: 2020-09-02 20:16:51
tags:
---
JDK性能分析工具使用命令记录，备忘。
<!-- more -->
# jstat
jstat可以查看堆内各个部分的使用量，以及加载类的数量
jstat -options可以列出当前JVM版本支持的选项，常见的有：
* -class（类加载器）
* -compiler
* -gc（GC堆状态）
* -gccapacity（各区大小）
* -gccause（最近一次GC统计和原因）
* -gcmetacapacity（方法去大小）
* -gcnew（新区统计）
* -gcnewcapacity（新区大小）
* -gcold（老区统计）
* -gcoldcapacity（老区大小）
* -gcutil（GC统计汇总）
* -printcompilation（HotSpot编译统计）

## jstat -class <pid>
显示加载class的数量，以及所占空间等信息。
```
显示列名	具体描述
Loaded		装载的类的数量
Bytes		装载类所占用的字节数
Unloaded	卸载的类的数量
Bytes		卸载类的字节数
Time		装载和卸载类所花费的时间
```
```bash
jstat -class 17484

Loaded  Bytes  Unloaded  Bytes     Time
   949  1799.7        0     0.0       0.75
```
## jstat -gc <pid>
显示gc的信息，查看gc的次数及时间
```
显示列名	具体描述
S0C			年轻代中第一个survivor（幸存区）的容量 (字节)
S1C			年轻代中第二个survivor（幸存区）的容量 (字节)
S0U			年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U			年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
EC			年轻代中Eden（伊甸园）的容量 (字节)
EU			年轻代中Eden（伊甸园）目前已使用空间 (字节)
OC			Old代的容量 (字节)
OU			Old代目前已使用空间 (字节)
MC			MetaSpace(持久代)的容量 (字节)
MU			MetaSpace(持久代)目前已使用空间 (字节)
YGC			从应用程序启动到采样时年轻代中gc次数
YGCT		从应用程序启动到采样时年轻代中gc所用时间(s)
FGC			从应用程序启动到采样时old代(全gc)gc次数
FGCT		从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT			从应用程序启动到采样时gc用的总时间(s)
```
```bash
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
5120.0 5120.0  0.0    0.0   32768.0  13136.0   86016.0      0.0     4480.0 775.8  384.0   76.4       0    0.000   0      0.000    0.000
```
## jstat -gccapacity <pid>
显示VM内存中三代（young，old，metaspace）对象的使用和占用大小
```
显示列名	具体描述
NGCMN    	年轻代(young)中初始化(最小)的大小(字节)
NGCMX     	年轻代(young)的最大容量 (字节)
NGC     	年轻代(young)中当前的容量 (字节)
S0C   		年轻代中第一个survivor（幸存区）的容量 (字节)
S1C       	年轻代中第二个survivor（幸存区）的容量 (字节)
EC      	年轻代中Eden（伊甸园）的容量 (字节)
OGCMN      	old代中初始化(最小)的大小 (字节)
OGCMX       old代的最大容量(字节)
OGC			old代当前新生成的容量 (字节)
OC      	Old代的容量 (字节)
PGCMN    	perm代中初始化(最小)的大小 (字节)
PGCMX     	perm代的最大容量 (字节)  
PGC       	perm代当前新生成的容量 (字节)
PC     		Perm(持久代)的容量 (字节)
YGC    		从应用程序启动到采样时年轻代中gc次数
FGC			从应用程序启动到采样时old代(全gc)gc次数
```
## jstat -gcutil <pid>
统计gc信息
```
S0     	年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
S1     	年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
E      	年轻代中Eden（伊甸园）已使用的占当前容量百分比
O      	old代已使用的占当前容量百分比
M     	方法区已使用的占当前容量百分比
YGC     从应用程序启动到采样时年轻代中gc次数
YGCT    从应用程序启动到采样时年轻代中gc所用时间(s)
FGC    	从应用程序启动到采样时old代(全gc)gc次数
FGCT    从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT		从应用程序启动到采样时gc用的总时间(s)
```
```bash
jstat -gcutil 17484

  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  40.09   0.00  17.32  19.90      0    0.000     0    0.000    0.000
```
## jstat -gcnew <pid>
年轻代对象的信息
```
S0C    	年轻代中第一个survivor（幸存区）的容量 (字节)
S1C    	年轻代中第二个survivor（幸存区）的容量 (字节)
S0U    	年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U   	年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
TT		持有次数限制
MTT  	最大持有次数限制
EC      年轻代中Eden（伊甸园）的容量 (字节)
EU     	年轻代中Eden（伊甸园）目前已使用空间 (字节)
YGC    	从应用程序启动到采样时年轻代中gc次数
YGCT	从应用程序启动到采样时年轻代中gc所用时间(s)
```
```bash
jstat -gcnew 17484

 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
5120.0 5120.0    0.0    0.0 15  15    0.0  32768.0  13136.0      0    0.000
```
## jstat -gcold <pid>
老年代对象的信息
```
MC       	方法区(持久代)的容量 (字节)
MU        	方法区(持久代)目前已使用空间 (字节)
OC          Old代的容量 (字节)
OU       	Old代目前已使用空间 (字节)
YGC    		从应用程序启动到采样时年轻代中gc次数
FGC    		从应用程序启动到采样时old代(全gc)gc次数
FGCT     	从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT			从应用程序启动到采样时gc用的总时间(s)
```
```bash
jstat -gcold 17484

   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
  4480.0    775.8    384.0     76.4     86016.0         0.0      0     0    0.000    0.000
```

# jmap
jmap可以输出所有内存中的对象
基本参数
-dump:[live,]format=b,file=<filename> 使用hprof二进制形式，输出jvm的heap内容到文件中，假如指定live选项，那么只输出活的对象到文件。
-heap 打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况。
```bash
jmap -heap 17484

Attaching to process ID 17484, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 8 thread(s)  #GC名称

Heap Configuration: #堆配置情况
   MinHeapFreeRatio         = 0                         #最小堆使用比例
   MaxHeapFreeRatio         = 100                       #最大堆可用比例
   MaxHeapSize              = 2111832064 (2014.0MB)     #最大堆空间大小
   NewSize                  = 44040192 (42.0MB)         #新生代分配大小
   MaxNewSize               = 703594496 (671.0MB)       #新生代最大分配大小
   OldSize                  = 88080384 (84.0MB)         #老年代大小
   NewRatio                 = 2                         #新生代比例
   SurvivorRatio            = 8                         #新生代与survivor比例
   MetaspaceSize            = 21807104 (20.796875MB)    #方法区大小
   CompressedClassSpaceSize = 1073741824 (1024.0MB)     
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage: #堆使用情况
PS Young Generation #新生代
Eden Space: #Eden区
   capacity = 33554432 (32.0MB) #容量
   used     = 13451272 (12.828132629394531MB) #使用大小
   free     = 20103160 (19.17186737060547MB) #剩余
   40.08791446685791% used #使用比例
From Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
To Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
PS Old Generation
   capacity = 88080384 (84.0MB)
   used     = 0 (0.0MB)
   free     = 88080384 (84.0MB)
   0.0% used

3876 interned Strings occupying 318128 bytes.
```
-histio[:live] 打印每个class的实例数目，内存占用，类全名信息。VM内部类名字开头会加上*
