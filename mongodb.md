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

