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

## Modeling in JSON - A Blog

- Imagine that we wanted to create a blogging system.
- This blog will have users, posts, and comments.
- Users can have many posts and comments.
- How can we model this with JSON?

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

- Let's import the restaurants.json file from here: https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/restaurants.json

```bash
curl -o restaurants.json https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/restaurants.json
```

- Let's also download and import the zips.json file from here: https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/zips.json

```bash
curl -o zips.json https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/zips.json
``` 

## Basic Queries

#### Find All Records

- The most simplistic query is to find all records for a certain collection.
- The `.find()` method can take no parameters, thus returning all records:

```javascript
db.collection.find();
```

- Let's try that on the `zips` collection.

#### Select Certain Fields

- If you need to select only certain fields instead of returning the entire document, you can specify this after the query object:

```javascript
db.zips.find({}, { state: 1, city: 1, _id: 0 });
```

#### Conditional Queries

- Conditional queries allow us to specify conditions for results that get returned.
- Let's take an example with the zips dataset:

```javascript
db.zips.find({ state: "CA" });
```

- We can also alter the query to return just a single result:

```javascript
db.zips.findOne({ state: "CA" });
```

- AND operations are very simple and can be accomplished by simply adding more attributes to the object:

```javascript
db.zips.find({ state: "CA", city: "LOS ANGELES" });
```

#### Limiting

- When you need to limit the amount of data that comes back you can use the `.limit()` method.

```javascript
db.zips.find().limit(5);
```

#### Using Operators

- Operators allow you to use in-built helpers to accomplish filtered queries.
- A list of query operators can be found here: https://docs.mongodb.com/manual/reference/operator/query
- Let's see an example with an operator that checks for population size above a certain level:

```javascript
db.zips.find({
    pop: {
        $gt: 5000
    }
});
```

- Let's see another example of querying for longitude below -72:

```javascript
db.zips.find({
    loc: {
        $elemMatch: {
            $lt: -80
        }
    }
});
```

#### Multiple Conditions with AND

- When you have multiple conditions that you need to execute together you can use the `$and` operator.
- Let's take an example where we are finding all records within the state of PA, with population greater than 30000, and with the city name starting with the letters "ra":

```javascript
db.zips.find({
    $and: [
        { state: "PA" },
        { pop: { $gte: 30000 } },
        { city: /^ra+/i }
    ]
});
```

#### Multiple Conditions with OR

- WHen you have multiple conditions that you want to execute non mutually exclusive, you can use the `$or` operator.
- Let's say we want to find zip records for PA or CA:

```javascript
db.zips.find({
    $or: [
        { state: "PA" },
        { state: "CA" }
    ]
});
```

## Sub Document Queries

- In MongoDB you can request information based on conditions in a sub document.
- The syntax is similar to JavaScript object syntax. Let's take the example of retrieving records based on building number:

```javascript
db.restaurants.find({
    "address.building": "1007"
});
```

- You can even use operators on sub documents:

```javascript
db.restaurants.find({
    "grades.score": {
        $gt: 30
    }
});
```

## Mongo Lab 2

- Step 1: Write a query that retrieves one document in the restaurants database so you can see the structure of the data.
```
db.restaurants.findOne({})
```
- Step 2: Write a query that searches the restaurants database for locations where the borough is "Manhattan".
```
db.restaurants.find({'borough': 'Manhattan'})
```
- Step 3: Write a query that pulls all restaurants that have received a "B" rating at some point.
```
db.restaurants.find({'grades': {$elemMatch: {'grade': 'B'}})

// Short cut - not recommended
db.restaurants.find({'grades.grade': 'B'})
```
- Step 4: Add to the query in step 3 a sort filter that sorts alphabetically by borough name. You will have to look up the `.sort()` method.

```
db.restaurants.find({'grades': {$elemMatch: {'grade': 'B'}}}).sort({'borough': -1})
```

## Basic Querying Section Lab

<details>
    <summary>Section Lab Question 1</summary>
    Write a MongoDB query to display all the documents in the collection restaurants.
	
```
db.restaurants.find({})

//or

db.restaurants.find().limit(1)
```
	
</details>

<details>
    <summary>Section Lab Question 2</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and cuisine for all the documents in the collection restaurant.
	
```
db.restaurants.find({}, {'_id': 1, 'name': 1, 'borough': 1, 'cuisine': 1})
```
	
</details>

<details>
    <summary>Section Lab Question 3</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and cuisine, but exclude the field _id for all the documents in the collection restaurant.

```
db.restaurants.find({}, {'_id': 0, 'name': 1, 'borough': 1, 'cuisine': 1})
```

	
</details>

<details>
    <summary>Section Lab Question 4</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and zip code, but exclude the field _id for all the documents in the collection restaurant.
	
```
db.restaurants.find({}, {'_id': 0, 'name': 1, 'borough': 1, 'cuisine': 1, 'address.zipcode': 1})
```
	
</details>

<details>
    <summary>Section Lab Question 5</summary>
    Write a MongoDB query to display all the restaurant which are in the borough Bronx.

```
db.restaurants.find({borough: 'Bronx'})
```
</details>

<details>
    <summary>Section Lab Question 6</summary>
    Write a MongoDB query to display the first 5 restaurant which are in the borough Bronx.
	
```
db.restaurants.find({borough: 'Bronx'}).limit(5)
```
	
</details>

<details>
    <summary>Section Lab Question 7</summary>
    Write a MongoDB query to display the next 5 restaurants after skipping first 5 which are in the borough Bronx. (Hint: `.skip()`)
	
```
db.restaurants.find({'borough': 'Bronx'}).skip(5).limit(5)
```
	
</details>

<details>
    <summary>Section Lab Question 8</summary>
    Write a MongoDB query to find the restaurants who achieved a score more than 90.

```
db.restaurants.find({'grades': {$elemMatch: {score: {$gt: 90}}}})
// This just finds restaurants with at least one score gt 90. To match against all the score gt 90 - need to use unwind operator - day 2
```

</details>

<details>
    <summary>Section Lab Question 9</summary>
    Write a MongoDB query to find the restaurants that achieved a score more than 80 but less than 100.
	
```
db.restaurants.find({'grades': {$elemMatch: {score: {$gt: 80, $lt: 100}}}})
```	

</details>

<details>
    <summary>Section Lab Question 10</summary>
    Write a MongoDB query to find restaurants which are located in longitude value less than -95.754168.

```
db.restaurants.find({'address.coord.0': {$lt: -95.754168}})
```

</details>

<details>
    <summary>Section Lab Question 11</summary>
    Write a MongoDB query to find the restaurants that do not prepare any cuisine of 'American' and are located at a longitude less than -65.754168. (Hint: `$ne`)
</details>

<details>
    <summary>Section Lab Question 12</summary>
    Write a MongoDB query to find the restaurants which do not prepare any cuisine of 'American' and achieved a grade point 'A' and do not belong to the borough Brooklyn. The document must be displayed according to the cuisine in descending order.
</details>

<details>
    <summary>Section Lab Question 13</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which contain 'Wil' as first three letters for its name.
</details>

<details>
    <summary>Section Lab Question 14</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which contain 'ces' as last three letters for its name.
</details>

<details>
    <summary>Section Lab Question 15</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which contain 'Reg' as three letters somewhere in its name.
</details>

<details>
    <summary>Section Lab Question 16</summary>
    Write a MongoDB query to find the restaurants which belong to the borough Bronx and prepare either American or Chinese dishes.
</details>

<details>
    <summary>Section Lab Question 17</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which belong to the borough Staten Island or Queens or Bronx or Brooklyn.
</details>

<details>
    <summary>Section Lab Question 18</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which do not belong to the borough Staten Island or Queens or Bronx or Brooklyn.
</details>

<details>
    <summary>Section Lab Question 19</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which achieved a score which is not more than 10.
</details>

<details>
    <summary>Section Lab Question 20</summary>
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which prepare dishes except 'American' and 'Chinese' and the restaurant's name begins with letters 'Wil'.
</details>

<details>
    <summary>Section Lab Question 21</summary>
    Write a MongoDB query to find the restaurant Id, name, and grades for those restaurants which achieved a grade of "A" and scored 11 on an ISODate "2014-08-11T00:00:00Z" among many of survey dates.
</details>

<details>
    <summary>Section Lab Question 22</summary>
    Write a MongoDB query to find the restaurant Id, name and grades for those restaurants where the 2nd element of grades array contains a grade of "A" and score 9 on an ISODate "2014-08-11T00:00:00Z".
</details>

<details>
    <summary>Section Lab Question 23</summary>
    Write a MongoDB query to find the restaurant Id, name, and address for those restaurants where the 2nd element of the coord array contains a value which is more than 42 and up to 52.
</details>

<details>
    <summary>Section Lab Question 24</summary>
    Write a MongoDB query to arrange the name of the restaurants in ascending order along with all of the columns.
</details>

<details>
    <summary>Section Lab Question 25</summary>
    Write a MongoDB query to arrange the name of the restaurants in descending order along with all of the columns.
</details>

<details>
    <summary>Section Lab Question 26</summary>
    Write a MongoDB query to sort the name of the cuisine in ascending order along with the borough in descending order.
</details>

<details>
    <summary>Section Lab Question 27</summary>
    Write a MongoDB query that returns all addresses that contain streets.
</details>

<details>
    <summary>Section Lab Question 28</summary>
    Write a MongoDB query which will select all documents in the restaurants collection where the coord field value type is "double".
</details>

<details>
    <summary>Section Lab Question 29</summary>
    Write a MongoDB query which will select the restaurant Id, name and grades for those restaurants which return 0 as a remainder after dividing the score by 7.
</details>

<details>
    <summary>Section Lab Question 30</summary>
    Write a MongoDB query to find the restaurant name, borough, longitude, latitude and cuisine for those restaurants which contain 'mon' as three letters somewhere in its name.
</details>

<details>
    <summary>Section Lab Question 31</summary>
    Write a MongoDB query to find the restaurant name, borough, longitude, latitude and cuisine for those restaurants which contain 'Mad' as the first three letters of its name.
</details>
