# MongoDB Continued

## Warmup Practice

- Step 1: Create a new database called "mongo_practice".
- Step 2: Insert the documents below into a "movies" collection using a JSON file that you create.
- Note: In order to use `mongoimport` with JSON array data you need to pass the `--jsonArray` flag along with the command.

```
title : Fight Club
writer : Chuck Palahniuk
year : 1999
actors : [
  Brad Pitt
  Edward Norton
]
```
```
title : Pulp Fiction
writer : Quentin Tarantino
year : 1994
actors : [
  John Travolta
  Uma Thurman
]
```
```
title : Inglorious Basterds
writer : Quentin Tarantino
year : 2009
actors : [
  Brad Pitt
  Diane Kruger
  Eli Roth
]
```
```
title : The Hobbit: An Unexpected Journey
writer : J.R.R. Tolkein
year : 2012
franchise : The Hobbit
```
```
title : The Hobbit: The Desolation of Smaug
writer : J.R.R. Tolkein
year : 2013
franchise : The Hobbit
```
```
title : The Hobbit: The Battle of the Five Armies
writer : J.R.R. Tolkein
year : 2012
franchise : The Hobbit
synopsis : Bilbo and Company are forced to engage in a war against an array of combatants and keep the Lonely Mountain from falling into the hands of a rising darkness.
```
```
title : Pee Wee Herman's Big Adventure
```
```
title : Avatar
```

- Step 3: Query the movies collection to:

1. Get all documents
2. Get all documents with `writer` set to "Quentin Tarantino"
3. Get all documents where `actors` include "Brad Pitt"
4. Get all documents with `franchise` set to "The Hobbit"
5. Get all movies released in the 90s
6. Get all movies released before the year 2000 or after 2010

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

## Using GridFS to Store Files

- Normally if you want to save files in the BSON format, there is a 16 MB limit on the filesize.
- With GridFS, you can store much larger files, and MongoDB will divide the file into chunks and store each chunk as a separate document.
- The default chunk size for GridFS is 255 KB.
- GridFS uses two collections to store files - one collection stores the file chunks themselves (chunks), and the other stores file metadata (files).
- To interact with GridFS we will use the `mongofiles` command line utility.

#### Listing Files in GridFS

- You can list files stored using GridFS in a specific database by running this command:

```
mongofiles -d sample_data list
```

- This will list all of the files that are currently saved to the database named "sample_data".

#### Adding Files to GridFS

- You can use the `put` command to insert files into GridFS:

```
mongofiles -d sample_data put zips.json
```

#### Getting Files from GridFS

- You can retrieve files from GridFS using the `get` command.
- Note that the file will be saved in your current command line location.

```
mongofiles -d sample_data get zips.json
```

#### Deleting Files from GridFS

- You can also delete files from GridFS using the `delete` command.

```
mongofiles -d sample_data delete zips.json
```

## Integration with Node JS

- Since MongoDB contains a JavaScript runtime, it is an obvious choice to use it in conjunction with Node JS.
- If we want Node JS to talk to MongoDB however we need to install a driver that interfaces between the two.
- We will be using the NPM package "mongodb" to make this happen.

#### Install Node JS

- Let's install Node JS: https://nodejs.org/en
- Make sure the install worked

```bash
node -v
npm -v
```

#### Set Up New Node Project

- We will be working with a new Node project, so let's create a folder called "todo_list" and CD into it.
- We will then setup the project to use NPM and mongodb:

```bash
npm init

npm install mongodb --save
```

## One-to-Many Relationships

- MongoDB wasn't designed to be a relational database; however, simple relationships can still exist.
- In a one-to-many scenario, we can have each todo item contain a user_id field, which will hold the ObjectId of the user who created it.
- Let's create two files - one to add a new user and one to add a new todo.
- We will create the functionality of each:

add_user.js

```javascript
const MongoClient = require("mongodb").MongoClient;

const url = "mongodb://localhost:27017/todo_db";

MongoClient.connect(url, (err, db) => {
  db.collection("users").insert({
    firstname: "Arun",
    lastname: "Sood"
  }, (err, savedUser) => {
    console.log(savedUser);
    db.close();
  });
});
```

add_todo.js

```javascript
const MongoClient = require("mongodb").MongoClient;
const ObjectId = require("mongodb").ObjectID;

const url = "mongodb://localhost:27017/todo_db";

const userId = new ObjectId("59c49d69fab174092c62b2b6");

MongoClient.connect(url, (err, db) => {
  db.collection("todos").insert({
    todo_text: "Mop the floor",
    user_id: userId
  }, (err, savedTodo) => {
    console.log(savedTodo);
    db.close();
  });
});
```

- Let's try to use similar logic to implement read, update, and destroy operations.

## Pet List Lab

- We will be working on a pet list application to write each of the MongoDB queries necessary to create the functionality.
- Please refer to the project here: https://github.com/arun-projects/Pet-List-Node-Mongo

## Replication

- On a single server it is common to have one `mongod` process running.
- However, sometimes you may want to have multiple processes running to provide redundancy and high availability.
- Replication has the following benefits:
    - Read capacity is increased since clients can send read operations to different servers.
    - Maintaining copies of data in different data centers can increase data locality and availability for distributed applications.
    - You can choose to retain copies for purposes such as disaster recovery, reporting, or backup.

![Replica Set](img/replica_set.svg)

## Sharding

- Sharding is a method of distributing data across multiple machines.
- The fundamental reason for using sharding is for deployments with very large datasets and high throughput operations.
- Addressing system growth is usually thought of from either a vertical or horizontal scaling perspective.

#### Vertical Scaling

- Vertical scaling involves increasing the capacity of the server.
- Using a more powerful CPU, more RAM, and increasing the storage space are some examples of how this can be done.
- This leads to a practical maximum ceiling for vertical scaling solutions.

#### Horizontal Scaling

- Horizontal scaling involves distributing the dataset and load over multiple servers, thus spreading out the work.
- As the system grows, more servers (mongos) can be added to take on the extra load.
- This architecture is called "sharding", and is performed in the following way:

![Sharded Clusters](img/sharded_clusters.svg)