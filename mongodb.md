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

