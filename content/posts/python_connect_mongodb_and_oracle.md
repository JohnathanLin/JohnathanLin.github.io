---
title: "Python连接MongoDB和Oracle实战"
date: 2020-02-09T17:06:29+08:00
draft: false
categories: ["技术文章"]
---

## Python连接MongoDB
### 安装

首先要安装pymongo，用pip装一下就好了。

#### 工具类python文件

以下直接给出我写的mongodb操作类
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/12/23 14:02
# @Author  : Johnathan Lin 林佳庆
"""
数据导入Mongo模块
"""

import os

import pymongo

from utils.configReader import read_mongo_config

# 设置编码
os.environ['NLS_LANG'] = 'SIMPLIFIED CHINESE_CHINA.UTF8'


def get_mongodb_collection(collection):
    """
    根据配置文件和数据库和表得到表（collection）
    :param database: 数据库
    :param collection: 表
    :return:  表的对象
    """
    mongo_config = read_mongo_config()
    client = pymongo.MongoClient(mongo_config['client'])
    db = client[mongo_config['database']]
    if mongo_config['auth'] == 'True' or mongo_config['auth'] == 'true':
        db.authenticate(mongo_config['username'], mongo_config['password'])
    col = db[collection]
    return col


def insert_data(collection, document):
    """
    将数据插入MongoDB中
    :param collection:数据将插入的集合名
    :param doc:数据文档
    :return:无
    """
    col = get_mongodb_collection(collection)
    col.insert_one(document)


def insert_data_list(collection, document_list):
    """
    批量插入数据到MongoDB
    :param collection: 数据将插入的集合名
    :param document_list: 数据文档列表
    :return:
    """
    col = get_mongodb_collection(collection)
    col.insert_many(document_list)


def get_children_classify_id(parent_id):
    """
    获取所有子孙节点
    :param parent_id: 父节点ID
    :return: 所有子孙节点
    """
    col = get_mongodb_collection('crawler_classify')
    res = col.aggregate([
        {'$match': {'classify_id': parent_id}},
        {'$graphLookup': {'from': 'crawler_classify', 'startWith': '$classify_id', 'connectFromField': 'classify_id',
                          'connectToField': 'parent_id', 'as': 'son'}}
    ])
    res_list = []
    for doc in res:
        res_list.append(doc)
    if len(res_list) == 0 or len(res_list[0]['son']) == 0:
        return []
    else:
        return [classify['classify_id'] for classify in res_list[0]['son']]


def get_document_by_condition(collection, key, value):
    """
    根据集合和单一条件查找匹配的文档
    :param collection:  集合名
    :param key:  键
    :param value: 值
    :return: 匹配的文档列表
    """
    col = get_mongodb_collection(collection)
    res = col.find({key: value})
    res_list = []
    for doc in res:
        res_list.append(doc)
    return res_list


def update_document(collection, condition_key, condition_value, updated_key, new_value):
    """
    更新文档
    :param collection:
    :param condition_key:
    :param condition_value:
    :param updated_key:
    :param new_value:
    :return:
    """
    col = get_mongodb_collection(collection)
    col.update({condition_key: condition_value}, {'$set': {updated_key: new_value}})


def get_document_by_condition_list(collection, condition_list):
    """
    多条件同时成立查询匹配的文档
    :param collection: 集合名
    :param condition_list:  条件列表（列表中的元素为{key:value}形式）
    :return: 匹配的文档
    """
    col = get_mongodb_collection(collection)
    res = col.find({'$and': condition_list})
    res_list = []
    for doc in res:
        res_list.append(doc)
    return res_list


def remove_document(collection, condition_list):
    """
    删除文档
    :param collection: 删除文档的集合
    :param condition_list:
    :return:
    """
    col = get_mongodb_collection(collection)
    col.remove({'$and': condition_list})


def update_document_condition_list_and_new_value_obj(collection, condition_list, new_value_obj):
    """
    更新update文档，通过多个用“&”连接的条件和新对象更新
    :param collection: 集合
    :param condition_list: 与条件的列表
    :param new_value_obj: 要更新的新值对象
    :return:
    """
    col = get_mongodb_collection(collection)
    col.update({'$and': condition_list}, {'$set': new_value_obj})
```

#### 配置文件
配置文件长这样：
```config
[CONFIG]
client=mongodb://localhost:27017/
auth=False
username=xxx
password=xxx
database=yyy
```

#### 获取集合
我们以 get_mongodb_collection(collection)函数为例：

```python
def get_mongodb_collection(collection):
    """
    根据配置文件和数据库和表得到表（collection）
    :param database: 数据库
    :param collection: 表
    :return:  表的对象
    """
    mongo_config = read_mongo_config()
    client = pymongo.MongoClient(mongo_config['client'])
    db = client[mongo_config['database']]
    if mongo_config['auth'] == 'True' or mongo_config['auth'] == 'true':
        db.authenticate(mongo_config['username'], mongo_config['password'])
    col = db[collection]
    return col
```

在使用Mongo的时候，我们需要先获取其Mongo服务器地址，也就是上文配置文件的client字段内的内容，通过pymongo.MongoClient()方法获取其服务器对象

然后，使用client[‘数据库名’]选择一个数据库，但选中的数据库可能需要验证用户名密码，所以如果需要验证，则使用db.authenticate(用户名，密码)验证；

最后，用db[‘集合名’]取出对应的集合即可使用了。
#### 使用
可以发现，在python里使用mongo，用法和直接敲mongoDB脚本差不多，但是也存在一些不同，比如说‘$and’必须是字符串。

## Python连接Oracle
pip装 cx-Oracle（这里注意，会有很多双胞胎几胞胎兄弟，认准这个带横杠的，后面O大写的）。

这是我之前写的python调用oracle。
```python
import cx_Oracle

from utils.dataOperator import insert_data


def start():
    conn = cx_Oracle.connect('oracle用户名/oracle密码@IP地址:端口号/oracle用户')
    parent_id = '6c9da9e2b17b4e1da967f2fdfa05d89a'
    sql = 'select * from 一个表的名字 start with CLASSIFY_ID = \'' + parent_id + '\'connect by prior classify_id = parent_id order by seq'
    cur = conn.cursor()
    cur.execute(sql)
    result_list = cur.fetchall()
    for result in result_list:
        res_obj = {}
        res_obj['classify_id'] = str(result[0])
        # 一顿操作。。。。
        # 在此不表
        insert_data('数据库名', '集合名', res_obj)
```
使用cx_Oracle.connect（注意这里cx_Oracle是下划线了）调用方法，参数是oracle连接字符串

之后创建一个游标cursor，用它执行sql语句，然后用fetchAll()函数把结果拿回来就可以用了。

但具体怎么用呢？

假如CLASSIFY_ID是执行这行sql返回结果中第一列的列名，那么result[0]就是该行数据在该列的值。

所以需要多加注意。