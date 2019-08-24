# twilioblog_nodejs_microservice-tutorial

url: https://www.twilio.com/blog/building-javascript-microservices-node-js

There are few things worth emphasizing about the superiority of microservices, and distributed systems generally, over monolithic architecture:

Modularity – responsibility for specific operations is assigned to separate pieces of the application
Uniformity – microservices interfaces (API endpoints) consist of a base URI identifying a data object and standard HTTP methods (GET, POST, PUT, PATCH and DELETE) used to manipulate the object
Robustness – component failures cause only the absence or reduction of a specific unit of functionality
Maintainability – system components can be modified and deployed independently
Scalability –  instances of a service can be added or removed to respond to changes in demand.
Availability – new features can be added to the system while maintaining 100% availability.
Testability – new solutions can be tested directly in the “battlefield of production” by implementing them for restricted segments of users to see how they behave in real life.





To accomplish the tasks in this post you will need the following:

Node.js and npm (The Node.js installation will also install npm.)
To learn most effectively from this post you should have the following:

Working knowledge of JavaScript and Node.js
Some exposure to the HTTP protocol
There is a companion repository for this post available on GitHub.

Create the heroes service
Go to the directory under which you’d like to create the project and create following directory and file structure:

./heroes/heroes.js
./heroes/img/
If you’d like to use source code control, this would be a good time to initialize a repository. Don’t forget to add a .gitignore file like this one if you’re using Git.

Initialize the npm project inside ./heroes directory and install necessary dependencies by executing the following command instructions:

npm init -y
npm install express body-parser
It’s time to implement the service. Copy this JavaScript code to the ./heroes/heroes.js file:

const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');

const port = process.argv.slice(2)[0];
const app = express();
app.use(bodyParser.json());

const powers = [
  { id: 1, name: 'flying' },
  { id: 2, name: 'teleporting' },
  { id: 3, name: 'super strength' },
  { id: 4, name: 'clairvoyance'},
  { id: 5, name: 'mind reading' }
];

const heroes = [
  {
      id: 1,
      type: 'spider-dog',
      displayName: 'Cooper',
      powers: [1, 4],
      img: 'cooper.jpg',
      busy: false
  },
  {
      id: 2,
      type: 'flying-dogs',
      displayName: 'Jack & Buddy',
      powers: [2, 5],
      img: 'jack_buddy.jpg',
      busy: false
  },
  {
      id: 3,
      type: 'dark-light-side',
      displayName: 'Max & Charlie',
      powers: [3, 2],
      img: 'max_charlie.jpg',
      busy: false
  },
  {
      id: 4,
      type: 'captain-dog',
      displayName: 'Rocky',
      powers: [1, 5],
      img: 'rocky.jpg',
      busy: false
  }
];

app.get('/heroes', (req, res) => {
  console.log('Returning heroes list');
  res.send(heroes);
});

app.get('/powers', (req, res) => {
  console.log('Returning powers list');
  res.send(powers);
});

app.post('/hero/**', (req, res) => {
  const heroId = parseInt(req.params[0]);
  const foundHero = heroes.find(subject => subject.id === heroId);

  if (foundHero) {
      for (let attribute in foundHero) {
          if (req.body[attribute]) {
              foundHero[attribute] = req.body[attribute];
              console.log(`Set ${attribute} to ${req.body[attribute]} in hero: ${heroId}`);
          }
      }
      res.status(202).header({Location: `http://localhost:${port}/hero/${foundHero.id}`}).send(foundHero);
  } else {
      console.log(`Hero not found.`);
      res.status(404).send();
  }
});

app.use('/img', express.static(path.join(__dirname,'img')));

console.log(`Heroes service listening on port ${port}`);
app.listen(port);
Download the superhero and superhero team pictures from the following links and place them in the /heroes/img directory:

cooper.jpg
jack_buddy.jpg
max_charlie.jpg
rocky.jpg
The code for the heroes service provides:

a list of heroes super powers
a list of heroes
API endpoints to get the list of heroes, update a hero’s profile, and get a hero’s picture
Test the heroes.js service
The heroes service code shown below enables you to run service on any HTTP port you want.

const port = process.argv.slice(2)[0];
...
app.listen(port);
Ellipsis (“...”) in code represents a section redacted for brevity.

Run the service by executing the following command line instruction:

node ./heroes/heroes.js 8081
You can check to see if the service works as expected by using Postman, curl, PowerShell Invoke-WebRequest, or your browser. The curl command line instruction is:

curl -i --request GET localhost:8081/heroes
If the service is working correctly you should see results similar to the following console output from curl:

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 424
ETag: W/"1a8-BIZzoIRo/ZugcWv+LFVGSU1qIZU"
Date: Thu, 04 Apr 2019 12:07:07 GMT
Connection: keep-alive

[{"id":1,"type":"spider-dog","displayName":"Cooper","powers":[1,4],"img":"cooper.jpg","busy":false},{"id":2,"type":"flying-dogs","displayName":"Jack & Buddy","powers":[2,5],"img":"jack_buddy.jpg","busy":false},{"id":3,"type":"dark-light-side","displayName":"Max & Charlie","powers":[3,2],"img":"max_charlie.jpg","busy":false},{"id":4,"type":"captain-dog","displayName":"Rocky","powers":[1,5],"img":"rocky.jpg","busy":false}]
In Postman the body of the response should look like this:

Postman UI

If you want to catch up to this step using the code from the GitHub repository, execute the following commands in the directory where you’d like to create the project directory:

git clone https://github.com/maciejtreder/introduction-to-microservices.git
cd introduction-to-microservices/heroes
git checkout step1
npm install
Create the threats service
What’s the purpose of a superhero if there’s no peril? The microservices architecture of our application uses a separate service to represent the challenges that only a superhero can overcome. It also provides an API endpoint for matching superheros to threats.

The procedure for creating the threats service is similar to heroes service.

In the main directory of your project, create following directory and file structure:

./threats/threats.js
./threats/img/
In the ./threats directory, initialize the project and install its dependencies with the following npm command line instructions:

npm init -y
npm install express body-parser request
Don’t forget to include this project in source code control, if you’re using it.

Place this JavaScript code in the ./threats/threats.js file:

const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');
const request = require('request');

const port = process.argv.slice(2)[0];
const app = express();

app.use(bodyParser.json());

const heroesService = 'http://localhost:8081';

const threats = [
  {
      id: 1,
      displayName: 'Pisa tower is about to collapse.',
      necessaryPowers: ['flying'],
      img: 'tower.jpg',
      assignedHero: 0
  },
  {
      id: 2,
      displayName: 'Engineer is going to clean up server-room.',
      necessaryPowers: ['teleporting'],
      img: 'mess.jpg',
      assignedHero: 0
  },
  {
      id: 3,
      displayName: 'John will not understand the joke',
      necessaryPowers: ['clairvoyance'],
      img: 'joke.jpg',
      assignedHero: 0
  }
];

app.get('/threats', (req, res) => {
  console.log('Returning threats list');
  res.send(threats);
});

app.post('/assignment', (req, res) => {
  request.post({
      headers: {'content-type': 'application/json'},
      url: `${heroesService}/hero/${req.body.heroId}`,
      body: `{
          "busy": true
      }`
  }, (err, heroResponse, body) => {
      if (!err) {
          const threatId = parseInt(req.body.threatId);
          const threat = threats.find(subject => subject.id === threatId);
          threat.assignedHero = req.body.heroId;
          res.status(202).send(threat);
      } else {
          res.status(400).send({problem: `Hero Service responded with issue ${err}`});
      }
  });
});

app.use('/img', express.static(path.join(__dirname,'img')));

console.log(`Threats service listening on port ${port}`);
app.listen(port);
You can download the pictures from the following links and place in the /threats/img directory:

tower.jpg
mess.jpg
joke.png
Apart from the threats list, and basic methods like listing them, this service also has a POST method, /assignment, which attaches a hero to the given threat:

app.post('/assignment', (req, res) => {
   request.post({
       headers: {'content-type': 'application/json'},
       url: `${heroesService}/hero/${req.body.heroId}`,
       body: `{
           "busy": true
       }`
   }, (err, heroResponse, body) => {
       if (!err) {
           const threat = threats.find(subject => subject.id === req.body.threatId);
           threat.assignedHero = req.body.heroId;
           res.status(202).send(threat);
       } else {
           res.status(400).send({problem: `Hero Service responded with issue ${err}`});
       }
   });
});
Because the code implements inter-services communication, it needs to know the address of the heroes service, as shown below. If you changed the port on which the heroes service runs you’ll need to edit this line:

const heroesService = 'http://localhost:8081';
Test the threats service
If you’ve stopped the heroes service, or closed its terminal window, restart it.

Open another terminal window and start the threats service by executing the following command line instruction:

node threats/threats.js 8082
In the same way you tested the heroes service, test the threats service by executing a web request using Postman, curl, PowerShell Invoke-WebRequest, or your browser. Note that this time it’s a POST request.

The curl command line instruction is:

curl -i --request POST --header "Content-Type: application/json" --data '{"heroId": 1, "threatId": 1}' localhost:8082/assignment
If the service is working correctly you should see results similar to the following console output from curl:

HTTP/1.1 202 Accepted
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 121
ETag: W/"79-ER1WRPW1305+Eomgfjq/A/Cgkp8"
Date: Thu, 04 Apr 2019 19:32:56 GMT
Connection: keep-alive

{"id":1,"displayName":"Pisa tower is about to collapse.","necessaryPowers":["flying"],"img":"tower.jpg","assignedHero":1}
In Postman the body of the response should look like this:

Postman user interface.

In the terminal window where the heroes service is running you should see:

Heroes service listening on port 8081
Set busy to true in hero: 1
You just sent Cooper (localhost:8081/img/cooper.jpg) on a mission ...

Cooper the dog

… to fly to Pisa and save the historical monument! (localhost:8082/img/tower.jpg):

Tower of Pisa

You can use the hero and threats images to explore the functionality of the services on your own. These objects will also be part of a project in a forthcoming post that uses this project as a starting point.

If you want to catch up to this step using the code from the GitHub repository, execute the following commands in the directory where you’d like to create the project directory:

git clone https://github.com/maciejtreder/introduction-to-microservices.git
cd introduction-to-microservices/heroes
git checkout step2
npm install
cd ../threats
npm install
JavaScript Microservices with Node.js
In this post you learned how to create a basic distributed system architecture with Node.js. You saw how to delegate responsibility for different tasks to separate applications and to communicate between services. Both applications communicate with each other by exposed REST APIs. Each manipulates only the data for which it is responsible and can be maintained, extended, and deployed without involving the other service.

Additional Resources
Architectural Styles and the Design of Network-based Software Architectures, Roy Thomas Fielding, 2000 – Fielding’s doctoral dissertation describes Representational State Transfer (chapter 5) and other architectural styles.

Microservices – Although flawed, the Wikipedia article is a good starting place for finding more information about microservices architecture and implementation.

Node.js reference documentation.

Maciej Treder is a Senior Software Development Engineer at Akamai Technologies. He is also an international conference speaker and the author of @ng-toolkit, an open source toolkit for building Angular progressive web apps (PWAs), serverless apps, and Angular Universal apps. Check out the repo to learn more about the toolkit, contribute, and support the project. You can learn more about the author at https://www.maciejtreder.com. You can also contact him at: contact@maciejtreder.com or @maciejtreder on GitHub, Twitter, StackOverflow, and LinkedIn.
