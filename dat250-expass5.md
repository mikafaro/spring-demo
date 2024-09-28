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


### 3. Updating


### 4. Removing


### 5. Bulk Write



## Experiment 2


## Technical Difficulties and Pending Issues