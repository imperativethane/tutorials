# Mongoose

## Environment Setup

```javascript
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;
```
Requires in mongoose and ????

```javascript
const env = process.env.NODE_ENV || 'development';
const databaseUrl = process.env.DATABASE_URL || `mongodb://localhost/learn-mongoose_${env}`;

const options= {
  useMongoClient: true,
};
```
Database envrionment setup

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
runWithDatabase makes use of different MongoDB properties:
1. Connect - https://mongoosejs.com/docs/api.html#index_Mongoose-connect
1. Drop - https://docs.mongodb.com/manual/reference/method/db.dropDatabase/
1. Disconnect - https://mongoosejs.com/docs/api.html#index_Mongoose-disconnect

It's purpose:
1. Connects to the mongoose database using the given url
1. Drops the old database
1. Runs the function that is passed to it e.g. 

```javascript runWithDatabase(async () => {
  // Create and save a document
  await Poem.create(poemProperties);
});
```
4. Disconnects from the database

