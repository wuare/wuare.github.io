---
title: Redis集群
date: 2019-03-30 16:50:20
tags: Redis
---
阅读Redis集群官方文档[https://redis.io/topics/cluster-tutorial](https://redis.io/topics/cluster-tutorial)，记下笔记。
<!-- more -->
Redis集群要求3.0+版本。
# Redis集群TCP端口
每个Redis集群节点要求打开两个TCP连接。正常的TCP端口用来服务客户端，比如6379，另一个端口一般是正常端口加10000，如16379.第二个端口用于集群总线（一种用二进制协议在节点和节点之间通信的通道）。节点使用集群总线进行故障检测、配置更新、故障转移授权等。
# Redis集群数据分片
Redis集群没有使用一致性Hash，与分片不同的地方在于每个key是哈希槽（hash slot）的一部分。
在一个Redis集群中有16384个哈希槽。计算某个key的哈希槽的简单方法是使用CRC16计算一个数除16384取余。
在Redis集群中每个节点负责一部分哈希槽。例如：一个集群有三个节点，那么哈希槽分配如下：
* 节点A包含从0到5500的哈希槽
* 节点B包含从5501到11000的哈希槽
* 节点C包含从11001到16383的哈希槽

这种做法可以很容易增加或删除一个节点。比如想增加一个节点D，只需要从A，B，C节点转移一部分哈希槽到D。同样的如果要移除节点A，可以将A节点的哈希槽移到A，B，C。当A节点的哈希槽移完以后，摘除A节点即可。
由于把哈希槽从一个节点移到另一个节点不需要停止其他操作，所以添加或删除节点不需要停止集群。
# Redis集群主从模式
为了保证当一个master节点故障时仍然可用，集群使用主从模式，即每个哈希槽有1个或N个备份（N-1个slave节点）。
例如：一个集群由ABC3个节点组成，这3个节点为master节点，然后为这个3个节点分别设置一个slave节点A1，B1，C1。当B节点故障时，集群把B1基点提升为master节点，整个集群可以正常操作。但是当B节点和B1节点都故障，则集群不能正常操作。
# Redis集群一致性
Redis集群不能保证强一致性。这是由于主从模式下备份数据是异步的。具体情形如下：
* 客户端写入数据到master B
* master B节点向客户端回复OK
* master B开始写入slave节点B1，B2，B3

所以在客户端向B写入数据之后，B向slave节点写入数据之前，B节点发生了故障，则数据将会丢失。
如果绝对需要同步，集群也支持同步写入，该操作通过`wait`命令实现。即使同步写入，集群也不能实现强一致性。
# Redis集群配置参数
* **cluster-enabled**&lt;yes/no&gt;：如果启用集群，则必须设置为yes，否则以单独节点运行。
* **cluster-config-file**&lt;filename&gt;：该配置文件不允许用户编辑，Redis集群每次更改会自动保存集群状态，以便在启动时重新读取。文件列出了集群中其他节点、状态等。
* **cluster-node-timeout**&lt;millisecond&gt;：集群节点不可用最大时间值，如果超过该时间，则认为该节点故障。
* **cluster-slave-validity-factor**&lt;factor&gt;：如果设置为0，则slave节点不管与主节点连接和断开的时间长短始终尝试故障转移，如果该值为正值，节点超时时间乘以该系数作为断开连接最大时间。如果该节点是slave节点，如果主节点断开连接时间超过指定的值，则从节点不再故障转移。例如：节点超时时间为5秒，该值为10，如果和主节点断开时间超过50秒，则从节点不再尝试故障转移。
* **cluster-migration-barrier**&lt;count&gt;：主节点保持连接从节点的最小数量。
* **cluster-require-full-coverage **&lt;yes/no&gt;：默认情况为yes，当一定百分比的key没有被任一节点覆盖，则集群不再接受写入。

# 集群的创建和使用
一个最小的集群至少3个主节点。推荐启动3个主节点3个从节点6个实例。
具体操作步骤：新建一个文件夹，在文件夹中创建以redis端口为名称的文件夹。
```bash
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
```
修改配置
```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```
启动redis-server，将会看到日志，每个redis实例被分配了一个ID，该ID唯一标识一个节点。
## 创建集群
如果使用Redis 5，使用`redis-cli`中内置的命令很容易创建、检查或者对一个集群重新分片。
对于Redis 3或者4，有个相似的工作`redis-trib.rb`，需要安装redis gem去运行`reids-trib.rb`	，以下使用Redis 5演示执行命令。
```bash
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```
如果创建成功，则显示`[OK] All 16384 slots covered`。
## 集群重新分片
使用命令
```bash
redis-cli --cluster reshard 127.0.0.1:7000
```
集群将询问迁移多少个slot，输入数量，然后继续询问实例ID，可以使用命令查询ID。
```bash
redis-cli -p 7000 cluster nodes | grep myself
```
然后继续询问哈希槽来源的节点，输入`all`表示从其他所有主节点分配哈希槽。
使用命令`redis-cli --cluster check 127.0.0.1:7000`检查7000状态。
## 新增节点
使用命令`redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000`新增节点。
使用命令`redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000 --cluster-slave --cluster-master-id xxx`新增从节点。
## 移除节点
使用命令以下移除节点
```bash
redis-cli --cluster del-node 127.0.0.1:7000 `<node-id>`
```