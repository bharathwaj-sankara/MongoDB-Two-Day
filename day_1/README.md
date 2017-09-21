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

## Sub Document Queries and Operators

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
- Step 2: Write a query that searches the restaurants database for locations where the borough is "Manhattan".
- Step 3: Write a query that pulls all restaurants that have received a "B" rating at some point.
- Step 4: Add to the query in step 4 a sort filter that sorts alphabetically by borough name. You will have to look up the `.sort()` method.

## Basic Querying Section Lab

<details>

<summary>Section Lab Part 1</summary>

<details>
    <summary>Section Lab Question 1</summary>
    Write a MongoDB query to display all the documents in the collection restaurants.
</details>

<details>
    <summary>Section Lab Question 2</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and cuisine for all the documents in the collection restaurant.
</details>

<details>
    <summary>Section Lab Question 3</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and cuisine, but exclude the field _id for all the documents in the collection restaurant.
</details>

<details>
    <summary>Section Lab Question 4</summary>
    Write a MongoDB query to display the fields restaurant_id, name, borough and zip code, but exclude the field _id for all the documents in the collection restaurant.
</details>

<details>
    <summary>Section Lab Question 5</summary>
    Write a MongoDB query to display all the restaurant which are in the borough Bronx.
</details>

</details>

<details>

<summary>Section Lab Part 2</summary>

<details>
    <summary>Section Lab Question 6</summary>
    Write a MongoDB query to display the first 5 restaurant which are in the borough Bronx.
</details>

<details>
    <summary>Section Lab Question 7</summary>
    Write a MongoDB query to display the next 5 restaurants after skipping first 5 which are in the borough Bronx.
</details>

<details>
    <summary>Section Lab Question 8</summary>
    Write a MongoDB query to find the restaurants who achieved a score more than 90.
</details>

<details>
    <summary>Section Lab Question 9</summary>
    Write a MongoDB query to find the restaurants that achieved a score more than 80 but less than 100.
</details>

<details>
    <summary>Section Lab Question 10</summary>
    Write a MongoDB query to find restaurants which are located in latitude value less than -95.754168.
</details>

</details>

<details>

<summary>Section Lab Part 3</summary>

<details>
    <summary>Section Lab Question 11</summary>
    Write a MongoDB query to find the restaurants that do not prepare any cuisine of 'American' and their grade score more than 70 and latitude less than -65.754168.
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

</details>

<details>

<summary>Section Lab Part 4</summary>

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
    Write a MongoDB query to find the restaurant Id, name, borough and cuisine for those restaurants which prepare dishes except 'American' and 'Chinees' or the restaurant's name begins with letters 'Wil'.
</details>

</details>

<details>

<summary>Section Lab Part 5</summary>

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
    Write a MongoDB query to find the restaurant Id, name, address and geographical location for those restaurants where the 2nd element of the coord array contains a value which is more than 42 and up to 52.
</details>

<details>
    <summary>Section Lab Question 24</summary>
    Write a MongoDB query to arrange the name of the restaurants in ascending order along with all of the columns.
</details>

<details>
    <summary>Section Lab Question 25</summary>
    Write a MongoDB query to arrange the name of the restaurants in descending order along with all of the columns.
</details>

</details>

<details>

<summary>Section Lab Part 6</summary>

<details>
    <summary>Section Lab Question 26</summary>
    Write a MongoDB query to sort the name of the cuisine in ascending order along with the borough in descending order.
</details>

<details>
    <summary>Section Lab Question 27</summary>
    Write a MongoDB query to check whether all of the addresses contain streets.
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
</details>

## Aggregation

- We will use the zips data that we imported before.
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

## Mongo Lab 3 Part 1

- In this lab we will try out a variety of aggregation steps to produce different data sets.
- Step 1: Write a query that will return all states along with their population IF the sum of their population is above 10 million. You will need to look up `$match` and `$gte`.
- Step 2: Write a query that will return the average population for each city. You will need to look up `$avg`.
- Step 3: Alter the query above to sort by the average population in descending order.

## Mongo Lab 3 Part 2

- In this lab we will use aggregation but now for the restaurants dataset.
- Step 1: Start by writing an aggregation query that groups the restaurants by borough.
- Step 2: Alter your query to $unwind the grades array. You will have to look this up.
- Step 3: Use the $avg operator to return the average score for restaurants in the various boroughs.
- Step 4: Add to your query above to restrict the results to only an average score of above 10.

## Geospatial Queries

- One aspect that MongoDB does really well is its implementation of geospatial queries.
- Operations like finding the records within a certain radius or that intersect with latitude and longitude coordinates can be performed with built-in methods.
- For this part let's download and import the restaurants sample data set:

```bash
curl -o restaurants_geo.json https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/restaurants_geo.json
```

- We will also download and import the neighborhoods sample data set:

```bash
curl -o neigborhoods.json https://raw.githubusercontent.com/arun-curriculum/MongoDB-Two-Day/master/sample_data/neighborhoods.json
```

- Now we can query for a user's neighborhood. Let's say the user is at 40.82302903 longitude and -73.93414657 latitude. Let's find their neighborhood:

```javascript
db.neighborhoods.findOne({
    geometry: {
        $geoIntersects: {
            $geometry: {
                type: "Point",
                coordinates: [ -73.93414657, 40.82302903 ]
            }
        }
    }
});
```

- $geometry is a special field that uses the GeoJSON format. $geoIntersects is specially designed to work with this format.
- What if we now want to find restaurants in a given neighborhood?

```javascript
var neighborhood = db.neighborhoods.findOne({
    geometry: {
        $geoIntersects: {
            $geometry: {
                type: "Point",
                coordinates: [ -73.93414657, 40.82302903 ]
            }
        }
    }
});

db.restaurants.find({
    location: {
        $geoWithin: {
            $geometry: neighborhood.geometry
        }
    }
});
```

- Notice here that we can use JavaScript code in our queries (`var`).
- This query essentially uses the neighborhood polygon denoted by the various latitude and longitude points to calculate restaurants inside.
- If we want to use a more traditional spherical format instead we can use $centerSphere:

```javascript
db.restaurants.find({
    location: {
        $geoWithin: {
            $centerSphere: [
                [ -73.93414657, 40.82302903 ],
                5 / 3963.2
            ]
        }
    }
});
```

## One-to-Many Relationships

## Mongo Lab 5 Part 1

## Mongo Lab 5 Part 2

## Using GridFS to Store Files

## Mongo Lab 6