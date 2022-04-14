---
title: "MongoDB常用查询语句"
date: 2020-02-09T16:36:27+08:00
draft: true
---

## 增删改查部分
### MongoDB单条件查询
```
db.xxx.find({'city.value':'深圳市'})
```
文档部分json结构为：
```json
{
    ...
    "city": {
        "chinese": "城市",
        "value": "深圳市"
    }
 
}
```
使用key.key的形式，得到对应的value。
### MongoDB与查询
```
db.xxx.find({$and:[{'city.value':'深圳市'},{'status': 'finished'}]})
```

#### MongoDB的Group By和Having
```javascript
db.xxx.aggregate(
    {
        $match: {
            'city.value':'马鞍山市'
        }
    },
    {
        $group : {
            _id : "$service",
            num_tutorial : {
                $sum : 1
            }
        }
    },
    {
        $match:{
            num_tutorial:{
                $gt:1
            }
        }
    }
)
```
$group指定对哪个属性分组，用num_tutorial来判断数量是否满足某个条件。
$gt:greater than
### 删除文档
```
db.xxx.remove({'city.value':'伊犁哈萨克自治州'})
```
### 删除重复数据
删除某个属性重复的数据，只保留第一条
``` javascript
db.xxx.aggregate(
    {
        $match:{
            'city.value':‘xxx’
        }
    },
    {
        $group : {
            _id : "$service", 
            num_tutorial : {
                $sum : 1
            },
            dups:{
                $addToSet:'$_id'
            }
        }
    },
    {
        $match:{
            num_tutorial:{
                $gt:1
            }
        }
    }).forEach(doc2=>{
        doc2.dups.shift();
        db.bszn.remove({_id:{$in:doc2.dups}});
    })
```

**为选中的文档增加属性**
为“status”属性赋值为“finished”。
```
db.xxx.update({}, {$set:{'status':'finished'}},{multi: true})
```
**为选中的文档删除属性**
删除“status”属性。
```
db.xxx.update({}, {$unset:{'status':''}},{multi: true})
```
**根据ObjectId查询**
```
db.xxx.find({'_id': new ObjectId('5d7368fb5a32cfe8d8651781')})
```
**向数组中插入值或对象**
```javascript
db.xxx.update({'_id': new ObjectId('5d7368fb5a32cfe8d8651781')},
{$push:{'某个字段'：'要插入的值'}})
```
## Mongo导入导出
### Mongo导出 – mongodump
```cmd
mongodump -h 192.168.xx.xxx:xxxxx -u xxxx -p xxx -d xxxxx -c xxx -o W:\xxxxx\dump\someDirectory
```
- -h 服务器ip:端口号
- -u 用户名
- -p 密码
- -d 选择的数据库
- -c 选择的集合（如果没有-c，则将所有集合导出备份）
- -o 选择导出的目录

注意，如果出现

```
Failed: error writing data for collection `bigdata_znkf_crawl.fgcx` to disk: error reading collection: 
Failed to parse: { find: "fgcx", skip: 0, snapshot: true, 
$readPreference: { mode: "secondaryPreferred" }, 
$db: "bigdata_znkf_crawl" }. Unrecognized field 'snapshot'.
```

则在后面加上：

```
--forceTableScan
```

### Mongo导入 – mongorestore
```
mongorestore -h 192.168.xx.xxx:xxxxx -u xxx -p xxxx -d xx  -c xxx 
W:\xxxxxx.bson
```
## Mongo数组查询
### 字符串数组
来源：[https://www.jianshu.com/p/cf983a28c2da](https://www.jianshu.com/p/cf983a28c2da)

使用$all操作：
最后那个文件前面不用 横杠+字母，直接选择一个.bson文件即可导入了。
```javascript
db.fruitshop.find({"fruits":{"$all":["apple","banana"]}});
```
### 对象数组
来源：[https://www.jb51.net/article/126911.htm](https://www.jb51.net/article/126911.htm)

使用$elemMatch操作：
```
{ "qList": { $elemMatch: { "qid": 1, "reorderFlag": 0} } }
```
### Mongo树查询（存在parent_id进行连接）
```javascript
db.crawler_classify.aggregate([
    {$match: {'classify_id' : '94ebb480247d42dcb5c99a2ef19550ab'}},
    {$graphLookup : {from: 'crawler_classify', startWith: "$classify_id" , connectFromField: "classify_id", connectToField: "parent_id", as :"son"}}
    ])
```
### Mongo插入语句
```javascript
db.crawler_classify.insert({
    'xxxx': 'yyyy',
    'zzzz': 'aaa',
    ...
})
```