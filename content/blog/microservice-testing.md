+++
date = "2016-09-25T17:00:00"
draft = false
tags = ["microservice", "testing", "apiary"]
title = "Microservice testing with Apiary"
math = false
+++

## Introduction
I assume you are generally familiar with microservices, otherwise I can recommend [this](http://martinfowler.com/articles/microservices.html) article for further reading. Lets also assume we have a basic microservice architecture set up. So, why and how do we test it? Why: I won't even start preaching about tests, just write a bigger project without testing. I guess every programmer has to do that at some point of his life. How: Most of the tests, and for some services all of the tests will be very straight-forward. On method-level we can apply unit-tests and for API-routes we can use Behavior-Tests. If you are using node.js, I can recommend [mocha](https://mochajs.org/) in combination with [chai](http://chaijs.com/) (I am more of the tea person...). The problem are integration tests.

## Depending microservices
In a normal web-app, there is at max one microservice which can be tested standalone and that's the user management. Even at that point you will typically depend on some kind of database, and other microservices will have other dependencies within the application. So, what are ideas to test a dependant service

* **Ignore dependencies** Without having other microservices available, you will still be able to test quite a big portion of your code. Especially all the unit-tests will run, and there might aswell be a couple of requests that you can blackbox-test without having other services interfer. Typically public GETs are a good example of requests, that you can test without the need of other services. If you can run stuff without dependencies, don't pull in further dependencies and stick to the barebones version. This will also enforce more separation and independence of your service, allowing for better failure tolerance and easier scaling. (That's why we're building microservices, right?)

* **Full testing-environment** The optimal case is if you can provide a full system, with all services running in a testing environment, to get "real" replies to your requests. This will allow for most realistic testings, but has the major drawback of feasibility. It's not easily possible to set up a full architecture, possibly consisting of dozens of different service tiers, just to test basic requests. This setup will be necessary to run integration tests on the whole system, but for lower-level tests it would be nice to have something lightweigt, which might aswell integrate into common CI-frameworks

* **Mock services** What I found would be the best way to test is using a mock-server. We anyways drafted out APIs with [Apiary](https://apiary.io), so why not using this? Apiary offers the nice feature of an automatically set up mock-server, which will respond to requests with the data you have defined in the API-blueprint-markdown. Though that interface will of course not behave as the original service, it is at least a nice intermediate step towards a full testing environment.

## Injecting new configs in JS
This example uses node.js and a configuration read from a config.json. I assume you are doing [service discovery](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/) by DNS, it doesn't matter if you plug a loadbalancer to the dns address or if you just write the other service's ip to your /etc/hosts-file. The idea is to overwrite this address in case we are in a testing environment
```javascript
{
	"dev": {
		"user-management": "http://some-dns-name",
		...
	},

	"test": {
		"user-management": "http://yourproject.apiary-mock.com",
		...
	}
}
```

Now you will have to read this out, just, wherever you used to include config.json switch to a parseconfig.js including this snippet:
```javascript
var exp = config.dev;
if(process.env.NODE_ENV == 'test') {
	for (var attr in config.test) {
		exp[attr] = config.test[attr];
	}
}

module.exports = exp;
```

Now, if your service was started with the environment-variable test set, you will load the test configuration, otherwise you will go for the usual dev-configuration. You can even extend this to switch between development and production configuration. Setting that environment-variable is easily possible by using
```javascript
process.env.NODE_ENV = 'test';
```
in the first line of every test.

Now you can test against a simplified mock of the other services. The good thing is that this mock will always reply with the same data, so you can hardcode these assumptions into your code (and e.g. assume the logged-in user is always named Günther, and thus check if the post you just created in the test holds the author Günther (I recommend to always go for Günther as a testing-user, he is very experienced))

## Dynamic behavior
The problem with this approach is of course the flexibility of the mock-server. It is possible to draft different responses based on different requests in Apiary-blueprint, however it gets difficult if you want to differentiate based on headers. One Cookie/Auth-Token/Whatever-you-use-for-auth may be valid, for the other one you get a 403. I must admit that I also didn't yet find the holy grail of almost-integration-testing here, but I will update this post as soon as I find something. If you really need it, just set up a full testing-environment.