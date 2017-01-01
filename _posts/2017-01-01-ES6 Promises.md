---
layout: post
title: ES6 Promises
date: 2017-01-01
---

This tutorial is essentially meant for beginners who don't know what the hell promises are.
If you happen to already know what they are about, **please** waste these 5 minutes of your life
gaining a better understanding of this black [magic](https://en.wikipedia.org/wiki/Magic_(paranormal)).

# Introduction:

Consider the following [asynchronous](https://en.wikipedia.org/wiki/Asynchrony_(computer_programming)) Javascript function

```javascript
"use strict";

let lookFor = function(search, callback) {
	let xml = new XMLHttpRequest();
	xml.open('GET', 'https://crossorigin.me/https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch='+decodeURIComponent(search)+'&utf8=&format=json');
	
	xml.onload = function() {
		if ( xml.status === 200 )
			callback(null, JSON.parse(xml.response));
		else
			callback(Error(xml.statusText));
	}
	xml.send();
};

lookFor('google', function(err, data) {
	if ( err ) 
		; /*Recover from error*/
	
	for ( let article of data.query.search )
		console.log(article.title);
});

```

We are basically sending an request to get articles related to `search` using the wikipedia API.
Simple Huh ? Not quite. this function alse takes a `callback` argument which is called when the result it ready. But `XMLHttpRequest` is an asynchronous request, in other words, we the code we wrote will not be executed directly. instead It has to wait until we get a response from Wikipedia.

And so what ? I will use the callback, it's not really that complicated. This is really okay if you are sending a single request, or when you don't care in what order you want your callback get executed. Otherwise it gets really annoying and disappointing, that's what we call the [Callback Hell](http://callbackhell.com/).

One solution to this problem is a new feature presented in ES6 called Promise.

Basically a `Promise` a wrapper around an asynchronous request, that is not yet executed, but it expected to be executed in the near future.

## Description:

>A Promise represents a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers to an asynchronous action's eventual success value or failure reason. This lets asynchronous methods return values like synchronous methods: instead of the final value, the asynchronous method returns a promise of having a value at some point in the future.

That was a nice quote from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).
So what a `Promise`'s offer is a more reasonable way to write asynchronous code.

## Promises in Action:

So, let's rewrite the some example from before using the `Promise` object.

```javascript
"use strict";

let lookFor = function(search) {
	return new Promise((resolve, reject) => {
		let xml = new XMLHttpRequest();
		xml.open('GET', 'https://crossorigin.me/https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch='+decodeURIComponent(search)+'&utf8=&format=json');
		xml.onload = function() {
			if ( this.status === 200 )
				resolve(JSON.parse(this.response));
			else
				reject(Error(this.statusText));
		}
		xml.send();
	});
};

lookFor('google').then(results => {
	for ( let article of results.query.search )
		console.log(article.title);
}).catch(err => {
	/*Recover from error*/
});
```

So what is happening in the code snippet above is, that we are creating a new promise object, that has the ability to adjust its status.

When a promise object is created, its status is allways `pending`. However when we call `resolve` or `reject`, we can change the the status of that `Promise`.

If the Promise is `resolved`, the function that we passed to `.then` will get called and returns a new promise. if it is `rejected` then the function that we passed to `.catch` will be called. returning a new Promise object in the `.then` call essentially allows chaining. When the Promise is `pending` Javascript awaits that promise to `resolved` or `rejected`, calls `.then` or `.catch` based on the status.

### Can this get any better ?

Of course, consider the case when you want to look for multiples images, and display them one after another.
Doing this using bare `XMLHttpRequest`s is pain in the ass. so lets use promises.

```javascript
let images = ['/cat.png', '/dog.png', '/search-menu.png', '/buy-icon.png'],
    index = 0;

let onSucess = result => {
	displayImage(result);

	/*If there are still images to load*/
    if ( index <= images.length ) {
        loadImage(images[index++]).then(onSucess).catch(onError);
    } else {
        console.log('DONE !!'); /*Epilogue, if needed*/
    }
};

let onError = err => {
    /*Recover from error*/
};
loadImage(images[index++]).then(onSucess).catch(onError);
```

If you don't care about the order, but you want to show them all at the same time

```javascript
let images = ['/cat.png', '/dog.png', '/search-menu.png', '/buy-icon.png'],
	results = [],
	steps = 0;
	
let onSucess = result => {
	results.push(result);
	steps++;
	if ( steps === images.length ) {
		console.log('DONE!!');
		showAllImages(); /*Epilogue if needed*/
	}
};
	
let onError = err => {
	steps++;
	/*Recover from error*/
};
	
for ( let image of images )
	getImage(image).then(onSucess).catch(onError);
```

#### Links:

[Callback Hell](http://callbackhell.com/)<br/>
[MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)<br/>
[Promises/A+](https://promisesaplus.com/)<br/>
[Promises in Depth](https://ponyfoo.com/articles/es6-promises-in-depth)

Thanks everyone for reading this blog post. If you have liked it or learnt anything from it, **please, pleeeeease*, leave a comment, share your ideas, suggest blogposts. Cheers :D

*Happy new year <3, 2017 will be better, I Promise*.
<img src="/img/byebye.gif-c200" style="display: block; margin: 0 auto;"/>