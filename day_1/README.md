# Introduction to MongoDB

## Installation

- Let's follow the instructions here: https://docs.mongodb.com/manual/installation
- After installation it is important to start the MongoDB daemon process so Mongo runs in the background:

```bash
mongod --fork --syslog
```

- We will also install a GUI interface for interacting with MongoDB called Robo 3T: https://robomongo.org/download

## Introduction to MongoDB

- MongoDB is a NoSQL storage solution that is meant to store unstructured, non-relational data.
- Use cases for this may include chat messages, task queues (think cron jobs), and application logging for analytics.
- In MongoDB, you don't create a database, add tables to it, define relationships, and then query it.
- Instead, data is broken down into Collections, which are similar to the concept of tables, and Documents, which are similar to a row in a relational database.
- Contrary to relational databases, documents within a collection don't have to have the same fields. You can add a field to some documents in a collection without adding that field to all documents in the collection.
- Schemas are also flexible, which means that the structure of your database is not fixed.

##### Let's think of a structure for storing information about users in our database. Let's say that every user generally has these attributes:

- Name
- Username
- Email

##### Here is how we could represent users via JSON:

```javascript
var users = [
    {
        name: "Arun Sood",
        username: "arsood",
        email: "arsood@gmail.com"
    },
    {
        name: "John Smith",
        username: "jsmith",
        email: "john@smith.com"
    },
    {
        name: "Susan Mills",
        username: "smills",
        email: "smills@yahoo.com",
        address: "225 Bush Street, San Francisco, CA",
        gender: "Female",
        age: 34
    }
];
```

- As you can see, some users can have different fields in this model. There is no fixed schema.

## MongoDB Query Interface

- MongoDB has a rich set of tools that allow us to query collections to add data or find data.

##### Open Mongo interface

```
mongo
```

##### Show all databases

```
show dbs
```

##### Create database

```
use users_db
```

##### Add record to collection

- Note: Collections are generated upon the first insertion.

```javascript
db.users.insert({
    name: "Arun Sood",
    username: "arsood",
    email: "arsood@gmail.com"
});
```

##### Show all collections in a database

```
show collections
```

##### Find record in collection

By specific field:

```javascript
db.users.find({
    name: "Arun Sood"
});
```

By automatically generated ID:

```javascript
db.users.find({
    "_id": ObjectId("4ef224bf0fec2806da6e9b29")
});
```

##### Update record in collection

```javascript
db.users.update({
    name: "Arun Sood"
}, {
    $set: {
        email: "arun.instructor@gmail.com"
    }
});
```

##### Delete record in collection

```javascript
db.users.remove({
    name: "Arun Sood"
});
```

## Mongo Lab 1

- In this lab we will be practicing setting up a collection and querying it.
- You will need to follow these steps:
	- Step 1: Set up a new collection called "inventory" with the following attributes:
		- Product Name
		- Description
		- Quantity
		- Price
	- Step 2: Insert a few documents into the collection.
	- Step 3: Try a few different methods to query for the documents.
	- Step 4: Update a record.
	- Step 5: Remove a record.

## Capped Collections

- Capped collections are fixed-size collections that are optimized for high-throughput operations.
- Documents are inserted and retrieved based on insertion order.
- Once a collection fills its allocated space, it makes room for new documents by overwriting the oldest documents in the collection.
- Let's take an example of a log with a maximum of 5mb and 5000 documents:

```javascript
db.createCollection("Log", {
    capped: true,
    size: 5242880,
    max: 5000
});
```

## Importing Collections

- Import it into a MongoDB collection:

```bash
mongoimport --db some_db --collection some_collection --drop --file some_file.json
```

- Let's import the restaurants.json file from here: 

## Aggregation

- Download the file from here: http://media.mongodb.org/zips.json

```bash
curl -o zips.json http://media.mongodb.org/zips.json
```

- Import the file into MongoDB.
- The imported schema will have this format:

```javascript
{
    "_id": "10280",
    "city": "NEW YORK",
    "state": "NY",
    "pop": 5574,
    "loc": [
        -74.016323,
        40.710537
    ]
}
```

- MongoDB's `aggregate()` method processes documents into aggregated results.
- The aggregation pipeline consists of stages of processing that the documents pass through in order.
- Let's take an example with the zips data in which we will return grouped states along with the sums of their populations:

```javascript
db.zips.aggregate([
    {
        $group: {
            _id: "$state",
            totalPop: {
                $sum: "$pop"
            }
        }
    }
]);
```

- Notice that "totalPop" is not a property of the data set but is something that is created and included in the resulting dataset.
- Each step in the aggregation pipeline is defined in this object format.
- Every subsequent step will run in order.

## Mongo Lab 2

- In this lab we will try out a variety of aggregation steps to produce different data sets.
- Step 1: Write a query that will return all states along with their population IF the sum of their population is above 10 million. You will need to look up `$match` and `$gte`.
- Step 2: Write a query that will return the average population for each city. You will need to look up `$avg`.
- Step 3: Alter the query above to sort by the average population in descending order.