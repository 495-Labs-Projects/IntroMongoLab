# Introduction to MongoDB Lab
An introduction to using MongoDB, for CMU 67-392.

## Part 0: Documentation
[MongoDB](https://docs.mongodb.com/) (Mainly the MongoDB Server, Version 3.4)

## Part 1: Setup
1. If you haven't already, install MongoDB on your machine. Instructions for your operating system can be found at http://docs.mongodb.org/manual/installation/. Note: if you are on a Mac, I strongly recommend using homebrew package manager (http://brew.sh/) for installing MongoDB (and lots of other packages as well).

2. If you want your command line output formatted automatically as shown in class, install the .mongorc.js file found in [Prof. H's dotfiles](https://github.com/profh/dotfiles/). This is not required, but makes it easier to read the output from MongoDB. There is a free GUI for MongoDB called [Robo 3T](https://robomongo.org/) which combines an interactive shell with graphical components of the database for a more visual experience, but using the vanilla terminal version of MongoDB is recommended.

3. Clone the [starter code](https://github.com/profh/mongo_database_examples) for this lab, which includes some ruby, javascript, and SQL files. After cloning, open the file 'mongo_hero1.js' and review its contents. Note how this file is essentially a script performing a bunch of insert queries of the following form:

  ```javascript
  db.heroes.insert({
    name: <a name string>,
    power: <a superpower string or array>,
    gender: <a gender identification>
  });
  ```

4. Now that we have installed MongoDB and cloned the necessary files for the project, open up the terminal and start a local MongoDB server by executing `mongod` from the command line. If this is giving some startup issues, `sudo mongod` might do the trick (on Mac/Linux) or using the command line as administrator might help (on Windows).

5. Open a new terminal tab or window. Here, navigate to where you cloned the starter code from step 3 and run the following commands from the terminal:

  ```
  mongo mongo_hero1 mongo_hero1.js
  mongo mongo_hero2 mongo_hero2.js
  ```
Running these two commands will set up two databases `mongo_hero1` and `mongo_hero2` within MongoDB (since your server is running) and insert several documents into each.

6. Now, in the terminal, open the mongo shell by running `mongo`. You know you will be in the mongo shell if you see a `>` prompt in your terminal.

Here, we can view all of the databases in our system by running the `show_dbs` command from within the mongo shell. You should see a few databases available, including `mongo_hero1` and `mongo_hero2`.

Switch to the `mongo_hero1` database by running `use mongo_hero1` from within the mongo shell.

From here, you can view the collections from within this database by running `show collections`. You should see a collection, `heroes`! View everything in this collection by running:

  ```
  db.heroes.find({});
  ```

## Part 2: The .find() query

Now we will spend some time exploring basic commands in MongoDB, including looking into the Aggregate Pipeline framework introduced in MongoDB 3.0 in a bit.

1. Counting

When trying to count documents in a MongoDB query, the `.count()` method is useful. It can be attached to the end of a `.find()` query to simply return the total number of documents found.

Try using the following command to find the total number of heroes (assuming you are using the `mongo_hero1` database):

  ```javascript
  db.heroes.find({}).count();
  ```

Hopefully, you see that 14 heroes were returned to us. 

2. Filtering

We can take this further by applying a filter to our `.find()` query to restrict the superpowers we are looking for to only be heroes who fly:

  ```javascript
  db.heroes.find({ power: 'flies' }); 
  ```

Note we should have four heroes here.

3. Projections

In the filtering query, we got a lot of information back about each hero, including an `_id`, `name`, `power`, and `gender`. However, we only really care about the names of the superheroes. We can format the data we get back from the query using what is called as a **projection** through the following query

  ```javascript
  db.heroes.find({ power: 'flies' }, { name: 1 });
  ```

Note we pass another parameter to the `.find()` query saying we would like the name value back (a value of 1 means we wish to include this field in our results, setting all other values to be excluded otherwise). How could we modify the above query to give us back values for name and gender? How can we modify the above query to give us back everything but name?

4. Sorting

Okay so we are able to get back all of the documents by just their names, but there is a problem. This data is not sorted!

Like the SQL command of `ORDER BY`, there is an equivalent command of `.sort()` in the MongoDB Query Language. Try using the command below to sort the heroes who fly based on their name:

  ```javascript
  db.heroes.find({ power: 'flies' }, { name: 1 }).sort({ name: 1 });
  ```

Here we sort the heroes based on their names in ascending order. How would we sort the heroes based on their names in descending order? (HINT: it is not the same as projecting name out of the results of a query).

5. Regular Expressions

When we were searching for heroes who fly, we were searching for heroes who had a superpower that exactly matched "flies" in their array of superpowers. What if we don't know exactly what they call their superpower but we have some clue?

The Tick, in case you were unaware, is "nigh-{something}". We can find The Tick's full superpower by using a regular expression to search for powers:

  ```javascript
  db.heroes.find({ power: /nigh/ });
  ``` 

Ah, now we have found The Tick to be "nigh-invulnerable"!

## Part 3: Null-filtering and .update()

Now we will inspect the genders of the heroes we have entered into our database.

1. Run the following MongoDB queries:

  ```javascript
  db.heroes.find({ gender: "m" }).count();
  db.heroes.find({ gender: "f" }).count();
  db.heroes.find({ gender: "m" }).count() + db.heroes.find({ gender: "f" }).count();
  ```

Hmm, from before we see that we have a total of 14 heroes, but right now we see 10 male heroes and 3 female heroes for a combined total of 13. Who is missing?

2. In order to figure out who is missing, we should check to see if there is any hero with no gender. With the MongoDB Query Language, we can use `$exists` syntax to see if a specific field is left out of a document (and let's also project out everything but the name of the hero that should be returned). Remember, with MongoDB instead of having NULL values for fields, the field simply may not exist because of NoSQL docment design principles.

  ```javascript
  db.heroes.find({ gender: { $exists: false } }, { name: 1, _id: 0});
  ```

3. Ah, so we find Nightcrawler does not have a gender in the system, but we know Nightcrawler identifies as being male, so we should update our hero in the database to reflect this gender. We can do this by using MongoDB's `.update()` query with a filter and command operation:

  ```javascript
  db.heroes.update({ name: "Nightcrawler" }, { $set: { gender: "m" } });
  ```

Run this query and ensure that Nightcrawler is now identified as male in our database.

## Part 4: Embedded Documents

Now we have practices many of the basics of the MongoDB Query Language, so **switch to the `mongo_hero2` database** (using the command `use mongo_hero2`) for a more interesting take on heroes.

By running `show collections`, we can see we have a `superheroes` collection, so be sure to understand the `heroes` collection does not exist in this database!

1. Examine the general structure of a superhero by running the following command to get just one superhero to look at:

  ```javascript
  db.superheroes.findOne({});
  ```

2. Now we see the model of the superhero to follow something close to this structure:

  ```json
  {
    "_id": "ObjectId",
    "name": "String",
    "power": "String",
    "gender": "String",
    "people": [
      {
        "first_name": "String",
        "last_name": "String",
        "height": "Number",
        "weight": "Number"
      }
    ]
  }
  ```

Here we see it is possible for a superhero to be identified as many people, giving us an array of people objects for such field. An example of a superhero with multiple identities is Green Lantern. Use `.find()` to verify multiple identities with Green Lantern.

3. What if we wished to filter for individual people who could identify as a specific superhero? Find all the "Peter"s who identify as a superhero by running the following command (projecting only the superhero's name and the name of the people who identify as such a hero):

  ```javscript
  db.superheroes.find( { "people.first_name" : "Peter" },{name: 1, "people.first_name": 1, "people.last_name": 1, _id: 0 } ).sort( {name: 1} );
  ```

Note that with embedded documents (in this case the embedded `people` document), we must surround our field name with quotation marks because of the dot syntax used to dig into the embedded document.

4. One last example with embedded documents, how can we find superheroes with some identity that has a weight of at least 200 pounds? Run the query below, noting the `$gte` operator checks to see if the weight field is at least 200:

  ```javascript
  db.superheroes.find({"people.weight": { $gte: 200 }},{name: 1, "people.weight": 1, _id: 0}).sort({"people.weight": -1});
  ```

## Part 5: The Aggregate Pipeline

TBD

## Part 6: SQL to MongoDB QL

There is a nice "Rosetta Stone" available online at http://docs.mongodb.org/manual/reference/sql-comparison/ that helps map typical SQL concepts and commands into MongoDB. Using that reference and what you've learned here and the mongo_hero2 db, rewrite some queries we've done before in terms of MongoDB commands:

```SQL
1. SELECT first_name, last_name FROM people WHERE last_name SIMILAR TO '(G|J)%';
2. SELECT name, power FROM heroes WHERE power ~* 'human';
3. SELECT COUNT(*) FROM heroes WHERE name ~* '^s' AND gender = 'f';
4. SELECT name, height FROM people p JOIN identities i USING (person_id) JOIN heroes USING (hero_id) WHERE height BETWEEN 68 AND 71;
5. SELECT * FROM people p JOIN identities i USING (person_id) JOIN heroes USING (hero_id) WHERE is_male = true AND (height > 72 OR weight < 180);
6. SELECT last_name, first_name FROM people p JOIN identities i USING (person_id) JOIN heroes USING (hero_id) WHERE name = 'Green Lantern';
```

Put these Mongo queries into a file called [your_andrew_id]-mongo.js and send to the Head TA when complete.
