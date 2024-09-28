# DAT250 Experiment Assignment 5 Report - MongoDB

## Introduction

The report details the basic usage of MongoDB by following the steps taken
with the official tutorial at MongoDB's website. The aim is to become familiar with
using the technology.

## Installation

In order to install I had to first download the gpg key and the mongodb repository as
I am using ubuntu.

```
> curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

```
> echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

I then reloaded the package database and installed the latest version:

```
> sudo apt-get update
> sudo apt-get install -y mongodb-org
```

When it was installed I used systemd to start the daemon

```
> sudo systemctl start mongod
```

## Experiment 1

### 1. Insertion

In order to insert a single dobument into a collection I used the following command, which
also creates the collection as it does not already exist at this time.

```
> db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

This gave me a response with the "acknowledged" field set to true, as well as the new object id.
To make sure that the document is in fact in the database I used the following command to retrieve it.

```
> db.inventory.find( { item: "canvas" } )
```

Now, to insert many documents at once I used the insertMany command and got a successful response.

```
> db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```

I also successfully retrieved all the documents

```
> db.inventory.find( {} )

[
  {
    _id: ObjectId('66f7cff3519ea972c2964033'),
    item: 'canvas',
    qty: 100,
    tags: [ 'cotton' ],
    size: { h: 28, w: 35.5, uom: 'cm' }
  },
  {
    _id: ObjectId('66f7d150519ea972c2964034'),
    item: 'journal',
    qty: 25,
    tags: [ 'blank', 'red' ],
    size: { h: 14, w: 21, uom: 'cm' }
  },
  {
    _id: ObjectId('66f7d150519ea972c2964035'),
    item: 'mat',
    qty: 85,
    tags: [ 'gray' ],
    size: { h: 27.9, w: 35.5, uom: 'cm' }
  },
  {
    _id: ObjectId('66f7d150519ea972c2964036'),
    item: 'mousepad',
    qty: 25,
    tags: [ 'gel', 'blue' ],
    size: { h: 19, w: 22.85, uom: 'cm' }
  }
]
```

### 2. Querying

I've already looked at simple find statements in the previous example, but it is interesting to
note how these statements correspond to statements in SQL, like the find statement with a field and value
specified being equivalent to the SQL statment:

```sql
SELECT * FROM inventory WHERE field = "value"
```

It is also possible to use query operators to retrieve documents where the field
satisfies one of the options provided. The following will then return the documents where
the status is either "A" or "D".

```
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```

We can also specify that the documents to be retrieved should satisfy one of the
field conditions we specify:

```
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

These commands all worked as expected on the supplied documents.


### 3. Updating

This part of the tutorial familiarizes us with the following commands

```
db.collection.updateOne(<filter>, <update>, <options>)
db.collection.updateMany(<filter>, <update>, <options>)
db.collection.replaceOne(<filter>, <update>, <options>)
```

First I had to insert documents for testing the updating functionality

```
db.inventory.insertMany( [
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
   { item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
   { item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );
```

Then I tried updating a single document through the following command, taking into
consideration that this only updates the *first* document that meets the criteria.

```
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

In order to update many documents at the same time I tested the following command,
which would update all documents where the "qty" field was less than 50. It
successfully updated 3 documents in the inventory collection.

```
db.inventory.updateMany(
   { "qty": { $lt: 50 } },
   {
     $set: { "size.uom": "in", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

In order to replace a document I supplied the function with a replacement document
containing only field:value pairs as it is not possible to use operators here:

```
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

To check that the replacement worked I then queried for the specific document again:

```
> db.inventory.find({item:"paper"})

[
  {
    _id: ObjectId('66f7dcdde51d4c498a964038'),
    item: 'paper',
    instock: [ { warehouse: 'A', qty: 60 }, { warehouse: 'B', qty: 40 } ]
  }
]
```

Notice how the document can have different fields when replacing.

### 4. Removing

The inventory collection now contains multiple documents, and I then tested removal of
documents meeting some criteria. I decided to use the "qty" field for this:

```
> db.inventory.deleteMany({qty:{ $gt: 50 }})
{ acknowledged: true, deletedCount: 5 }
```

Then there were 5 documents remaining in the collection, and I wanted to try deleting
the document for the item "paper" that we replaced earlier.

```
> db.inventory.deleteOne({item:"paper"})
{ acknowledged: true, deletedCount: 1 }
```

And finally to delete all documents in the collection I performed a deleteMany without specifying
any fields

```
> db.inventory.deleteMany({})
{ acknowledged: true, deletedCount: 4 }
```

### 5. Bulk Write

The final functionality to test in this first experiment is bulk write.
This allows us to perform many different operations in one go.
In order to do this I first had to set up the collection used for this example:

```
db.pizzas.insertMany( [
   { _id: 0, type: "pepperoni", size: "small", price: 4 },
   { _id: 1, type: "cheese", size: "medium", price: 7 },
   { _id: 2, type: "vegan", size: "large", price: 8 }
] )
```

Then we perform the bulk operation, using many of the commands we have previously tested:

```
try {
   db.pizzas.bulkWrite( [
      { insertOne: { document: { _id: 3, type: "beef", size: "medium", price: 6 } } },
      { insertOne: { document: { _id: 4, type: "sausage", size: "large", price: 10 } } },
      { updateOne: {
         filter: { type: "cheese" },
         update: { $set: { price: 8 } }
      } },
      { deleteOne: { filter: { type: "pepperoni"} } },
      { replaceOne: {
         filter: { type: "vegan" },
         replacement: { type: "tofu", size: "small", price: 4 }
      } }
   ] )
} catch( error ) {
   print( error )
}
```

which gave the result

```sql
{
  acknowledged: true,
  insertedCount: 2,
  insertedIds: { '0': 3, '1': 4 },
  matchedCount: 2,
  modifiedCount: 2,
  deletedCount: 1,
  upsertedCount: 0,
  upsertedIds: {}
}
```

## Experiment 2

For the second experiment I had a look at the mapReduce functionality of
MongoDB, although it is deprecated in the latest version in facor of Aggregation Pipelines.

First I had to create a collection to work with

```
db.orders.insertMany([
   { _id: 1, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-01"), price: 25, items: [ { sku: "oranges", qty: 5, price: 2.5 }, { sku: "apples", qty: 5, price: 2.5 } ], status: "A" },
   { _id: 2, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-08"), price: 70, items: [ { sku: "oranges", qty: 8, price: 2.5 }, { sku: "chocolates", qty: 5, price: 10 } ], status: "A" },
   { _id: 3, cust_id: "Busby Bee", ord_date: new Date("2020-03-08"), price: 50, items: [ { sku: "oranges", qty: 10, price: 2.5 }, { sku: "pears", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 4, cust_id: "Busby Bee", ord_date: new Date("2020-03-18"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 5, cust_id: "Busby Bee", ord_date: new Date("2020-03-19"), price: 50, items: [ { sku: "chocolates", qty: 5, price: 10 } ], status: "A"},
   { _id: 6, cust_id: "Cam Elot", ord_date: new Date("2020-03-19"), price: 35, items: [ { sku: "carrots", qty: 10, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 7, cust_id: "Cam Elot", ord_date: new Date("2020-03-20"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 8, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 75, items: [ { sku: "chocolates", qty: 5, price: 10 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 9, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 55, items: [ { sku: "carrots", qty: 5, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 }, { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 10, cust_id: "Don Quis", ord_date: new Date("2020-03-23"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" }
])
```

The aim was to perform the map-reduce on this collection and return the total price per customer.
Then I had to define the map and reduce functions to be used:

```js
var mapFunction1 = function() {
   emit(this.cust_id, this.price);
};

var reduceFunction1 = function(keyCustId, valuesPrices) {
    return Array.sum(valuesPrices);
};
```

I then performed the map reduce on the collection, and stored the result in a collection named "map_reduce_example".

```
db.orders.mapReduce(
   mapFunction1,
   reduceFunction1,
   { out: "map_reduce_example" }
)
```

I could now query the new collection for the results

```
> db.map_reduce_example.find().sort( { _id: 1 } )

[
  { _id: 'Ant O. Knee', value: 95 },
  { _id: 'Busby Bee', value: 125 },
  { _id: 'Cam Elot', value: 60 },
  { _id: 'Don Quis', value: 155 }
]
```

This concludes the tutorial and my investigation into the usage of MongoDB


## Technical Difficulties and Pending Issues

There were no real technical difficulties in this assignment. The only thing being
that in the latest version of MongoDB, the mapReduce functionality is deprecated (though still usable).
