# Introduction to MongoDB Lab
An introduction to using MongoDB, for CMU 67-392.


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

From here, you can view the collections from within this database by running `show collections`. You should see a collection, `heroes`! View everything in this collection by running

	```javascript
	db.heroes.find({});
	```
