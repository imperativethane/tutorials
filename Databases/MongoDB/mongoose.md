# Mongoose

## Environment Setup

```javascript
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;
```
1. Requires in mongoose
2. Sets Mongoose promise as global promise (this is no longer required as of Mongoose 5)

```javascript
const env = process.env.NODE_ENV || 'development';
const databaseUrl = process.env.DATABASE_URL || `mongodb://localhost/learn-mongoose_${env}`;

const options= {
  useMongoClient: true,
};
```
1. `env` let's the server know whether it should be running in 'development' or 'production' mode.
2. `databaseUrl` - On your localhost, you are telling the program to either go to your .env file to find connection string, and in case of failure, to fall back to a hard-coded version. MongoDB sets its own process.env.DATABASE_URL to the connection string you will need to connect to the database on their servers
3. `useMongoClient` - related to authentication. May have been deprecated in Mongoose 5.

```javascript 
const runWithDatabase = async (runWhileConnected) => {
  console.log('connecting to database...\n');
  await mongoose.connect(databaseUrl, options);

  console.log('dropping old data...\n');
  await mongoose.connection.db.dropDatabase();

  console.log('running function...\n');
  await runWhileConnected();

  console.log('\n');

  console.log('disconnecting from database...\n');
  await mongoose.disconnect();
  console.log('complete!\n');
};

module.exports = {
  mongoose,
  runWithDatabase,
};
```
**runWithDatabase** makes use of different MongoDB methods:
1. Connect - https://mongoosejs.com/docs/api.html#index_Mongoose-connect
2. Drop - https://docs.mongodb.com/manual/reference/method/db.dropDatabase/
3. Disconnect - https://mongoosejs.com/docs/api.html#index_Mongoose-disconnect

It's purpose:
1. Connects to the mongoose database using the given url
2. Drops the old database
3. Runs the function that is passed to it:

```javascript 
runWithDatabase(async () => {
  // Create and save a document
  await Poem.create(poemProperties);
});
```
4. Disconnects from the database

## Schema

Example

```javascript
const magicItemSchema = new mongoose.Schema({
  item: {
    type: String,
    required: true
  },
  magicalProperty: {
    type: String,
    required: true
  },
  unitCost: {
    type: Number,
    required: true
    },
  totalUnits: {
    type: Number,
    required: true
  }
});
```

1. Create an object that initiates a new mongoose schema.
2. Set it **paths** - similar to properties of an object. 
3. Define the property type e.g. string, number, date - https://mongoosejs.com/docs/schematypes.html
4. Can set it validators such as required - https://mongoosejs.com/docs/validation.html

## Models

Once the schema has been created, we then need to create a model to use this schema. The model maps to a collection in your MongoDB database.

Models are constructors that we define based on our schema. They represent documents which can be saved and retrieved from our database. All document creation and retrieval from the database is handled by these models.

Example
```javascript

const MagicItem = mongoose.model('MagicItem', magicItemSchema);
```
1. First argument is the singular name of the collection your model is for.
2. The second argument is your previously created schema.

## Creating the database

Creating documents and saving them to the database can be done by calling .create() on our model.

Example

```javascript
const MagicItem = mongoose.model('MagicItem', magicItemSchema);

const properties = {
    item: 'cloak',
    magicalProperty: 'invisibility',
    unitCost: 25,
    totalUnits: 100
  }

runWithDatabase(async () => {
  // Create and save a document
  await MagicItem.create(properties);
});
```
1. Use the `runWithDatabase()` function that connects to the database and runs our functions.
2. Pass in the `.create` function with the properties. In more complex examples this will be an array of objects rather than just one object.

## Queries

Once the database has been created and then you can run queries on the database. 

List of queries at: https://mongoosejs.com/docs/queries.html

Example

```javascript
const MagicItem = mongoose.model('MagicItem', magicItemSchema);

runWithDatabase(async () => {
    await  MagicItem.create(manyItems);
    let finder = await MagicItem.findOne({item: 'cloak'});
    console.log(`Found one: ${finder.item}`)
    let cheapObjects = await MagicItem.find({unitCost: {$lt: 50}});
    console.log(`Found ${cheapObjects.length} magic objects`);
});
```

1. `.findOne` is used to find one document within the collection.
2. `.find` will return all the documents that match the criteria.
3. Uses query operators to find all items with a `unitCost` less than 50. You can see a list of query operators at: https://docs.mongodb.com/manual/reference/operator/query/
    * `$gt` - greater than
    * `$gte` - greater than or equal to
    * `$lt` - less than
    * `$lte` - less than or equal to
    * `$eq` - equal to
    * `$and` - returns document which matches all clauses
    * `$not` - returns documents that doesn't match the query expression
    * `$or` - returns documents that match one of the query expressions.

## Methods

### Model Methods - .statics()

.statics() adds static “class” methods to the model.

Example

```javascript
const magicItemSchema = new mongoose.Schema({
...
});

magicItemSchema.statics.findMostExpensive = function(callback) {
  return this.findOne({}).sort('unitCost').exec(callback);
};
```

In this example we create a method `.findMostExpensive`, which is a callback function that sorts the documents by `unitCost` then returns the first one. 

We can then use the method by calling it inside `runWithDatabase()`

```javascript
runWithDatabase(async () => {
    await MagicItem.create(manyItems);
  
    const mostExpensive = await MagicItem.findMostExpensive();
    console.log(`The most expensive object is the ${mostExpensive.item}`);
    console.log(`The ${mostExpensive.item} started with ${mostExpensive.totalUnits} charges.`);
});
```

### Document Methods - .methods()

`.methods()` adds an instance method to documents.

Example

```javascript
magicItemSchema.methods.use = function(callback) {
  this.totalUnits -= this.unitCost;
  return this.save();
};
```

1. Creates a document method called `.use` that is a callback funciton that deducts `unitCost` from `totalUnits`.
2. `.save` writes the current JavaScript object as a MongoDB document.

We can then use the method by calling it inside `runWithDatabase`.

```javascript
runWithDatabase(async () => {
	await  MagicItem.create(manyItems);
  
    const mostExpensive = await MagicItem.findMostExpensive();
    console.log(`Using ${mostExpensive.item}...`);    
    await mostExpensive.use();
    console.log(`The ${mostExpensive.item} has ${mostExpensive.totalUnits} charges left.`);
});
```