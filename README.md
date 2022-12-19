# Project-1 : MEAN Stack Implementation

Introduction:
MEAN stack is an example of technology stack. A technology stack is a stack of tools and frameworks that work together to provide well-functioned software.
MEAN is an abbreviation of MongoDB, Express.js, Angular.js and Node.js. These tools together form a technology stack. An implementation of this stack will be shown in this project. 

Task: A simple book register web form is implemented using MEAN stack.

# Prerequisites:

1. Launching EC2 instance:

    An EC2 instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image is created.
    ![79F9ABE8-ABEA-4486-8440-B68DC4D3224E](https://user-images.githubusercontent.com/119781770/208349341-3f8ef8e2-2557-44e5-95f9-ca148d33ed46.jpeg)

2. Setting up security group for the launched EC2 instance.
![722F12A8-74AC-4317-AA37-99AAAEFB4999](https://user-images.githubusercontent.com/119781770/208349440-b4720f11-b225-4ccd-a2bc-a0a826c76dee.jpeg)

3. SSH into this EC2 instance via the terminal using the below commands.

```
sudo chmod 0400 <private-key-name>.pem
```

This command is used to change file permissions.

Connect to the instance by running

```
ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>
```

# Step 1 : Install Node.js

Before installation of Node.js, Ubuntu is updated and upgraded.



```
sudo apt update
```

![E6CED2C3-9B66-4A81-82A2-DD56E5DB32F7](https://user-images.githubusercontent.com/119781770/208350359-18dddabd-51f8-4c77-be11-89b2f02755a3.jpeg)



    ```
    sudo apt upgrade
    ```

![890613CE-6850-41B3-9FD0-86D5909E0A71](https://user-images.githubusercontent.com/119781770/208350376-f175d0d2-e5c6-4f5c-8886-9c78905c7e82.jpeg)

Next, certificates are installed using the below command followed by installation of Node.js.

      ```
      sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

      curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
      ```


Install NodeJS

      ```
      sudo apt install -y nodejs
      ```
![9C09AD0F-61AD-421B-BADF-83F741DD158E](https://user-images.githubusercontent.com/119781770/208350791-062616d5-d37f-4191-8c66-10b6dbfd9ed3.jpeg)


# Step 2 : Install MongoDB

MongoDB is installaed using the command below. We then start MongoDB and check the status of the server.

      ````
      sudo apt install -y mongodb
      ```

      ```
      sudo service mongodb start
      ```

      ```
      sudo systemctl status mongodb
      ```

![2F2372D1-C13A-44C4-9E2D-2BBE7CBEF22E](https://user-images.githubusercontent.com/119781770/208352300-421156a2-0205-4321-9d6d-98cee0e35c14.jpeg)

We then install node package manager (NPM) followed by body-parser as shown below:


      ```
      sudo apt install -y npm
      ```

We need ‘body-parser’ package to help us process JSON files passed in requests to the server.


      ```
      sudo npm install body-parser
      mkdir Books && cd Books
      ```



Then, a folder named Books is created. This folder is going to be our project folder.





![1C2635D7-3399-4A2E-B3EE-226DC6B79219](https://user-images.githubusercontent.com/119781770/208352965-18e94d83-9e3f-4f37-afc9-3b52ee5bbbc7.jpeg)


In the books directory, npm project is initialized .



      ```
      npm init
      ```


Add a file to it named server.js



      ```
      vi server.js
      ```

Copy and paste the web server code below into the server.js file.


      ```
      var express = require('express');
      var bodyParser = require('body-parser');
      var app = express();
      app.use(express.static(__dirname + '/public'));
      app.use(bodyParser.json());
      require('./apps/routes')(app);
      app.set('port', 3300);
      app.listen(app.get('port'), function() {
          console.log('Server up: http://localhost:' + app.get('port'));
      });

      ```





![D459ED20-49D9-4112-BC20-C1D5453E553D](https://user-images.githubusercontent.com/119781770/208353510-23649f5c-9aba-41ce-b7d9-86abd532fe5e.jpeg)


# Step 3 : Install Express and setup routes to the server

Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications.
We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. 
We will use Mongoose to establish a schema for the database to store data of our book register.

    ```
    sudo npm install express mongoose
    ```
![C00762CE-8F0C-4CD3-8DBB-2A7180C12E81](https://user-images.githubusercontent.com/119781770/208354695-c24be835-2d80-4ddf-a68a-dab86f8a25e7.jpeg)


In ‘Books’ folder, create a folder named apps

      ```
      mkdir apps && cd apps
      ```

Create a file named routes.js

      ```
      vi routes.js
      ```

Copy and paste the code below into routes.js


        ```
        var Book = require('./models/book');
        module.exports = function(app) {
          app.get('/book', function(req, res) {
            Book.find({}, function(err, result) {
              if ( err ) throw err;
              res.json(result);
            });
          }); 
          app.post('/book', function(req, res) {
            var book = new Book( {
              name:req.body.name,
              isbn:req.body.isbn,
              author:req.body.author,
              pages:req.body.pages
            });
            book.save(function(err, result) {
              if ( err ) throw err;
              res.json( {
                message:"Successfully added book",
                book:result
              });
            });
          });
          app.delete("/book/:isbn", function(req, res) {
            Book.findOneAndRemove(req.query, function(err, result) {
              if ( err ) throw err;
              res.json( {
                message: "Successfully deleted the book",
                book: result
              });
            });
          });
          var path = require('path');
          app.get('*', function(req, res) {
            res.sendfile(path.join(__dirname + '/public', 'index.html'));
          });
        };

        ```
In the ‘apps’ folder, create a folder named models

      ```
      mkdir models && cd models
      ```

Create a file named book.js

      ```
      vi book.js
      ```

Copy and paste the code below into ‘book.js’


      ```
      var mongoose = require('mongoose');
      var dbHost = 'mongodb://localhost:27017/test';
      mongoose.connect(dbHost);
      mongoose.connection;
      mongoose.set('debug', true);
      var bookSchema = mongoose.Schema( {
        name: String,
        isbn: {type: String, index: true},
        author: String,
        pages: Number
      });
      var Book = mongoose.model('Book', bookSchema);
      module.exports = mongoose.model('Book', bookSchema);

      ```

# Step 4 – Access the routes with AngularJS

AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express 
and perform actions on our book register.

Change the directory back to ‘Books’

      ```
      cd ../..
      ```
      
Create a folder named public

```
mkdir public && cd public
```

Add a file named script.js


    ```
    vi script.js
    ```

Copy and paste the Code below (controller configuration defined) into the script.js file.

      ```
      var app = angular.module('myApp', []);
      app.controller('myCtrl', function($scope, $http) {
        $http( {
          method: 'GET',
          url: '/book'
        }).then(function successCallback(response) {
          $scope.books = response.data;
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
        $scope.del_book = function(book) {
          $http( {
            method: 'DELETE',
            url: '/book/:isbn',
            params: {'isbn': book.isbn}
          }).then(function successCallback(response) {
            console.log(response);
          }, function errorCallback(response) {
            console.log('Error: ' + response);
          });
        };
        $scope.add_book = function() {
          var body = '{ "name": "' + $scope.Name + 
          '", "isbn": "' + $scope.Isbn +
          '", "author": "' + $scope.Author + 
          '", "pages": "' + $scope.Pages + '" }';
          $http({
            method: 'POST',
            url: '/book',
            data: body
          }).then(function successCallback(response) {
            console.log(response);
          }, function errorCallback(response) {
            console.log('Error: ' + response);
          });
        };
      });

      ```

In public folder, create a file named index.html;

      ```
      vi index.html
      ```

Copy and paste the code below into index.html file.


        ```
        <!doctype html>
        <html ng-app="myApp" ng-controller="myCtrl">
          <head>
            <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js">               </script>
            <script src="script.js"></script>
          </head>
          <body>
            <div>
              <table>
                <tr>
                  <td>Name:</td>
                  <td><input type="text" ng-model="Name"></td>
                </tr>
                <tr>
                  <td>Isbn:</td>
                  <td><input type="text" ng-model="Isbn"></td>
                </tr>
                <tr>
                  <td>Author:</td>
                  <td><input type="text" ng-model="Author"></td>
                </tr>
                <tr>
                  <td>Pages:</td>
                  <td><input type="number" ng-model="Pages"></td>
                </tr>
              </table>
              <button ng-click="add_book()">Add</button>
            </div>
            <hr>
            <div>
              <table>
                <tr>
                  <th>Name</th>
                  <th>Isbn</th>
                  <th>Author</th>
                  <th>Pages</th>

                </tr>
                <tr ng-repeat="book in books">
                  <td>{{book.name}}</td>
                  <td>{{book.isbn}}</td>
                  <td>{{book.author}}</td>
                  <td>{{book.pages}}</td>

                  <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
                </tr>
              </table>
            </div>
          </body>
        </html>

        ```

Change the directory back up to Books


      ```
      cd ..
      ```

Start the server by running this command:


      ```
      node server.js
      ```
![56784381-FD63-49B1-815E-69D24E5FCA85](https://user-images.githubusercontent.com/119781770/208356159-b38e6c24-a89c-41e8-ad15-e640991285cf.jpeg)


The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.

      ```
      curl -s http://localhost:3300
      ```

![5F04C576-0118-47FB-90C6-755544A2BFF0](https://user-images.githubusercontent.com/119781770/208356169-debdd888-c491-4663-b5ad-5d22626929cd.jpeg)

Opening the application on a web browser:

![78F48F35-62B7-467A-9999-CE03EE8C2DEC](https://user-images.githubusercontent.com/119781770/208356181-d590e03c-ba92-4863-9020-2250665a6a5e.jpeg)

-------END----------
