---
layout: post
title: mongoDB删除重复数据
category : mongo
tagline: "Supporting tagline"
tags : [mongo]
---
{% include JB/setup %}
# **mongoDB删除重复数据**

---


最近接了一个需求，发现由于之前的程序bug导致数据库中有大量的重复数据，需要把这些重复数据删掉，最开始想的是两遍遍历去作比较删除，这样的话时间复杂度是O(n2)，
<!--break-->
```
var used = new Array();
for(var i=0;i<db.getCollection('collection_name').find({"fieldName" : fieldValue}).count();i++){
    var res = db.getCollection('collection_name').find({"fieldName" : fieldValue});
    var re = res.next();
    while(used.indexOf(re)>-1){
        re= res.next();
    }
    if(used.indexOf(re)==-1){
        used.push(re);
     }
        var arr = new Array();
        var res1 = db.getCollection('collection_name').find({"fieldName" : fieldValue});
        while(res1.hasNext()){
            re1=res1.next();
            if(re.lon==re1.lon && re.lat==re1.lat && re.customerId.toString() == re1.customerId.toString() && re.time.toISOString() == re1.time.toISOString() && !re._id.equals(re1._id) ){
                arr.push(re1._id);
            }
        }
        used.length = 0;
        db.getCollection('collection_name').remove({"_id":{"$in":arr}});
    }
```

本来没觉得有问题，后来看了下mongo中文档的总数，是亿级的，瞬间放弃了这个方法，开始考虑有没有线性复杂度的，只做一遍比较就可以，搜索了一下发现可以用mongo已有的方法来实现，先按条件聚合，然后再统计数量大于1的，具体实现如下

```
db.getCollection('collection_name').aggregate([{$group:{_id:{customerId:"$customerId"}, dups:{$push:"$_id"}, count: {$sum: 1}}},
{$match:{count: {$gt: 1}}}
]).length .forEach(function(doc){
    if(doc._id.customerId == 31781){
          doc.dups.shift();
  db.getCollection('customer_gps').remove({_id : {$in: doc.dups}});
        }
});
```
问题解决。