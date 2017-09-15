# Introduction to MongoDB

## Installation

- Let's follow the instructions here: https://docs.mongodb.com/manual/installation
- After installation it is important to start the MongoDB daemon process so Mongo runs in the background:

```bash
mongod --fork --syslog
```

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