---
title: Simple NodeJS API integration testing using Mocha + Chai
date: 2016-11-02 21:13:00
tags: [nodejs, javascript, mocha, chai, testing]
---

# Introduction

Integration testing is a critical part of the QA lifecycle. It can be used as a final check before any backend API deployment and give confidence to the development team that the deploy is safe.

We'll work through a simple API test written in NodeJS and use a couple of popular testing frameworks:

* [mocha](https://mochajs.org/) - JavaScript testing framework
* [chai](http://chaijs.com/) - library that provides assertion logic
* [chai-http](https://github.com/chaijs/chai-http) - extensions for making HTTP requests to APIs

For our example we'll use an existing public API - the [Yahoo Weather API](https://developer.yahoo.com/weather/). We'll build two tests, one positive and one negative.

# Create the project

Let's get the project started. Assuming you already have npm installed, in a new directory, initialize the project and install the relevant npm modules:

``` bash
$ mkdir sample-yahoo-api-tests
$ npm init
$ npm install --save-dev mocha chai chai-http
```

This creates a `package.json` file that contains the package dependencies mentioned above. Let's make a few changes to this as so (don't change the versiopn numbers after each package in the devDependencies section as they will probably be different in your case):

``` json
{
  "name": "sample-yahoo-api-tests",
  "version": "1.0.0",
  "description": "Yahoo Weather API tests",
  "main": "index.js",
  "dependencies": {
  },
  "devDependencies": {
    "chai": "^3.5.0",
    "chai-http": "^3.0.0",
    "mocha": "^3.1.2"
  },
  "author": "",
  "license": "ISC"
}
```

Mocha gives us a standard and simple format of writing tests. Generally we group tests into individual test files. Each test file then has a smaller group and then the tests within.

Each file has a describe...it structure as so:

```
describe("Some feature or API")
    it("Should or should not do something")
    it("Should or should not do something else")
```

Our general structure will be:

``` javascript
describe('Yahoo Weather API', function() {
    it("returns a result for Santa Monica, CA", function() {
        // Test code here
    });

    it("returns null results for invalid location", function() {
        // Test code here
    });
});
```

chai-http allows us to easily make requests to an HTTP endpoint and handle the response. The general format is:

``` javascript
chai.request(server)
    .get(path)
    .then(function(res) {
        // Test code here
});
```

The Yahoo endpoint we're going to test is `https://query.yahooapis.com/v1/public/yql?q=select%20item.condition%20from%20weather.forecast%20where%20woeid%20in%20(select%20woeid%20from%20geo.places(1)%20where%20text%3D%22Santa%20Monica%2C%20CA%22)&format=json&env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys` which will return us basic weather information for Santa Monica, CA:

``` json
{
 "query": {
  "count": 1,
  "created": "2016-11-03T05:18:01Z",
  "lang": "en-US",
  "results": {
   "channel": {
    "item": {
     "condition": {
      "code": "31",
      "date": "Wed, 02 Nov 2016 09:00 PM PDT",
      "temp": "70",
      "text": "Clear"
     }
    }
   }
  }
 }
}
```


# Building the first (positive) test

Putting this all together, here's our first test that we'll create in `test/yahoo-weather.spec.js`:

``` javascript
var chai = require('chai');
var chaiHttp = require('chai-http');
var expect = chai.expect;

// Tell chai to use chai-http
chai.use(chaiHttp);

describe('Yahoo Weather API', function() {

    it("returns a result for Santa Monica, CA", function() {
        return chai.request('https://query.yahooapis.com')
            .get("/v1/public/yql?env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys&format=json&q=select%20item.condition%20from%20weather.forecast%20where%20woeid%20in%20%28select%20woeid%20from%20geo.places%281%29%20where%20text%3D%27Santa%20Monica%2C%20CA%27%29")
            .then(function(res) {
                expect(res).to.have.status(200);
                expect(res.body).to.have.property('query');
                expect(res.body.query).to.have.property('results');
                expect(res.body.query.results).to.be.instanceof(Object);
        });
    });

});
```

Note that there are actually 4 assertion tests here:

``` javascript
expect(res).to.have.status(200);
```

This ensures that the HTTP response code for this request is 200, which is a general success response. If the server returns some error, we'll get a failure in our tests here.

``` javascript
expect(res.body).to.have.property('query');
expect(res.body.query).to.have.property('results');
```

Here we test that the JSON response has a query property, and inside we have a response property.

``` javascript
expect(res.body.query.results).to.be.instanceof(Object);
```

Now ensure that query.results is an Object.

# Running the first test

We can now run this test to check the results by invoking mocha:

``` 
$ node_modules/mocha/bin/mocha test/yahoo-weather.spec.js
  Yahoo Weather API
    ✓ returns a result for Santa Monica, CA (272ms)

  1 passing (272ms)
```

# Building the second (negative) test

As you can see, we now have a simple working API test! Let's go ahead and add our second test that ensure we get a null response for a city that doesn't exist:

``` javascript
var chai = require('chai');
var chaiHttp = require('chai-http');
var expect = chai.expect;

// Tell chai to use chai-http
chai.use(chaiHttp);

describe('Yahoo Weather API', function() {

    it("returns a result for Santa Monica, CA", function() {
        return chai.request('https://query.yahooapis.com')
            .get("/v1/public/yql?env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys&format=json&q=select%20item.condition%20from%20weather.forecast%20where%20woeid%20in%20%28select%20woeid%20from%20geo.places%281%29%20where%20text%3D%27Santa%20Monica%2C%20CA%27%29")
            .then(function(res) {
                expect(res).to.have.status(200);
                expect(res.body).to.have.property('query');
                expect(res.body.query).to.have.property('results');
                expect(res.body.query.results).to.be.instanceof(Object);
        });
    });

    it("returns null results for invalid location", function() {
        return chai.request('https://query.yahooapis.com')
            .get("/v1/public/yql?env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys&format=json&q=select%20item.condition%20from%20weather.forecast%20where%20woeid%20in%20%28select%20woeid%20from%20geo.places%281%29%20where%20text%3D%27AAAAAAAAAAAA%2C%20CA%27%29")
            .then(function(res) {
                expect(res).to.have.status(200);
                expect(res.body).to.have.property('query');
                expect(res.body.query).to.have.property('results');
                expect(res.body.query.results).to.be.null;
        });
    });

});
```

Our second test is similar to the first, except we're passing in a city name that we know doesn't exist ('AAAAAAAAAAAA') and expecting the results to be null. Let's run this and make sure it works:

``` 
$ node_modules/mocha/bin/mocha test/yahoo-weather.spec.js
  Yahoo Weather API
    ✓ returns a result for Santa Monica, CA (272ms)
    ✓ returns null results for invalid location (207ms)

  2 passing (488ms)
```

# Tidy up

Let's do some tidying up. Firstly, typing in the full test command is long-winded. We'll create a script in package.json to make it easier to run. Add the `scripts` section into your package.json file:

``` json
"scripts": {
  "test": "./node_modules/.bin/mocha --reporter spec test/*.spec.js"
},
```

Now, all you have to do to run the tests is:

``` bash
$ npm test
```

Much easier!

Next, let's clear up our code a little. We'll add a helper file that'll clean up the API URL that we're calling for both tests. Create `test/yahoo-helpers.js` and add this as content:

``` javascript
var queryString = require('query-string');

module.exports.getQueryForLocation = function(location) {
    return "select item.condition from weather.forecast where woeid in (select woeid from geo.places(1) where text='" + location + "')";
};

module.exports.getUrlForLocation = function(location) {
    var query = this.getQueryForLocation(location);
    var params = {
        q: query,
        format: 'json',
        env: 'store://datatables.org/alltableswithkeys'
    };
    return "/v1/public/yql?" + queryString.stringify(params);
};
```

We also need to install the query-string npm module which helps us create the full URL:

``` bash
$ npm install --save-dev query-string
```

And make changes in the test file to use these new helpers:

``` javascript
var chai = require('chai');
var chaiHttp = require('chai-http');
var expect = chai.expect;
var yahooHelpers = require('./yahoo-helpers.js');

chai.use(chaiHttp);

var apiBase = 'https://query.yahooapis.com';

describe('Yahoo Weather API', function() {

    it("returns a result for Santa Monica, CA", function() {
        return chai.request(apiBase)
            .get(yahooHelpers.getUrlForLocation('Santa Monica, CA'))
            .then(function(res) {
                expect(res).to.have.status(200);
                expect(res.body).to.have.property('query');
                expect(res.body.query).to.have.property('results');
                expect(res.body.query.results).to.be.instanceof(Object);
        });
    });

    it("returns null results for invalid location", function() {
        return chai.request(apiBase)
            .get(yahooHelpers.getUrlForLocation('AAAAAAAAAAAAAAAA'))
            .then(function(res) {
                expect(res).to.have.status(200);
                expect(res.body).to.have.property('query');
                expect(res.body.query).to.have.property('results');
                expect(res.body.query.results).to.be.null;
            });
    });

});
```

# Recap

And we're done! To recap:

* We used mocha as our basic testing framework
* We used chai to give us assertion support using the expect keyword
* We used chai-http to give us the ability to make HTTP requests
* We created two integration tests - a positive (to check a valid response) and a negative (to ensure we get a null response for a bad request)

The full code is in the [github project](https://github.com/preinvent/sample-yahoo-api-tests)