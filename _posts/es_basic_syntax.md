---
title: es基础
date: 2020-11-14 19:11:48
tags: 
---
创建Index并且插入添加document
<!-- more -->
```
PUT /customer/_doc/1
{"name": "John Doe"}
```
检索document id
```
GET /customer/_doc/1
```

查询
访问`_search`
eg: 检索`bank`Index中的所有document，并且按照`account_number`排序
```
GET /bank/_search
{
	"query": {"match_all":{}},
	"sort":[
		{"account_number":"asc"}
	]
}
```
默认情况下以上查询返回前10条数据。
返回结果字段说明
took：es执行查询耗时，单位毫秒
time_out：是否超时
_shards：多少分片被搜索
hits.total.value：匹配的document数量

分页使用`from`，`size`
```
GET /bank/_search
{
	"query": {"match_all":{}},
	"sort":[
		{"account_number":"asc"}
	],
	"from":10,
	"size":10
}
```
查询某个字段中包含某个词，用match
下面语句会查询address包含mill或者lane的记录
```
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```
如果要查询的是短语，而不是分开的单词，则使用match_phrase

更复杂的查询使用bool
bool查询中must表示必须匹配，must_not表示必须不匹配
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

aggs聚合查询
	