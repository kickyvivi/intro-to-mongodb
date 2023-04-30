# intro-to-mongodb
# Introduction to MongoDB

## Key terms to remember

[Official learning path](https://learn.mongodb.com/learning-paths/mongodb-python-developer-path)

[Coursera learning path](https://www.coursera.org/learn/introduction-mongodb)

## Week 1 - Getting Started with MongoDB & Basic Data Analysis

1. $group
2. $sort
3. $sortByCount
4. Filter using 'find' collection method in python and also '$match' aggregation
5. $limit
6. $bucketAuto
7. $skip
8. $sum

```python
pipeline = [
    {
        '$sortByCount': "$language"
    },
    {
        '$facet': {
            'top language combinations': [{'$limit': 100}],
            'unusual combinations shared by': [{
                '$skip': 100
            },
            {
                '$bucketAuto': {
                    'groupBy': "$count",
                    'buckets': 5,
                    'output': {
                        'language combinations': {'$sum': 1}
                    }
                }
            }]
        }
    }
]
```

```python
import pymongo
from pymongo import MongoClient
from IPython.display import clear_output
import pprint
from collections import OrderedDict
```

## Week 2 - Cleaning Data with MongoDB and Query Essentials

1. Ways to clean up data:
    - Declarative framework (Aggregation framework)
    - MongoDB query language using pyMongo driver in python

2. Aggregation framework

    - $project - cleaning up data - reshaping data
    - Include = 1, Exclude = 0
    - Restructure data into embedded documents
    - Reformat strings to correct formats for values E.g.: Dates
    - $cond : if, then, else
    - $ne : not equal to, !=, <>
    - $dateFromString
    ```python
    'released': {
                '$cond': {
                    'if': {'$ne': ["$released", ""]},
                    'then': {
                        '$dateFromString': {
                            'dateString': "$released"
                        }
                    },
                    'else': ""}}
    ```
    - $addFields
    - $arrayElemAt
    - E.g. lastUpdated: "2015-08-26 00:06:16.697000000"
    ```python
    #Splitting the string on '.' and choosing 0th index value
    {
        '$addFields': {
            'lastupdated': {
                '$arrayElemAt': [
                    {'$split': ["$lastupdated", "."]},
                    0
                ]}
        }
    }
    ```
    ```python
    #converting to datString format with timezone
    'lastUpdated': {
                '$cond': {
                    'if': {'$ne': ["$lastupdated", ""]},
                    'then': {
                        '$dateFromString': {
                            'dateString': "$lastupdated",
                            'timezone': "America/New_York"
                        }
                    },
                    'else': ""}}
    ```
    - $split
    - [Aggregation Pipeline Quick Reference](https://www.mongodb.com/docs/manual/meta/aggregation-quick-reference/?jmp=coursera-intro-mongodb)

3. MongoDB Query Language

    - **updating one document at a time**
    ```python
    db.movies.update_one({'_id': movie['_id']}, update_doc)
    ```
    - The above snippet says update the document with identifier movie['_id'] as the _id with the updates specified in update_doc
    - $set:
        - The key-values that need to be uupdated or newly added to document
    - $unset
        - The keys that need to be deleted from the document, either because they were transformed using $set and renamed or because you want to delete a bunch of keys on the docs with empty string values
    - **Updating bulk**
    - batch together updates and make single request to database for that batch
    - UpdateOne operation class from pyMongo is used
    - Create UpdateOne operation object
    - Create a list of operation objects stores as updates[] list and when size of updates list hit the batch size, we call the bulk_write method
    - Server will receive the batch and apply the update one at a time
    ```python
    from pymongo import MongoClient, UpdateOne
    ....
    ....
    batch_size = 1000

    update_doc = {}
    if fields_to_set:
        update_doc['$set'] = fields_to_set
    if fields_to_unset:
        update_doc['$unset'] = fields_to_unset

    #create list of operation objects
    updates.append(UpdateOne({'_id': movie['_id']}, update_doc))

    count += 1
    if count == batch_size:
        client.mflix.movies.bulk_write(updates)
        updates = []
        count = 0
    ```

4. Datatypes in MongoDB
    - Refer documentation for all datatypes
    - We can query MongoDB using the data types
        - cleansing data
        - when a field has different datatypes across different documents
    ```python
    movies.find_one({"runtime": {"$type": "string"}})
    ```
    - You should store money using the NumberDecimal (Decimal128) data type.
    - By default MongoDB stores all numbers as doubles.

5. Filtering on Array Fields
    - [find_by_language_all.ipynb](./notebooks%20week2/find_by_language_all.ipynb)
    - [korean_and_english_only.ipynb](notebooks%20week2\korean_and_english_only.ipynb)
    - [korean_first_language-in-array.ipynb](notebooks%20week2\korean_first_language-in-array.ipynb)

6. Sort, Skip, and Limit
    - [db.py](./mflix/mflix%202/mflix/db.py)

7. Query movies using operators
    - notebooks [movies-query-operators.ipynb](./notebooks%20week2/movies-query-operators.ipynb)
    - greater than equal to
    ```python
    filters = {'year': { '$gte': 1983 }}
    ```
    - greater than equal to and lower than
    ```python
    filters = { 'year': {'$gte': 1989, '$lt': 2000} }
    ```
    - exact match : $in
    ```python
    filters = { 'year': { '$in': [ 1995, 2005, 2015 ] } }
    ```
    - exclude match : $not, $eq
    ```python
    filters = { 'year': { '$in': [ 1995, 2005, 2015 ] }, 
                     'genre': { '$not' : {'$eq': 'Adult'} } }
    ```
    - [Query and Projection Operators](https://www.mongodb.com/docs/manual/reference/operator/query/?jmp=coursera-intro-mongodb)

8. The $elemMatch Operator
    - Useful when you want to filter based on a sub-document within a document
    - Sub-document has multiple fields
    - [The+$elemMatch+Operator.ipynb](./notebooks%20week2/The%2B%24elemMatch%2BOperator.ipynb)
    - [querying-for-documents-on-an-array-field.ipynb](./notebooks%20week2/querying-for-documents-on-an-array-field.ipynb)

9. Using dot(.) notation to sort and find
    - [querying-reviews-subdocument.ipynb](./notebooks%20week2/querying-for-documents-on-an-array-field.ipynb)
    - subdocuments helps us to:
        - reduce the need for database joins.
        - express queries using the fields of subdocuments.

10. insert_one method
    - insert a documnent into mongoDB collection
    - [insert-one.ipynb](./notebooks%20week2/insert-one.ipynb)
    - Insert made can validated in mongoDB before accepting. Override this using **_bypass_validation_** flag
    - If we do not provide an _id, mongoDB will provide one for us
        - If we try to insert another document with same _id, a duplicate error is triggered
    ```python
    insert_result = mflix.comments.insert_one(comment, bypass_validation)
    pprint.pprint(insert_result.acknowledged) #True if server aknowledges our insert
    pprint.pprint(insert_result.inserted_id) #returns the id of newly inserted item, if not provided explicitly then the server assigned id
    ```

11. Updating a document
    - When a new comment is added to a movie page on website we need to update the document to reflect that
    - [add-comment-to-movie](./mflix/mflix%202/mflix/db.py)
    ```python
    def add_comment_to_movie(movieid, user, comment, date):
    MOVIE_COMMENT_CACHE_LIMIT = 10

    comment_doc = {
        "name": user.name,
        "email": user.email,
        "movie_id": movieid,
        "text": comment,
        "date": date
    }

    movie = get_movie(movieid)
    if movie:
        update_doc = {
            "$inc": {
                "num_mflix_comments": 1
            },
            "$push": {
                "comments": {
                    "$each": [comment_doc],
                    "$sort": {"date": -1},
                    "$slice": MOVIE_COMMENT_CACHE_LIMIT
                }
            }
        }

        # let's set an `_id` for the comments collection document
        comment_doc["_id"] = "{0}-{1}-{2}".format(movieid, user.name, \
            date.timestamp())

        db.comments.insert_one( comment_doc )

        db.movies.update_one({"_id": ObjectId(movieid)}, update_doc)
    ```
    - Array update operators
    - Field update operators
    - More info: [Advanced Schema Design patterns](https://www.slideshare.net/mongodb/advanced-schema-design-patterns?jmp=coursera-intro-mongodb)
    - Latest 10 comments are cached inside the ***document of that movie itself***, but when we click view all comments it pulls data from another collection called ***comments***. This is a deisgn pattern followed here.
    - So the same comment is added to the cache and then to the comments collection

12. Delete data
    - delete_one
    - delete_many
    - we pass a ***filter*** to the methods for deleting documents or deleting within documents

## Week 3 - Additional MongoDB Concepts and Basic Charting

1. Indexes
    - Optimize different stages of our queries
    - See how a query runs? : _8collection.index_information()*_
    - Primary and Secondary indexes
    - Nomenclature: Index Field_1/-1_Next Field_1/-1
    - In above e.g. we are using 2 fields as index fields and 1/-1 represents if they are sorted asc or desc
    - Fields can be root level or embedded fields using dot
    - ***Index improves read and slows write***
    - Max 64 indexes
    - Some types:
        - Single field indexes
        - Compound
        - Multikey indexes
        - Text indexes (Only 1 per collection)
            - Not exact match
            - Uses text search
        - Geospatial
        - Hash
    - [Indexes Documentation](https://www.mongodb.com/docs/v6.0/indexes/)
    - [improve-query-performance.ipynb](./notebooks%20week3/improve-query-performance.ipynb)
    - [indexes-on-movies.ipynb](./notebooks%20week3/indexes-on-movies.ipynb)

2. Geospatial Queries
    - [Geospatial Query Operators](https://www.mongodb.com/docs/manual/reference/operator/query-geospatial/)
    - GeoJSON
        - GeoJSON specifies coordinates as [<longitude>, <latitude>]. This is opposite to normal convention of (latitude, longitude)
    - $geoWithin
        - Selects geometries within a bounding GeoJSON
        - radius is defined in radians
    - $nearSphere
        - Returns geospatial objects in proximity to a point on a sphere. 
        - Requires a geospatial index. 
        - $minDistance and $maxDistance are specified in meters
    - [geospatial-queries.ipynb](./notebooks%20week3/geospatial-queries.ipynb)
    - [finding-things-nearby.ipynb](./notebooks%20week3/finding-things-nearby.ipynb)
    - E.g. document
    ```json
    {
        '_id': ObjectId('59a47286cfa9a3a73e51e72c'),
        'location': 
            {
                'address': {'city': 'Bloomington',
                          'state': 'MN',
                          'street1': '340 W Market',
                          'zipcode': '55425'},
                'geo': {
                    'coordinates': [-93.24565, 44.85466], 
                    'type': 'Point'}
            },
        'theaterId': 1000
    }
    ```

3. Graphing with MongoDB
    - [Graphing+with+MongoDB.ipynb](./notebooks%20week3/Graphing%2Bwith%2BMongoDB.ipynb)
    - using matplotlib.pyplot
        - Scatter plots
        - 3D plots
        - Box (whisker) plots
