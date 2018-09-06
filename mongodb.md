## Create an Index

To create an index in the[Mongo Shell](https://docs.mongodb.com/manual/tutorial/getting-started/), use[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex).

[copycopied]()

```
db.collection.createIndex( <key and index type specification>, <options> )
```

The following example creates a single key descending index on the`name`field:

[copycopied]()

```
db.collection.createIndex( { name: -1 } )
```

The[`db.collection.createIndex`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)method only creates an index if an index of the same specification does not already exist.

## Create a Compound Index

To create a[compound index](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)use an operation that resembles the following prototype:

[copycopied]()

```
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )
```

The value of the field in the index specification describes the kind of index for that field. For example, a value of`1`specifies an index that orders items in ascending order. A value of`-1`specifies an index that orders items in descending order. For additional index types, see[index types](https://docs.mongodb.com/manual/indexes/#index-types).

IMPORTANT

You may not create compound indexes that have`hashed`index type. You will receive an error if you attempt to create a compound index that includes[a hashed index field](https://docs.mongodb.com/manual/core/index-hashed/).

# MongoDB索引管理－索引的创建、查看、删除

索引是提高查询查询效率最有效的手段。索引是一种特殊的数据结构，索引以易于遍历的形式存储了数据的部分内容（如：一个特定的字段或一组字段值），索引会按一定规则对存储值进行排序，而且索引的存储位置在内存中，所在从索引中检索数据会非常快。如果没有索引，MongoDB必须扫描集合中的每一个文档，这种扫描的效率非常低，尤其是在数据量较大时。

1. [创建／重建索引](https://itbilu.com/database/mongo/E1tWQz4_e.html#index-create)
2. [查看索引](https://itbilu.com/database/mongo/E1tWQz4_e.html#show-index)
3. [删除索引](https://itbilu.com/database/mongo/E1tWQz4_e.html#drop-index)

### 1. 创建／重建索引 {#index-create}

MongoDB全新创建索引使用`ensureIndex()`方法，对于已存在的索引可以使用`reIndex()`进行重建。

#### 1.1 创建索引`ensureIndex()` {#ensure-index}

MongoDB创建索引使用`ensureIndex()`方法。

**语法结构**

```
db.COLLECTION_NAME.ensureIndex(keys[,options])
```

* `keys`
  ，要建立索引的参数列表。如：
  `{KEY:1}`
  ，其中
  `key`
  表示字段名，
  `1`
  表示升序排序，也可使用使用数字
  `-1`
  降序。
* `options`
  ，可选参数，表示建立索引的设置。可选值如下：
* * `background`
    ，Boolean，在后台建立索引，以便建立索引时不阻止其他数据库活动。默认值 false。
  * `unique`
    ，Boolean，创建唯一索引。默认值 false。
  * `name`
    ，String，指定索引的名称。如果未指定，MongoDB会生成一个索引字段的名称和排序顺序串联。
  * `dropDups`
    ，Boolean，创建唯一索引时，如果出现重复删除后续出现的相同索引，只保留第一个。
  * `sparse`
    ，Boolean，对文档中不存在的字段数据不启用索引。默认值是 false。
  * `v`
    ，index version，索引的版本号。
  * `weights`
    ，document，索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。

如，为集合`sites`建立索引：

```
>
 db.sites.ensureIndex({name: 1, domain: -1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

_注意：_`1.8`版本之前创建索引使用`createIndex()`，`1.8`版本之后已移除该方法

#### 1.2 重建索引`reIndex()` {#reIndex}

```
db.COLLECTION_NAME.reIndex()
```

如，重建集合`sites`的所有索引：

```
>
 db.sites.reIndex()
{
  "nIndexesWas" : 2,
  "nIndexes" : 2,
  "indexes" : [
    {
      "key" : {
        "_id" : 1
      },
      "name" : "_id_",
        "ns" : "newDB.sites"
    },
    {
      "key" : {
        "name" : 1,
        "domain" : -1
      },
      "name" : "name_1_domain_-1",
      "ns" : "newDB.sites"
    }
  ],
  "ok" : 1
}
```

### 2. 查看索引 {#show-index}

MongoDB提供了查看索引信息的方法：`getIndexes()`方法可以用来查看集合的所有索引，`totalIndexSize()`查看集合索引的总大小，`db.system.indexes.find()`查看数据库中所有索引信息。

#### 2.1 查看集合中的索引`getIndexes()` {#getIndexes}

```
db.COLLECTION_NAME.getIndexes()
```

如，查看集合`sites`中的索引：

```
>
db.sites.getIndexes()
[
  {
    "v" : 1,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "newDB.sites"
  },
  {
    "v" : 1,
    "key" : {
      "name" : 1,
      "domain" : -1
    },
    "name" : "name_1_domain_-1",
    "ns" : "newDB.sites"
  }
]
```

#### 2.2 查看集合中的索引大小`totalIndexSize()` {#totalIndexSize}

```
db.COLLECTION_NAME.totalIndexSize()
```

如，查看集合`sites`索引大小：

```
>
 db.sites.totalIndexSize()
16352
```

#### 2.3 查看数据库中所有索引`db.system.indexes.find()` {#indexes}

```
db.system.indexes.find()
```

如，当前数据库的所有索引：

```
>
 db.system.indexes.find()
```

### 3. 删除索引 {#drop-index}

不在需要的索引，我们可以将其删除。删除索引时，可以删除集合中的某一索引，可以删除全部索引。

#### 3.1 删除指定的索引`dropIndex()` {#dropIndex}

```
db.COLLECTION_NAME.dropIndex("INDEX-NAME")
```

如，删除集合`sites`中名为"name\_1\_domain\_-1"的索引：

```
>
 db.sites.dropIndex("name_1_domain_-1")
{ "nIndexesWas" : 2, "ok" : 1 }
```

#### 3.3 删除所有索引`dropIndexes()` {#dropIndexes}

```
db.COLLECTION_NAME.dropIndexes()
```

如，删除集合`sites`中所有的索引：

```
>
 db.sites.dropIndexes()
{
  "nIndexesWas" : 1,
  "msg" : "non-_id indexes dropped for collection",
  "ok" : 1
}
```



