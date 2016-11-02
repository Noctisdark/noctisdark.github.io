---
layout: post
title: Animation
date: 2016-10-30
---

This tutorial will try to explain in a simplified manner how to implement 2D animation in the Web.
I'm going to use the [Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) API to
keep things simple. However you can use any rendering API. So open up your favorite [Text Editor](https://en.wikipedia.org/wiki/Text_editor),
prepare your coffee and get ready for this adventure.

# Computer Animation:

### Abstract:

&emsp;computer animation is the process used to create animated images, as opposed to static images
that stay the same all the time, like your favorite game's background.
Computer animation is essentially the digital successor to the stop motion techniques that attempts to
trick your brains into thinking that they are seeing a smoothly moving object when they are in fact 
seeing different states of motion of that object. This is possible thanks to a phenomenon called the
[Persistence of Vision](https://en.wikipedia.org/wiki/Persistence_of_vision).
The pictures should be draw at around 12 frames per seconds and usually faster, otherwise most people
will detect some sort of jerkiness associated with the drawing of new images that detracts from the illusion
of realistic movement. Lots of conventions are used when it comes to this, For example hand-drawn cartoon animation
often use 15 frame/s whereas films seen in theaters run at 24 frame/s.
There are different techniques used to achieve computer animations such as [Keyframing](https://en.wikipedia.org/wiki/Key_frame#Animation_by_means_of_computer_graphics),
[Blocking](https://en.wikipedia.org/wiki/Blocking_(animation)), [Morphing](https://en.wikipedia.org/wiki/Morphing), etc... .

### Basic Animation Steps:

&emsp;Implementing 2D computer animation is fairly simple and straightforward, these are the minimal steps you
should take to draw every frame.

- Clear the canvas:<br/>
Each frame a completely different frame from the last one at the pixel level, so you'll have to clear all previously
drawn object and prepare for drawing the next frame. the easiest way to this is the use the `clearRect` method.

- Save the canvas state:<br/>
To draw anthing on the screen you might will often need to change the canvas state. For example, you need translate the canvas a bit to make
your character seem to move and them go back to the original state. You can do this using the `save` method in canvas to save the current transformations associated
with the rendering context.

- Do all your rendering logic here:<br/>
This is where you will do all your image processing and rendering here.

- Restore your canvas state:<br/>
If you have saved your last canvas state, you can restore it but calling the `restore` method, which will pop off the last state frame and
set it as the current one.

# Code:

### Setting up our rendering context:	
&emsp;Let's try to make a simple moving character. I used a free spritesheet for animation, [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
method for animation which tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. this
will improve your application's performance and make your animation smoothier. I will be using ES6 for scripting, so your browser might not support any of this code, but this will only be a problem
for those who use older browser.

Let's start off by setting up a view layer where we will draw all of our graphics.

```html
<canvas id="view">
</canvas>
```

```javascript
let canvas  = document.getElementById('view'); //get a reference to our canvas DOM element
let ctx = canvas.getContext('2d'); //create a 2D context for drawing.

canvas.width = 640;
canvas.height = 480; //640x480 resolution

let loader = {
	cache: {}, //mapping from image url to their promises
	loadImage(url) {
		let img = this.cache[url];
		if ( img )
			return img;

		img = this.cache[url] = new Promise((resolve, reject) => {
			let image = new Image();
			image.src = url;
			image.onload = () => resolve(image);
			image.onerror = (err) => reject(err);
		});
		
		return img;
	},
	await()	 {
		let promises = [];
		for ( let url in this.cache )
			promises.push(this.cache[url]);

		return Promise.all(promises);
	}
};
```

Up until now, we're not doing anything interesting, just getting a reference to a 2D content for drawing
on the canvas and setting its resolution to 640x480 pixel.
Then creating a loader object that will help us load all of our images and await for them asynchronously
before starting the game. At this point, if you try this, you will see nothing and this is expected.

### Loading our first image:

This will be pretty simple because we have all we need to start rendering our first image on the screen.
It's as simple as calling `putImage(image, x, y, width, height)`.

```javascript
let Sprite = function(x, y, width, height, url) {
	this.position = {x, y};
	this.width = width; //the width we want to show
	this.height = height; //the height we want to show
	this.texture = null;

	loader.loadImage(url).then(image => {
		this.texture = image;
	});
};

Sprite.prototype.draw = function() {
	ctx.drawImage(this.texture, this.position.x, this.position.y, this.width, this.height);
};

let sheet = new Sprite(0, 0, 640, 480, './run.png'); //our sprite sheet, image located at './run.png'

function animate() {
	window.requestAnimationFrame(animate); //request for the next frame of this animation

	ctx.clearRect(0, 0, 640, 480); //clear the canvas
	sheet.draw(); //draw out sprite sheet
};

loader.await().then(all => {
	window.requestAnimationFrame(animate); //start the animation when everything is loaded
});
```

We are making a nice `Sprite` class which will hold all the information we need to show our spritesheet on the screen,
spritesheet has a `draw` method which will draw the image on the screen and save us a lot of writing.

If you run this, you are supposed to get<br/>
<img src="/img/first.png" style="display: block; margin: 0 auto;"/>
Very nice, is it not ?

### Animating the sprite:

In order to bring this [Ninja](https://en.wikipedia.org/wiki/Ninja) back to life, we'll have to animate it, luckily our spritesheet
contains all the frames needed to do this, so we'll have to tune our code a little bit to get this right.
To do this, we need to cut our image to little chunk and display each at a time.
To do this, we need to call `drawImage(image, clipX, clipY, clipWidth, clipHeight, x, y, width, height)`.
The arguments names are self explanatory. If this is still hard to image, I hope this picture will explain it all.
<img src="/img/drawImage.jpg" style="display: block; margin: 0 auto;"/>

```javascript
let TextureFrame = function(x, y, width, height) {
	this.x = x;
	this.y = y;
	this.width = width;
	this.height = height;	
}; //Define where and how much we should clip the image

let Sprite = function(x, y, width, height, url) {
	//All the old stuff

	this.frame = null;
	loader.loadImage(url).then(image => {
		this.texture = image;
		this.frame = new TextureFrame(0, 0, image.width, image.height); //show the whole image, no clipping
	});
};

Sprite.prototype.draw = function() {
	ctx.drawImage(this.texture, this.frame.x, this.frame.y, this.frame.width, this.frame.height, this.position.x, this.position.y, this.width, this.height);
};
```

And as expected, if you run this you will see no difference.
Let's get onto the fun part and define our `Animation` class.

```javascript
let Animation = function(frames, ticks) {
	this.frames = frames;
	this.index = 0; //the current frame index
	this.ticksPerSec = ticks; //how many ticks before the next frame
	this.ticks = 0; //how many ticks the frames ticked so far
	this.playing = false; //is this playing ?
};

Animation.prototype.play = function() {
	this.playing = true; //play the animation	
};

Animation.prototype.stop = function() {
	this.playing = false;
};

Animation.prototype.tick = function() { //method that will be called each frame
	if ( ! this.playing )
		return; //if this is not playing, do nothing.

	this.ticks++;
	if ( this.ticks >= this.ticksPerSec ) {
		this.ticks = 0;
		this.index = (++this.index) % this.frames.length; //allow the animation to loop back
	}
};

//Change the draw procedure to be animation aware, simple checks aren't performance critical
Sprite.prototype.draw = function() {
	if ( ! this.animation )
		return ctx.drawImage(this.texture, this.frame.x, this.frame.y, this.frame.width, this.frame.height, this.position.x, this.position.y, this.width, this.height);

	let frame = this.animation.frames[this.animation.index];
	ctx.drawImage(this.texture, frame.x, frame.y, frame.width, frame.height, this.position.x, this.position.y, this.width, this.height);
	this.animation.tick();
};

let ninja = new Sprite(0, 0, 280, 385, './run.png'); //each frame is of size 280, 385
let frames = [
	new TextureFrame(0, 0, 280, 385),
	new TextureFrame(280, 0, 280, 385),
	new TextureFrame(560, 0, 280, 385),
	new TextureFrame(840, 0, 280, 385),
	new TextureFrame(1120, 0, 280, 385),

	new TextureFrame(0, 385, 280, 385),
	new TextureFrame(280, 385, 280, 385),
	new TextureFrame(560, 385, 280, 385),
	new TextureFrame(840, 385, 280, 385),
	new TextureFrame(1120, 385, 280, 385),
]; //All clippings

ninja.animation = new Animation(frames, 5); //5 ticks per frame, 12 frame per sec
ninja.animation.play();


function animate() {
	window.requestAnimationFrame(animate); //request for the next frame of this animation

	ctx.clearRect(0, 0, 640, 480); //clear the canvas
	ninja.draw(); //draw our sprite
};
```

And that's all there is to it. Congrats, you have now learnt how to animate your spritesheets. This approach
is very nice and easy to extend.
Here's a demo.

<canvas id="view" style="display: block; margin: 0 auto;">
</canvas>

<script type="text/javascript">
	let canvas  = document.getElementById('view'); //get a reference to our canvas DOM element
	let ctx = canvas.getContext('2d'); //create a 2D context for drawing.

	canvas.width = 640;
	canvas.height = 480; //640x480 resolution

	let loader = {
		cache: {}, //mapping from image url to their promises
		loadImage(url) {
			let img = this.cache[url];
			if ( img )
				return img;

			img = this.cache[url] = new Promise((resolve, reject) => {
				let image = new Image();
				image.src = url;
				image.onload = () => resolve(image);
				image.onerror = (err) => reject(err);
			});
			
			return img;
		},
		await()	 {
			let promises = [];
			for ( let url in this.cache )
				promises.push(this.cache[url]);

			return Promise.all(promises);
		}
	};

	let TextureFrame = function(x, y, width, height) {
		this.x = x;
		this.y = y;
		this.width = width;
		this.height = height;	
	}; //Define were and how much we should clip the image

	let Sprite = function(x, y, width, height, url) {
		this.position = {x, y};
		this.width = width; //the width we want to show
		this.height = height; //the height we want to show
		this.texture = null;
		this.frame = null;
		
		loader.loadImage(url).then(image => {
			this.texture = image;
			this.frame = new TextureFrame(0, 0, image.width, image.height); //show the whole image, no clipping
		});
	};

	Sprite.prototype.draw = function() {
		if ( ! this.animation )
			return ctx.drawImage(this.texture, this.frame.x, this.frame.y, this.frame.width, this.frame.height, this.position.x, this.position.y, this.width, this.height);

		let frame = this.animation.frames[this.animation.index];
		ctx.drawImage(this.texture, frame.x, frame.y, frame.width, frame.height, this.position.x, this.position.y, this.width, this.height);
		this.animation.tick();
	};

	let Animation = function(frames, ticks) {
		this.frames = frames;
		this.index = 0; //the current frame index
		this.ticksPerSec = ticks; //how many ticks before the next frame
		this.ticks = 0; //how many ticks the frames ticked so far
		this.playing = false; //is this playing ?
	};

	Animation.prototype.play = function() {
		this.playing = true; //play the animation	
	};

	Animation.prototype.stop = function() {
		this.playing = false;
	};

	Animation.prototype.tick = function() { //method that will be called each frame
		if ( ! this.playing )
			return; //if this is not playing stop

		this.ticks++;
		if ( this.ticks >= this.ticksPerSec ) {
			this.ticks = 0;
			this.index = (++this.index) % this.frames.length; //allow the animation to loop back
		}
	};

	let ninja = new Sprite(0, 0, 280, 385, '/img/run.png'); //each frame is of size 280, 385
	let frames = [
		new TextureFrame(0, 0, 280, 385),
		new TextureFrame(280, 0, 280, 385),
		new TextureFrame(560, 0, 280, 385),
		new TextureFrame(840, 0, 280, 385),
		new TextureFrame(1120, 0, 280, 385),

		new TextureFrame(0, 385, 280, 385),
		new TextureFrame(280, 385, 280, 385),
		new TextureFrame(560, 385, 280, 385),
		new TextureFrame(840, 385, 280, 385),
		new TextureFrame(1120, 385, 280, 385),
	]; //All clippings

	ninja.animation = new Animation(frames, 5); //5 ticks per frame, 12 frame per sec
	ninja.animation.play();

	function animate() {
		window.requestAnimationFrame(animate); //request for the next frame of this animation

		ctx.clearRect(0, 0, 640, 480); //clear the canvas
		ninja.draw();
	};

	loader.await().then(all => {
		window.requestAnimationFrame(animate); //start the animation when everything is loaded
	});
</script>

### Full Code:

```html
<canvas id="view">
</canvas>

<script type="text/javascript">
	let canvas  = document.getElementById('view'); //get a reference to our canvas DOM element
	let ctx = canvas.getContext('2d'); //create a 2D context for drawing.

	canvas.width = 640;
	canvas.height = 480; //640x480 resolution

	let loader = {
		cache: {}, //mapping from image url to their promises
		loadImage(url) {
			let img = this.cache[url];
			if ( img )
				return img;

			img = this.cache[url] = new Promise((resolve, reject) => {
				let image = new Image();
				image.src = url;
				image.onload = () => resolve(image);
				image.onerror = (err) => reject(err);
			});
			
			return img;
		},
		await()	 {
			let promises = [];
			for ( let url in this.cache )
				promises.push(this.cache[url]);

			return Promise.all(promises);
		}
	};

	let TextureFrame = function(x, y, width, height) {
		this.x = x;
		this.y = y;
		this.width = width;
		this.height = height;	
	}; //Define were and how much we should clip the image

	let Sprite = function(x, y, width, height, url) {
		this.position = {x, y};
		this.width = width; //the width we want to show
		this.height = height; //the height we want to show
		this.texture = null;
		this.frame = null;
		
		loader.loadImage(url).then(image => {
			this.texture = image;
			this.frame = new TextureFrame(0, 0, image.width, image.height); //show the whole image, no clipping
		});
	};

	Sprite.prototype.draw = function() {
		if ( ! this.animation )
			return ctx.drawImage(this.texture, this.frame.x, this.frame.y, this.frame.width, this.frame.height, this.position.x, this.position.y, this.width, this.height);

		let frame = this.animation.frames[this.animation.index];
		ctx.drawImage(this.texture, frame.x, frame.y, frame.width, frame.height, this.position.x, this.position.y, this.width, this.height);
		this.animation.tick();
	};

	let Animation = function(frames, ticks) {
		this.frames = frames;
		this.index = 0; //the current frame index
		this.ticksPerSec = ticks; //how many ticks before the next frame
		this.ticks = 0; //how many ticks the frames ticked so far
		this.playing = false; //is this playing ?
	};

	Animation.prototype.play = function() {
		this.playing = true; //play the animation	
	};

	Animation.prototype.stop = function() {
		this.playing = false;
	};

	Animation.prototype.tick = function() { //method that will be called each frame
		if ( ! this.playing )
			return; //if this is not playing stop

		this.ticks++;
		if ( this.ticks >= this.ticksPerSec ) {
			this.ticks = 0;
			this.index = (++this.index) % this.frames.length; //allow the animation to loop back
		}
	};

	let ninja = new Sprite(0, 0, 280, 385, './run.png'); //each frame is of size 280, 385
	let frames = [
		new TextureFrame(0, 0, 280, 385),
		new TextureFrame(280, 0, 280, 385),
		new TextureFrame(560, 0, 280, 385),
		new TextureFrame(840, 0, 280, 385),
		new TextureFrame(1120, 0, 280, 385),

		new TextureFrame(0, 385, 280, 385),
		new TextureFrame(280, 385, 280, 385),
		new TextureFrame(560, 385, 280, 385),
		new TextureFrame(840, 385, 280, 385),
		new TextureFrame(1120, 385, 280, 385),
	]; //All clippings

	ninja.animation = new Animation(frames, 5); //5 ticks per frame, 12 frame per sec
	ninja.animation.play();

	function animate() {
		window.requestAnimationFrame(animate); //request for the next frame of this animation

		ctx.clearRect(0, 0, 640, 480); //clear the canvas
		ninja.draw();
	};

	loader.await().then(all => {
		window.requestAnimationFrame(animate); //start the animation when everything is loaded
	});
</script>
```

I hope everything is clear enough, the code can be found here. It's free to use for any purpose and doesn't come with any restrictive license.

# So what ?:

Actually that's all there is to it. Animation spritesheets in canvas is very easy and can get a bit more challenging if you try this in OpenGL or DirectX.
The approach is the same and very easy to understand. If you have any question, don't hesitate to post a comment below and I will try to answer it as
fast as I can. If you liked this post, **please** leave a comment below, and thanks for existing.

<img src="/img/byebye.gif-c200" style="display: block; margin: 0 auto;"/>