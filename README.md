MongoLiteDB
===========

[![Build Status](https://travis-ci.org/hamiltop/MongoLiteDB.png)](https://travis-ci.org/hamiltop/MongoLiteDB)

An embeddable file based Mongo compatible database. Think SQLite for NoSQL.

Motive
------

NoSQL is a complicated and touchy subject these days. Some people love it, some people hate it. Some proponents say it gives them flexibility in how they structure their data so they can focus on application logic rather than how data is stored. Opponents say that "NoSQL is for people who don't understand relational databases" (that's a quote, source provided upon request).

Every flavor of database software has its strengths and weaknesses. The strengths are the result of optimization, the weaknesses the result of informed decisions (at least, I hope that's how it works). MongoLiteDB is no different. The things we want to optimize are ease of use and well... ease of use.

Ease of Use
-----------

The project came about via a random weekend hack. I wanted to play around with sinatra and datamapper and realized that I either had to a) use SQLite and deal with schema migrations or b) run a MongoDB server. Since my data format was going to be constantly in flux, I didn't want to deal with schema migrations. And having to run an external service (as well as install mongodb on my laptop) was a pain. I expected less than 100 db entries would be needed for my application. I thought to my self, "Is there a database that neither requires a schema nor an external service?" I couldn't find one. So I built this instead. It has the ease of setup that SQLite provides along with the ease of use that MongoDB provides. Like SQLite, it uses only a single file to store all data. Like MongoDB, no schema is required. Anything that is JSONable can be stored.

Performance
-----------

Performance sucks. As of version 0.0.2:

![Insertion Benchmarks](https://docs.google.com/spreadsheet/oimg?key=0AqV2RNwagAQrdERYOFh5NWcwWEZPcGFETmRLWnRNbUE&oid=2&zx=hwquzhzg2did)

![Read Benchmarks](https://docs.google.com/spreadsheet/oimg?key=0AqV2RNwagAQrdERYOFh5NWcwWEZPcGFETmRLWnRNbUE&oid=3&zx=arm3yh2yayv3)

I'm literally dumping one big json object to a file. To perform queries, I read it in and iterate over every document that is nested inside it. Performance is therefore O(n) where n is the number of documents in the database. But I don't really care. For my use case, performance is overrated. It should be more than adequate for databases with less than 1000 entries.

Usage
-----

It was designed to be query compatible with MongoDB. I haven't implemented everything, but quite a bit is done. An example of usage is as follows:

```ruby
require 'mongolitedb'
require 'pp'

filename = "demo.mglite"
db = MongoLiteDB.new filename

db.insert({
    "first_name" => "Joan",
    "last_name" => "Of Arc",
    "age" => 15,
    "armor_size" => "small"
})
db.insert({
    "first_name" => "Joan",
    "last_name" => "From Madmen",
    "age" => 30,
    "lipstick_color" => "red"
})
db.insert({
    "first_name" => "Joan",
    "last_name" => "Uh (as in Jonah)",
    "age" => 90,
    "armor_size" => "medium",
    "greatest_fear" => "whales"
})
puts "First Query"
pp db.find({"first_name" => "Joan"})
puts "Second Query"
pp db.find({"age" => { "$lt" => 25 } })
puts "Third Query"
pp db.find({"armor_size" => { "$exists" => true } })
```

which would output:

```text
First Query
[{"first_name"=>"Joan",
  "last_name"=>"Of Arc",
  "age"=>15,
  "armor_size"=>"small",
  "id"=>0},
 {"first_name"=>"Joan",
  "last_name"=>"From Madmen",
  "age"=>30,
  "lipstick_color"=>"red",
  "id"=>1},
 {"first_name"=>"Joan",
  "last_name"=>"Uh (as in Jonah)",
  "age"=>90,
  "armor_size"=>"medium",
  "greatest_fear"=>"whales",
  "id"=>2}]
Second Query
[{"first_name"=>"Joan",
  "last_name"=>"Of Arc",
  "age"=>15,
  "armor_size"=>"small",
  "id"=>0}]
Third Query
[{"first_name"=>"Joan",
  "last_name"=>"Of Arc",
  "age"=>15,
  "armor_size"=>"small",
  "id"=>0},
 {"first_name"=>"Joan",
  "last_name"=>"Uh (as in Jonah)",
  "age"=>90,
  "armor_size"=>"medium",
  "greatest_fear"=>"whales",
  "id"=>2}]
```

TODO
----

Documentation would be good. I haven't used it very extensively yet, so the next goal is to work out the bugs I must have created.

Supported Query Syntax
----------------------

Look at [MongoDB Query Language](http://docs.mongodb.org/manual/reference/operators/ "MongoDB Query Language"). Not everything is supported yet. For now I'll document the implemented features by pasting the Rspec -f doc output

```
rspec -f doc spec/mongolitedb_spec.rb

MongoLiteDB
  should initialize db file
  when using a single object
    should allow insertion
    should allow retrieval
    should allow update
    should allow deletion
  insert
    should add an id to record
    should accept duplicate entries
    should autoincrement id on each insert
    should make a copy of object on insert
    should allow batch inserts
  find
    should support querying nested documents
    should return empty set if nested documents do not exist
    should support multiple keyword conditions
    $or operator
      should return entries that match either condition
      should allow multiple conditions that are anded together
      should allow nested $or
    $nor operator
      should return entries that do not match any of the conditions
    $and operator
      should return entries that match all of the conditions
    $in operator
      should return entries that match any of the values
    $nin operator
      should return entries that do not match given values
    $exists operator
      should return entries that have the field defined when true
      should return entries that do not have the field defined when false
    numerical operations
      $gt
        should return entries where field is greater than the value
      $gte
        should return entries where field is greater than or equal to the value
      $lt
        should return entries where field is less than the value
      $lte
        should return entries where field is less than or equal to the value
      $ne
        should return entries where field is not equal to the value
      $mod
        should entries where the field value divided by the divisor has the specified remainder
```
