---
title: "Springboot操作MongoDB，包括增改查及复杂操作"
date: 2020-05-19T12:41:37+08:00
draft: false
categories: ["技术文章"]
---

## 单条件查询
使用BasicDBObject配置查询条件
```java
    List<AbstractMongoEntity> list = Lists.newArrayList();
    	// 配置查询条件
        BasicDBObject cond1 = new BasicDBObject();
        cond1.append("_id", new ObjectId("5de39f20684014f1d8b8fa37"));
        FindIterable<Document> findIterable = 
		// 执行查询
		mongoTemplate.getCollection("crawler_cjwt").find(cond1);
		// 装配查询结果
        MongoCursor<Document> cursor = findIterable.iterator();
        Document document = null;
        CjwtMongoEntity question = null;
        while (cursor.hasNext()) {
            document = cursor.next();
            // 使用MongoConverter可以将结果对象映射到Java Bean
            question = mongoConverter.read(CjwtMongoEntity.class, document);
            list.add(question);
        }
        System.out.println(question);
        cursor.close();
```
返回的是一个指针，所以我们需要通过该指针遍历结果，并装进list中返回使用。
对应的mongo脚本：

```bash
db.crawler_cjwt.find({'_id':new ObjectId("5de39f20684014f1d8b8fa37")})
```

## 查询该集合所有结果
```java
        List<FgcxMongoEntity> list = Lists.newArrayList();
		// find函数没有传参，即查询所有
        FindIterable<Document> findIterable = crawlMongoTemplate.getCollection("fgcx").find();

        MongoCursor<Document> cursor = findIterable.iterator();
        FgcxMongoEntity question = null;
        while (cursor.hasNext()) {
            question = mongoConverter.read(FgcxMongoEntity.class, cursor.next());
            list.add(question);
        }
        cursor.close();
```

## 多条件查询

```java
  String cityClassifyId = "b2c7147e804b48f8af562fbe1f1c32f6";
		// 上面这部分是获取省，市的名字
        IndustryRepoClassify cityClassify = industryRepoClassifyMapper.selectByPrimaryKey(cityClassifyId);
        IndustryRepoClassify provinceClassify = industryRepoClassifyMapper.selectByPrimaryKey(cityClassify.getParentId());
        IndustryRepoClassify bsznClassify = industryRepoClassifyMapper.selectByPrimaryKey(provinceClassify.getParentId());
        // 获取集合对象
        MongoCollection<Document> collection = mongoTemplate.getCollection("bszn");
     	// 配置查询BasicDBObject
        BasicDBObject filterCondition = new BasicDBObject();
        filterCondition.append("province.value", provinceClassify.getClassifyName());
        filterCondition.append("city.value", cityClassify.getClassifyName());
		// 执行查询
        FindIterable<Document> findIterable = collection.find(filterCondition);
        List<Document> res = (List<Document>) findIterable;
```
等价于Mongo脚本：

```bash
db.bszn.find({$and:[{'province.value': '某省份'}, {'city.value': '某城市'}]})
```

## $in查询

```java
 List<String> idList = Arrays.asList("5a08fea2704f44dda6e3d0a25d89014a", "96e71f036fa74bceac41130902f8d2be");
        BasicDBObject query = new BasicDBObject();
        BasicDBList conditionList = new BasicDBList();
        for (String id : idList) {
            conditionList.add(id);
        }
        // 在这里用$in做键，用上面定义的字符串数组为值
        query.put("classify_id", new BasicDBObject("$in", conditionList));
        FindIterable<Document> findIterable = mongoTemplate.getCollection("crawler_classify").find(query);
        MongoCursor<Document> cursor = findIterable.iterator();
        IndustryRepoClassify question = null;
        List<IndustryRepoClassify> list = Lists.newArrayList();
        while (cursor.hasNext()) {
            question = mongoConverter.read(IndustryRepoClassify.class, cursor.next());
            list.add(question);
        }
        cursor.close();
```
等价于Mongo脚本：

```bash
db.crawler_classify.find({'classify_id': 
{$in:['5a08fea2704f44dda6e3d0a25d89014a', '96e71f036fa74bceac41130902f8d2be']}})
```

## 更新某个属性

```java
 	Bson filter = new BasicDBObject().append("_id", new ObjectId("5def5b6d5f090d15618df344"));
    BasicDBObject set = new BasicDBObject();
    set.append("kobe.value","33,");
    Bson update =  new Document("$set",set);
    crawlMongoTemplate.getCollection("test").updateOne(filter, update);
```
等价于Mongo脚本：

```bash
db.test.update({'_id': new ObjectId("5def5b6d5f090d15618df344")}, {$set:{'kobe.value':'33,'}},{})
```

## 用于接取Mongo数据的Java Bean定义注意事项

```java
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Lists;
import lombok.*;
import org.bson.types.ObjectId;
import org.springframework.data.mongodb.core.mapping.Field;
import java.util.List;
import java.util.Map;

/**
 * @Date: 2019/12/11 17:37
 */
@Data
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class CjwtMongoEntity {
    /**
     * mongoid
     */
    private ObjectId id;
    /**
     * url
     */
    @Field
    private Map url;
    /**
     * 数据来源
     */
    @Field
    private Map source;
    /**
     * 数据采集时间
     */
    @Field
    private Map time;
    /**
     * 批次号
     */
    @Field("batch_number")
    private Map batchNumber;
    /**
     * 省份
     */
    @Field
    private Map province;
    /**
     * 数据来源 人为添加：artificial 爬虫添加 crawler
     */
    @Field("data_source")
    private Map dataSource;
}

```
注意的点有两个，一是id直接定义为ObjectId类型就可以了，二是其他的属性需要与Mongo文档中对应，如果不是完全一致，则需要在@Field注解后面加上Mongo文档中的键名。