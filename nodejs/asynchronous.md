From [https://www.codementor.io/codeforgeek/manage-async-nodejs-callback-example-code-du107q1pn](https://www.codementor.io/codeforgeek/manage-async-nodejs-callback-example-code-du107q1pn)

# Handling the Asynchronous Nature of Node.js: Sample Project

Node.js is built on top of Google's V8 engine, which in turns compiles JavaScript. As many of you already know, JavaScript is asynchronous in nature. Asynchronous is a programming pattern which provides the feature of non-blocking code i.e do not stop or do not depend on another function / process to execute a particular line of code.

Asynchronous is great in terms of performance, resource utilization and system throughput. But there are some drawbacks:

1. Very difficult for a legacy programmer to proceed with Async.
2. Handling control flow is really painful.
3. Callbacks are dirty.

If you are a function-oriented programmer, then it would be little difficult for you to grasp asynchronous programming. However, if you are familiar with multithreading in Java, then this is similar to that.

Node.js uses ECMAScript as its core language, hence it adopts every aspect of ECMAScript-like generators, fibers, asynchronous nature of code, etc. Consider following code, which is blocked in nature.

```js
var fs = require("fs");
fs.readFileSync(‘abc.txt’,function(err,data){
  if(!err) {
    console.log(data);
  }
});
console.log("something else");
```

Now consider following Asynchronous code:

```js
var fs = require("fs");
fs.readFile(‘abc.txt’,function(err,data){
    if(!err) {
     console.log(data);
    }
});
console.log("something else");
```

## Whats the Difference? {#whats-the-difference}

In the first module, the program was waiting while reading the file. It will not go further before completing the read operation, which is an example of blocking code. But ideally, we should proceed further while the program was reading the file and once it is done we should go back and process that. That is what happening in the second code.

In the second code, the program is not waiting, hence you see the console first and file contents later. However, controlling the flow of this kind of program will cost you a little more time and work. Before moving further, ask yourself: how you are going to know whether or not the file reading has been done?

There are plenty of ways to do so, but one of the popular ways is using Callback, which is like a notification system in Asynchronous programming. As the name implies, it will notify you when the Async task is completed. But wait, we have not solved the puzzle yet!

Having callbacks in your code does not mean you have control over the asynchronous nature of JavaScript. Most programmers make this mistake and get caught up in callback hell.

## Callback Hell: {#callback-hell}

![](https://process.filestackapi.com/cache=expiry:max/AnjDAmqjR538nxpWhs3Q "callback hell")

Having a series of callbacks after callbacks is known as the "callback hell" in the JavaScript zone. In this example, one callback stops the execution of code, waits for data, and then passes it to the next function, which again has callbacks and puts event loops to a suspended state until processing finishes, and so on.

It looks something like this:

```js
function readFile(filename,callback) {
  fs.readFile(filename,function(err,data)
    if(!err) {
      processData(data,function(res)) {
        printData(res);
      }
    }
  });
}
```

### Why Isn't it Good to Have Multiple Callbacks? {#why-isnt-it-good-to-have-multiple-callbacks}

When you use callbacks, you ultimately stop the execution flow of your whole program. Having fewer callbacks is not an issue, but if you have too many callbacks, your program will be in more of a waiting state than an executing state. You really don’t wanna do that, 'cause as said by Google, "Don’t present it unless it's fast".

## What’s the solution? {#whats-the-solution}

To avoid callback hell, you need to understand the design the program flow in such a way that you program utilizes resources and works fast. Node.js has plenty of modules to control the flow of a program, and one of the most popular ones is[Async.js](https://github.com/caolan/async).

Before heading to Async.js, let’s discuss a little bit about prototyping in Node.js. As you might know, object-oriented features are available in JavaScript using Prototype design pattern. The prototype design pattern is also called as "hidden class" mechanism.

In prototyping, we create classes and functions inside a class and instantiate it using objects just like we do in C++ or Java, but the biggest difference is that in JavaScript, everything is an object.

## Introduction to Async.js {#introduction-to-asyncjs}

Async is a utility module which provides straight-forward, powerful functions for working with asynchronous JavaScript. Functions are categorized in a collection, and control the flow along with utils.

```js
async.map(['file1','file2','file3'], fs.stat, function(err, results){
    // results is now an array of stats for each file
});

async.filter(['file1','file2','file3'], fs.exists, function(results){
    // results now equals an array of the existing files
});

async.parallel([
    function(){ ... },
    function(){ ... }
], callback);

async.series([
    function(){ ... },
    function(){ ... }
]);
```

Using Async.js, you can design parallel / Series / Batch flow very easily and efficiently. I will design a program to demonstrate same.

## Sample Project: {#sample-project}

We are going to design a database dumper. Database dumper is a program which ETL guys use to put lots of data in the database for application programmers. In our scene, since we don’t have any production ETL project, we will try and dump a single table that has lots of Movie data.

#### From where can we get movies' data? {#from-where-can-we-get-movies-data}

You all have heard about[OMDB](http://www.omdbapi.com/), the biggest collection of movies all time. Well, there is one service called OMDBAPI, which provides API to fetch movie information in JSON format. This is what we need.

### Control flow: {#control-flow}

Our control flow will be like this:

1. Get the movie name from the file.
2. Call OMDB to get information regarding the movie.
3. If found, dump in database.
4. If not, then go to next movie and repeat from step 1.

To do that we are going to use`Async.waterfall()`to control flow.

### Movies names? {#movies-names}

I have compiled and put the name of 3000 movies in JSON structure file for you! Yeah, you can get it from[Github](https://gist.github.com/codeforgeek/f29703bf7e7dc37183d5#file-moviename)too.

It's similar to this format:

```js
[  
  {  
    "name":"The 40-Year-Old Virgin"
  },
...
]
```

### Database design. {#database-design}

Create a database named "MoviesData", and run this query inside your SQL section.

```js
CREATE TABLE `MoviesData`.`movie` 
(
  `id` INT NOT NULL AUTO_INCREMENT ,
  `Name` VARCHAR(50) NOT NULL ,
  `ReleaseDate` VARCHAR(20) NOT NULL , 
  `Year` VARCHAR(10) NOT NULL , 
  `Cast` VARCHAR(50) NOT NULL , 
  `Plot` VARCHAR(50) NOT NULL , 
  `Genre` VARCHAR(50) NOT NULL , 
  `Rated` VARCHAR(20) NOT NULL , 
  `RunTime` VARCHAR(20) NOT NULL , 
  `Poster` VARCHAR(20) NOT NULL , 
  `Country` VARCHAR(20) NOT NULL , 
  `Language` VARCHAR(20) NOT NULL , 
  `Type` VARCHAR(20) NOT NULL ,
  PRIMARY KEY (`id`)
)
ENGINE = InnoDB;
```

Make sure this table is created.

### Package.json {#packagejson}

```js
{
  "name": "sadique",
  "version": "0.0.1",
  "dependencies": {
    "async": "~0.9.0",
    "mysql": "^2.7.0"
  }
}
```

Install dependencies used by running the

```js

npm install
```

command.

### Server.js {#serverjs}

```js
var async = require("async");
var http = require("http");
var movies = require("./inputJSON.json");
var mysql = require("mysql");
var movieArray = [];
var pool;
function processDB() {
  var self = this;
    pool = mysql.createPool({
      connectionLimit : 100,
      host          : 'localhost',
      user          : 'root',
      password : '',
      database  : 'MoviesData',
      debug       :  false
    });
    self.collectMovies();
}
processDB.prototype.collectMovies = function() {
  var self = this;
  for(var i=0;i < movies.length;i++) {
    movieArray.push(movies[i].name);
  }
  async.eachSeries(movieArray,self.processMovie,function(){
    console.log("I am done");
  });
}
processDB.prototype.processMovie = function(movieName,callback) {
  var self = this;
  async.waterfall([
  function(callback) {
    var response = "";
    movieName = movieName.split(' ').join('+');
    http.get("http://www.omdbapi.com/?t="+movieName+"&y=&plot=short&r=json",function(res){
      res.on('data',function(chunk){
      response += chunk;
      });
      res.on('end',function(){
        if(typeof response === "string") {
          response = JSON.parse(response);
    if(response.Response === 'False') {
            console.log("Movie not found");
            callback(true);
    } else {
      callback(null,response,movieName);
      }
  } else {
          callback(true);
          }
      });
      res.on('error',function(){
        console.log("Some error i think");
        callback(true);
      });
    });
  },
  function(MovieResponse,Movie,callback) {
    var SQLquery = 'INSERT into ?? (??,??,??,??,??,??,??,??,??,??,??,??) VALUES '
+ '(?,?,?,?,?,?,?,?,?,?,?,?)';
    var inserts  = ["movie","Name","ReleaseDate","Year","Cast","Plot","Genre","Rated","RunTime","Poster","Country","Language","Type",MovieResponse.Title,MovieResponse.Released,MovieResponse.Year,MovieResponse.Actors,MovieResponse.Plot,MovieResponse.Genre,MovieResponse.imdbRating,MovieResponse.Runtime,MovieResponse.Poster,MovieResponse.Country,MovieResponse.Language,MovieResponse.Type];
    SQLquery = mysql.format(SQLquery,inserts);

    pool.getConnection(function(err,connection){
      if(err) {
        self.stop(err);
        return;
      } else {
          connection.query(SQLquery,function(err,rows){
            connection.release();
            if(err) {
              console.error('error running query', err);
            } else {
             console.log("Inserted rows in DB");
            }
          });
        callback();
  }
      });
}],function(){
      callback();
});
}
processDB.prototype.stop = function(err) {
    console.log("ISSUE WITH MYSQL \n" + err);
    process.exit(1);
}
new processDB();
```

### Running the program {#running-the-program}

Let’s run the code. Open up your terminal, switch to the project directory and run

```js
node Server.js
```

to run the program.

Since we are running this in series, so for 3000 movies it will take around 7-8 minutes to dump the database.![](https://process.filestackapi.com/cache=expiry:max/oSIjf19RTE2H4PTPPBMA "dump database")

If you want to try and run this in parallel, then the OMDB api will block your IP, just to avoid a DOS attack due to getting lots of HTTP requests on a single timestamp. Here is a screenshot, I tried that![](https://process.filestackapi.com/cache=expiry:max/WPGq9JRnRNyi8uLZFJcm "OMDB api")

This is how it will work in series. It will pick the movie name from the file one at a time, and call the`processMovie`function. This function will call the OMDB API and then push it into your DB and gives a callback to the callee to provide next movie name.

The same process will get repeated till it reaches the end of the file.

In the scenario where you'd get an error such as "Movie not found in OMDB" or "response is blank or null program", the program will simply skip that movie name and will not add that into the database.

## Code explanation {#code-explanation}

To understand the code, I have divided it into modules and used prototyping to structure the program.

Our main class is`processDB()`, which is a hidden class too, in this class we will create a MySQL pool and call to next function.

In the`collectMovies()`function, we will read the name of movies from a file and store them in an array that is not in object format.

We are using`async.eachSeries()`collection and passing the whole array into it, this function is smart enough to pick one data from an array on time, and we are calling`processMovie()`function.`processMovie()`is the heart of this program. Here, since we have two operations in hand, we will use the`waterfall()`collection. What it does is it provides data to its next function which is what we need.

In`async.waterfall()`, we have two functions. The first one will call the OMDB API, while the second one will add information into database.

![](https://process.filestackapi.com/cache=expiry:max/AdbDt3YmQQqArnFq3pX8 "enter image description here")

We have used error handling scenarios in case an API call failure happens. In the SQL function, we are creating a query and calling MySQL to do that and accordingly provides the feedback.

## Conclusion {#conclusion}

Async.js is one of the popular and most widely used tools in the Node.js community. As a node.js developer, you should know how to use it, as it will surely solve most of your problems and provide you a better understanding about handling asynchronous nature.

