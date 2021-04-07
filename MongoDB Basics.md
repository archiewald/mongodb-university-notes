## What is MongoDB db?

Structure way to store data.

It is NOSQL document DB. Documents are stored in collections.

NoSQL - not stored in tables/rows/columns..

What is a document? Data stored as field-value pairs

```js
{
  "name" : "lenny",
  "age": 2
}
```

## Mongo Atlas - database as a service

Clusters - group of servers storing data

Replica set - connected instances storing the same data

## How MongoDB store data

BSON BinaryJSON

optimized for speed space and flexibility

Handles Date and Raw binary as well

Anything you can store in JSON you can store in BSON

Stored BSON and viewed as JSON

tools to import/export data

mongodumb/mongorestore - BSON
mongoimport/mongoexport - JSON

## find command

- cursor is a pointer to a result set of a query
- pointer is a direct address of the memory location

db.zips.find({"state": "NY"}).count()
db.zips.find({"state": "NY"}).pretty()

## inserting

- every document has a unique `_id` field. Created by default unless specified
- It is automatically generated as an ObjectId type value.
- You can select a non ObjectId type value when inserting a new document, as long as that value is unique to this document.
- MongoDB has schema validation funcionality
- MongoDB can store duplicate documents in the same collection, as long as their \_id values are different
- if insert to db/collection that doesnt exist Mongo will create it

## insert() command

- with `{ "ordered": false }` each document will be tried to be inserted. Otherwise the duplicate key error will prevent the operation from reaching the other documents

## updating

Update all documents in the zips collection where the city field is equal to "HUDSON" by adding 10 to the current value of the "pop" field.

```js
db.zips.updateMany({ city: "HUDSON" }, { $inc: { pop: 10 } });
```

Update a single document in the zips collection where the zip field is equal to "12534" by setting the value of the "pop" field to 17630.

```js
db.zips.updateOne({ zip: "12534" }, { $set: { pop: 17630 } });
```

Update one document in the grades collection where the student_id is `250` \*, and the class_id field is 339 , by adding a document element to the "scores" array.

```js
db.grades.updateOne(
  { student_id: 250, class_id: 339 },
  { $push: { scores: { type: "extra credit", score: 100 } } }
);
```

## deleting

```js
db.inspections.deleteMany({ test: 1 });
```

```js
db.inspections.deleteOne({ test: 1 });
```

delete collection:

```js
db.inspections.drop();
```

Removing all collections in a database also removes the database.

## MQL operators

$inc, $set, $unset

$eq - equal to (used as a default operator)
$ne - not equal to
$gt - greater than
$lt - less than

Find all documents where the tripduration was less than or equal to 70 seconds and the usertype was not Subscriber:

```js
db.trips
  .find({ tripduration: { $lte: 70 }, usertype: { $ne: "Subscriber" } })
  .pretty();
```

What is the difference between the number of people born in 1998 and the number of people born after 1998 in the sample_training.trips collection?

```js
db.trips.find({ "birth year": { $gt: 1998 } }).count() -
  db.trips.find({ "birth year": { $eq: 1998 } }).count();
```

## Logic operators

$and - default operator
$or
$nor
$not

implicit $and:

```json
{ "$and": [{ "student_id": { "$gt": 25 } }, { "student_id": { "$lt": 100 } }] }
```

```json
{ "student_id": { "$gt": 25 }, "student_id": { "$lt": 100 } }
```

```json
{ "student_id": { "$gt": 25, "$lt": 100 } }
```

How many zips in the sample_training.zips dataset are neither over-populated nor under-populated? In this case, we consider population of more than 1,000,000 to be over- populated and less than 5,000 to be under-populated.

```js
db.zips
  .find({ $nor: [{ pop: { $gt: 10000000 } }, { pop: { $lt: 5000 } }] })
  .count();
```

How many companies in the sample_training.companies dataset were either founded in 2004 and either have the social category_code or web category_code, or were founded in the month of October and also either have the social category_code or web category_code?

```js
db.companies
  .find({
    $or: [
      {
        founded_year: 2004,
        $or: [{ category_code: "web" }, { category_code: "social" }],
      },
      {
        founded_month: 10,
        $or: [{ category_code: "web" }, { category_code: "social" }],
      },
    ],
  })
  .count();
```

or

```js
db.companies
  .find({
    $and: [
      { $or: [{ founded_year: 2004 }, { founded_month: 10 }] },
      { $or: [{ category_code: "web" }, { category_code: "social" }] },
    ],
  })
  .count();
```

## expressive operator $expr

allows the use of aggregation expression within the query language

allows us to use variables and conditional statements

we can compare fields within the same document!

$ - denotes use of operator and adresses the field value

Find all documents where the trip lasted longer than 1200 seconds, and started and ended at the same station:

```js
db.trips
  .find({
    $expr: {
      $and: [
        { $gt: ["$tripduration", 1200] },
        { $eq: ["$end station id", "$start station id"] },
      ],
    },
  })
  .count();
```

## Array operators

Check if amenities contain shampoo (simplified query)

```json
{ "amenities": "shampoo" }
```

Find all documents with exactly 20 amenities which include all the amenities listed in the query array:

```js
db.listingsAndReviews
  .find({
    amenities: {
      $size: 20,
      $all: [
        // contains all the given elements regardless of order
        "Internet",
        "Wifi",
        "Kitchen",
        "Heating",
        "Family/kid friendly",
        "Washer",
        "Dryer",
        "Essentials",
        "Shampoo",
        "Hangers",
        "Hair dryer",
        "Iron",
        "Laptop friendly workspace",
      ],
    },
  })
  .pretty();
```

### Projection

```js
db.listingsAndReviews
  .find(
    {
      amenities: {
        $all: ["Internet"],
      },
    },
    { price: 1, address: 1 } // display their price and address
  )
  .pretty();
```

You can exclude a field with 0. You cannot mix 0s with 1s, with exception of `_id` field

```js
db.listingsAndReviews
  .find({ amenities: "Wifi" }, { price: 1, address: 1, _id: 0 })
  .pretty();
```

Find all documents where the student in class 431 received a grade higher than 85 for any type of assignment:

```js
db.grades
  .find({ class_id: 431 }, { scores: { $elemMatch: { score: { $gt: 85 } } } })
  .pretty();
```

## Array Operators and Sub-Docouments

Use dot notation to specify the adress of nested document.

```js
db.trips.findOne({ "start station location.type": "Point" }); // nested document querying
```

How many trips in the sample_training.trips collection started at stations that are to the west of the -74 longitude coordinate?

```js
db.trips.find({ "start station location.coordinates.0": { $lt: -74 } }).count();
```

## Aggregation framework

Another way of querying MongoDB documents. aggregate() can do what find() can and more.

```js
db.listingsAndReviews
  .find({ amenities: "Wifi" }, { price: 1, address: 1, _id: 0 })
  .pretty();
```

is equivalent to

```js
db.listingsAndReviews
  .aggregate([
    { $match: { amenities: "Wifi" } }, // pipeline pipe 1!
    { $project: { price: 1, address: 1, _id: 0 } }, // pipeline pipe 2!
  ])
  .pretty();
```

| Aggregation framework | Query Language |
| --------------------- | -------------- |
| $group                | filter         |
| compute               | update         |
| reshape               |                |

$group - takes stream of data and put in distinct "baskets"

Project only the address field value for each document, then group all documents into one document per address.country value, and count one for each document in each group.

```js
db.listingsAndReviews.aggregate([
  { $project: { address: 1, _id: 0 } },
  { $group: { _id: "$address.country", count: { $sum: 1 } } },
]);
```
