# MongoDB Continued

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

#### Deleteing Files from GridFS

- You can also delete files from GridFS using the `delete` command.

```
mongofiles -d sample_data delete zips.json
```

## Integration with Node JS Using Mongoose

- Since MongoDB contains a JavaScript runtime, it is an obvious choice to use it in conjunction with Node JS.
- There are a few different Node packages that give us API methods to interact with MongoDB in a clear and simple way.
- One of these packages is called Mongoose, and it is essentially a piece of software that gives us data modeling, validation, and querying capabilities beyond the standard MongoDB interface.

## One-to-Many Relationships

## Mongo Lab 5 Part 1

## Mongo Lab 5 Part 2

## Mongo Lab 6

## Vertical vs Horizontal Scaling

## Replication

## Sharding

## Mongo Lab 7

## Mongo Lab 8

## Mongo Lab 9