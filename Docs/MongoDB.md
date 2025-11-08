# MongoDB

install mongodb and MongoDB Compose

no sql
Bson
fields->documents->collection->databse
key value pair

mongod --version
sudo systemctl status mongod
dpkg -l | grep mongodb

mongosh, mongodb compose vs atlas

show dbs
db.getName()

can run js codes in mongosh

//writing

db.collectionName.insertOne()
db.movies.insertOne({"name":"Foo", rating:10})
db.movies.insertOne.help()
db.movies.insertMany.help()

m1 = {"name":"A Foo 1", rating:4}
m2 = {"name":"B Foo 2", rating:5}
m3 = {"name":"C Foo 3", rating:6}
db.movies.insertMany([m1,m2,m3])

//reading
show collections

db.movies.find()
db.movies.find({"name": "Foo 1"})
db.movies.find({"name": "Foo 1"}, {"name":1}) //only name
db.movies.find({"name": "Foo 1"}, {"name":0}) //nothing but the name
db.movies.find({}, {"name":1}) //all docs with only name

db.movies.find().count()
db.movies.find().limit(2) //first 2
db.movies.find({}, {"name":1}).sort({"name":1}) //sort by name //-1 for decending order
db.movies.find({}, {"name":1}).sort({"name":1}).skip(1)

db.movies.find({"rating": {$lt:5}}) //rating less than 5 //$gt for greater than, $and, $or...

//update
db.movies.updateOne.help()
db.movies.updateOne({"name": "Foo 1"}, {$set: {rating:6}})
db.movies.updateMany({rating:10}, {$set: {rating:5}})

//delete
db.movies.deleteOne.help()
db.movies.deleteOne({"name": "Foo 1"})
db.movies.deleteMany({rating:5})

//since js is possible in monogo shell
const results = db.movies.find().toArray()
console.table(results)

```javascript
npm install mongodb
const { MongoClient, ServerApiVersion } = require('mongodb');
const uri = "mongodb+srv://mdissanayake994:<db_password>@cluster0.o9rmcno.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0";

// Create a MongoClient with a MongoClientOptions object to set the Stable API version
const client = new MongoClient(uri, {
  serverApi: {
    version: ServerApiVersion.v1,
    strict: true,
    deprecationErrors: true,
  }
});

async function run() {
  try {
    // Connect the client to the server	(optional starting in v4.7)
    await client.connect();
    // Send a ping to confirm a successful connection
    await client.db("admin").command({ ping: 1 });
    console.log("Pinged your deployment. You successfully connected to MongoDB!");
  } finally {
    // Ensures that the client will close when you finish/error
    await client.close();
  }
}
run().catch(console.dir);
```

//mongoose
ODM library for Node and mongo db, provide high level abstraction layer,
manage relationship between data, provide schema validation, and used to translate between objects in code and the representation of those objects in MongoDb.

schema definitions
validations
support middlewares...

```javascript
npm init -y

npm i express nodemon mongoose dotenv

update package.json

"scripts": {
    "dev": "nodemon index.js"
},

//index.js
import express from 'express';
import connectDB from './db/connectDB.js'
const app = express();

const port = process.env.PORT || 3000;
const DATABASE_URL = process.env.DATABASE_URL || 'mongodb://127.0.0.1:27017/movies'
connectDB(DATABASE_URL)
app.listen(port, ()=>console.log(`Server is running on port ${port}`));

//db/connectDB.js
import mongoose from "mongoose"

const connectDB = async (DATABASE_URL) => {

    try{
        await mongoose.connect(DATABASE_URL)
        console.log('Database Connected...')
    }catch (error) {
        console.log("Error connecting to database: ", error);
    }
}
export default connectDB;
```

Schema? blueprint?

```javascript
//models/Movies.js

const movieSchema = new mongoose.Schema({
  key:type, //Shorthand
  keyTwo: {type:String}, //recommended
  //Examples
  name: {type: String, required:true, trim: true},
  rating: { type: Number, required: true, min: 1, max: 5 },
    money: {
        type: mongoose.Decimal128,
        required: true,
        validate: v => v >= 10
    },
    genre: { type: Array },
    isActive: { type: Boolean },
    comments: [
        {
            value: { type: String },
            published: { type: Date, default: Date.now }
        }
    ]
})
```

//Model
what is model?

```javascript

//in the schema definition file itself
const ModelName = mongoose.model("Movie", movieSchema) //uppercase? plural convert to singular in creation phase
export default movieModel;

//index.js
import movieModel from './models/Movies.js';

//Or with a CRUD operation like this

const createMovie = async () => {
    try {
        const m1 = new movieModel({
            name: "A Foo 1",
            rating: 4,
            money: 60000,
            genre: ['action', 'adventure'],
            isActive: true,
            comments: [{ value: "some comments", published: Date.now }]
        })

        const result = await m1.save() //insert one
        const result = movieModel.insertMany([m1,m2]) //insert many
        console.log(result)
    } catch (error) {
        console.log(error)
    }
}

export {createMovie};

//index.js
import {createMovie} from './models/Movies.js';

createMovie();

//now you can check and refresh the MongoDB Compass and see the newly inserted record

//Retrieving
//use a function same as insertData
const result = await movieModel.find() //All data
//can iterate ove the result like result.forEach(()=>{})
const result = await movieModel.findById("id string") //findById
const result = await movieModel.findById("id string", "name") //findById and get only one property
const result = await movieModel.find({name:'Foo'}) //find by property

//Update
//use a function same as insertData
const result = await movieModel.updateOne({
  {_id:id},
  {name: "Updated"}
})

const result = await movieModel.updateMany({rating:5}, {name:"rating 5 name updated"})

//Deleting
//use a function same as insertData
const result = await movieModel.findByIdAndDelete("68280a77c3933f705a9dc8b1");
const result = await movieModel.deleteOne({name:"Foo"});
const result = await movieModel.deleteMany({rating:5});
```