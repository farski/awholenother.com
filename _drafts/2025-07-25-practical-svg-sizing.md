---
layout: post
title: Practical SVG sizing
date: 2025-07-25 17:48 -0400
reading_time: 15 minutes
tags:
  - HTML
  - SVG
  - web
---

<style>
	div[itemprop="articleBody"] {
		img, svg {
			outline: 2px dashed grey;
			box-sizing: content-box;
		}

		.img-code {
			font-family: monospace;
			font-size: var(--13px);
			border-radius: 10px;
			background-color: var(--block-bg-color);
			padding-inline: 4px;
			white-space: pre;
		}
	}
</style>

_All images in this article will have a gray outline, which has been added using CSS and is not part of any SVG markup._

_Any circles you see that are red are inline SVGs._

<hr>

Here's what it looks like when you put an SVG into a webpage using an `img` tag, like `<img src="/images/green.svg">`

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_no_size.svg"></p>

The code for this SVG file is very simple:

```xml
<svg xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="green" />
</svg>
```

Note that there's really nothing in either the HTML or the SVG code to say how big this image should be. The resulting image that you're looking at in your browser is almost certainly 300px by 150px. That's the default size for most images in most browsers when no other sizing information is available.

If we put that SVG code inline on the page…

<p>
	<svg xmlns="http://www.w3.org/2000/svg">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

…we get the same result.

This is a very simple demonstration of an important property of SVGs: the canvas (not a technical term) that elements of the SVG are drawn onto extends infinitely in all directions. When we're thinking about the space where we can put circles and squares and stuff, there are no outer bounds.

We should also take care to note at this point that when we have numbers and sizes within the SVG document, those aren't pixels. The `r="50"` in this simple image is **not** setting a radius of 50 pixels. It's just…50. 50 will be twice the size of something that's 25, and half the size of something that's 100.

Ultimately, we will want to display the image on a screen, which is made up of pixels, so there will have to be some translation that happens from these unit-less sizes in the SVG's world to pixels in the real world.

A lot of what we are about to cover has to do with how that translation happens.

But back to these circles. We coded circles that had a radius of 50, and the browser decided to display images that were 300x150. Without even getting into how big the circle ended up being, in terms of pixels, we know that the image wasn't the same lenght on all sizes, like the bounding box of a circle would be.

But the browser was able to draw an image without distorting the circle. What we are experiencing here is the browser showing more of that infinite canvas than perhaps we were expecting. With SVGs, there's always the option of deciding how much of that canvas to actually draw to the screen, after you've placed all your shapes. If you make a circle of size 100, there's nothing requiring you to display 100 of anything, pixels or otherwise.

The shapes we place into the canvas, and the region of that canvas that we make visible to a user are two separate operations.

Let's go back to our basic circle. Let's say the default image size doesn't work well for our website. We want the circle graphic to only be as big as necessary to fit the circle. It's great that SVGs have an infinite canvas that we can use when we need to, but for now, let's just make a 100x100 pixel circle.

```html
<img src="/images/green.svg" style="width: 100px; height: 100px;">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_no_size.svg" style="width: 100px; height: 100px;"></p>

That's a good looking circle. But it's actually a little too big—let's make it 50x50px instead

```html
<img src="/images/green.svg" style="width: 50px; height: 50px;">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_no_size.svg" style="width: 50px; height: 50px;"></p>

Huh. That result is maybe a bit unexpected. Rather than ending up with a smaller circle, we've got part of out 100x100px circle that's been cropped. This is actually the same thing we saw with the 300x150px image, though, so maybe you are starting to expect it. In all these cases we've looked at so far, the circle we've ended up with on the screen has had a diameter of 100 pixels, and what's been changing is how much of the SVG's canvas gets displayed.

Why is the diameter 100 pixels, though? Didn't we say that sizes and dimensions in SVGs are unit-less?

Well, yes, but at the point where the browser has to actually draw pixels on the screen, some decisions need to be made for how to translate those unit-less sizes into pixels.

Since we haven't given the browser a whole lot of information to work with, it uses some reasonable defaults. With no other sizing or scaling information, the browser will treat 1 unit of size in the SVG as 1 pixel. It has applied a scaling factor of 1 unit = 1 pixel to the infinite SVG canvas. We still have an infinite canvas, but it's now infite pixels in every dimension, and right near the origin is our circle, which is now concretely 100px in diameter.

The browser then decides which part of that infinite pixel canvas to display to the user, and we've seen how it takes dimension information from the `<img>` tag to do that. Starting from the origin point in the canvas, the region the size of the `<img>` is displayed. When the image is 300x150, we see the whole circle and some extra. When the image is 50x50, we see just the top quarter of the image.

If we provide more details about the sizing and scaling, we can maintain more control over how the browser translates things from the vector SVG world to the pixel world.

Let's update our SVG file to indicate a width and height.

```xml
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="blue" />
</svg>
```

And we'll place that on our page with `<img src="/images/blue.svg">`.

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_100.svg"></p>

Now we get a 100x100px image on the page, without any sizing info in the HTML. So what did that `width` and `height` attributes in the SVG actually do here?

Did they decrease the size of the SVG canvas? Nope, the SVG world of `blue.svg` is still infinite in all directions.

What these values tell the browser is that, when rendering the SVG as a graphic, after you'd applied a scale to convert the canvas into pixels, here's how much of the canvas to show.

In this case, since we still haven't provided any scaling information, the browser will again scale at 1 unit = 1 pixel, which results in a 100px diameter circle within the canvas, and then selects the 100x100px region starting from the origin. Just like how the browser would know that a 10x10 GIF should be displayed as 10x10 (unless you tell it otherwise), the browser now has enough information to know that this pixelized SVG should be 100x100, even without styling the `img` element.

But what if we do style the image element?

```html
<img src="/images/blue.svg" style="width: 50px; height: 50px;">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_100.svg" style="width: 50px; height: 50px;"></p>

We now get a smaller, complete circle. Unlike before when we got just a cropped quarter of a circle. Because the browser had more information early in the vector-to-pixel translation process about how much of the canvas to display, it can use our styles to control the size of the resulting graphic, rather than using it to actually create the graphic.

Setting some funkier dimensions demostrates this.

```html
<img src="/images/blue.svg" style="width: 150px; height: 50px;">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_100.svg" style="width: 150px; height: 50px;"></p>

We can see that the browser was able to find the complete circle from the infinite canvas, and then as part of the actually displaying it on our webpage it could transform it to be the size we asked for.

We can use this to create other results as well. Let's say we want to show only the top quarter of the circle, but we want it to be very small on the screen. We know we could use the `img` tag's style to achieve that sort of cropping for an SVG with no height/width, but then we've already used the tool we have to resize it.

So instead we can again use the SVG's height and width the instruct the browser which region of the infinite canvas to use when constructing the less-infinite pixels for the image…

```xml
<svg width="50" height="50" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="orange" />
</svg>
```

```html
<img src="/images/orange.svg">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_50.svg"></p>

…and then style the `img` element to make that resulting graphic the size and shape we want.


```xml
<svg width="50" height="50" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="orange" />
</svg>
```

```html
<img src="/images/orange.svg" style="width: 10px; height: 10px;">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_50.svg" style="width: 10px; height: 10px;"></p>

We're at a point where we have a good amount of control of how shapes in an SVG's infinte canvas get displayed on a webpage. But right now we're stuck making a lot of decisions based around the origin point of that canvas.

Let's say we want to display the lower left quarter of the circle, rather than the upper left.

We can take advantage of another attribute available on SVGs called `viewBox`, which gives us even more control over deciding which part of the infinite canvas gets rendered.

Let's add `viewBox="0 50 50 50"` to our orange circle SVG.

```xml
<svg width="50" height="50" viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="magenta" />
</svg>
```

Before we look at the results, let's talk about what this will do. `viewBox` takes a starting point (the first two values: `0 50`) and a width and height (the last two value: `50 50`), which define the specific region of the infinite canvas that we want to render out to the page. This is what allows us to get away from the origin, since we can set that starting point to anywhere on the infinite canvas.

So if the `viewBox` has a size (50x50 in this example), what are `width` and `height` doing for us at this point?

They are now playing the important final role of allowing us to decide on how the shapes in our SVG canvas are scaled to pixels. So far in our examples, if we've wanted anything to be scaled differently than the 1 unit = 1 pixel scale we keep seeing, we've had to do it by styling the `img` element. We haven't been able to say, "I know this circle is size 100 in SVG land, but its meant to be a tiny little circle on the page, not actually 100 pixels," unless we transform it after getting those 100 pixels back from the SVG translation.

Now let's take look at the result of adding `viewBox`:

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_50_2.svg"></p>

That's not bad! We got the lower-left quarter of the circle like we wanted. We still have a circle that's has a diameter of 100px, though. We know we could resize it by styling the `img` element, but we also know that we can employ scaling as part of the SVG itself.

Let's make one more variation of this SVG file that will scale the circle down to the size we want.

```xml
<svg width="10" height="10" viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="yellow" />
</svg>
```

```html
<img src="/images/yellow.svg">
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_50_3.svg"></p>

So now we've got a image with a natural size of 10x10, we didn't have to size or style the `img` element at all, and it's rendering the portion of the SVG's canvas that we wanted, even though the complete circle still has a diameter of 100 units in the vector world (meaning, we didn't have to change `r="20"` or anything like that to make the image smaller). And remember, we _could_ style the image to change the image from its natural size if we wanted to at this point.

Let's put it all together: we have an infinite vector canvas, unto which we can draw shapes. We define the sizes of those shapes using unit-less values, **not** pixels. We can then use `viewBox` to select a specific portion of the canvas that we want the rendered version of the SVG to be. We can also set a scaling factor, which determines how many pixels that selected portion should turn into. Finally, we can further transform the resulting pixels using methods shared with raster images on a webpage, like resizing an `img` element.

Applying that to our last example… on our infinite canvas, we drew a circle. The circle's center was at (50, 50) and it had a radius of 50. We decided that we wanted to render the region of this canvas starting at (0, 50), with a width and height of 50. And we said that that 50x50 region should represent an image that is 10x10 pixels, or, every 5 units of size in vectorland should be 1 pixel in rasterland.

It took us a little while to get here, but hopefully now you have a sense of the interaction between SVG dimensions and viewBox, and how they help you render out the shapes you've created on an SVG canvas.

The examples we've looked at conveniently used the same aspect ratio for the region selected by viewBox and the SVG's width/height, so you may be wondering what happens if they don't align.

You have options of how that gets handled, but by defauly basically what happens is the region you _select_ with `viewBox` will always be present in the resulting rendered image, and the region that is actually _used_ to produce that rendered image will expand from that selected region to create a region that is the aspect ratio of the width and height (i.e., something akin to letterboxing, or `contain` in CSS). When the aspect ratios match, the region doesn't expand at all.

Here's an example of a `viewBox` with a 1:1 aspect ratio, but a 1:2 width/height ratio.

```xml
<svg width="40" height="80" viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="yellow" />
</svg>
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_aspect.svg"></p>

You'll notice that the region we actually cared about and selected using `viewBox` has been centered in the portion of the canvas that was ultimately selected and rendered in the final image. That positioning is one of the things you have control over when handling these discrepencies.

In many simple cases, you will likely keep these aspects ratios aligned, and won't have to worry about how positioning is handled. But in cases like where you may be animating parts of the SVG, you'll want to define those properties yourself to make sure things behave as desired.

Be aware that SVG files used with `img` tags cannot use `<svg style="width; height;">` to set the size, or scale with `viewBox`, of the resulting image. You must use the `width` and `height` attributes.

<hr>

We've mainly looked at the use of SVGs through `img` tags, but obviously one of the most compelling reasons to use SVGs is that they can be included in your HTML documents inline, which allows them to be styled with CSS and manipulated as part of the DOM just like any other HTML elements.

All the fundamentals we've discussed hold for inline SVGs, in terms of the canvas, selecting a region with `viewBox`, and scaling with the SVG dimensions. When you supply all the values explicitly, there is no practical difference between how inline or file-based SVGs get rendered.

You may run into some cases where some of these values aren't defined, though, and in those situations the behaviors between inline and files can be a bit different.

Here are some rapid fire examples to demonstrate, but we won't tear them down too much. _All images from this point forward are inline SVGs._

With no size/scale, we get the default 300x150px image.

```xml
<svg xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg xmlns="http://www.w3.org/2000/svg">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

With width and height, we select a region of the canvas and it's displayed at the default scale.

```xml
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

With only CSS sizing on the `svg` element, the size is used to select the canvas region with the default scale.

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="width: 50px, height: 50px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg xmlns="http://www.w3.org/2000/svg" style="width: 50px; height: 50px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

If you include both `width`/`height` attributes and set the `svg` element's size with `style`, the `style` will be used for canvas region selection, and `width`/`height` will have no affect.

```xml
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg" style="width: 50px, height: 50px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg" style="width: 50px; height: 50px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

With a `viewBox` and `style`, the width and height from `style` are used to determine scale.

```xml
<svg viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg" style="width: 10px; height: 10px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg" style="width: 10px; height: 10px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

With a `viewBox`, `style`, and `width`/`height`, the width and height attributes are again ignored.

```xml
<svg width="1000" height="1000" viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg" style="width: 10px; height: 10px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg width="1000" height="1000" viewBox="0 50 50 50" xmlns="http://www.w3.org/2000/svg" style="width: 10px; height: 10px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

I would recommend using `style` over `width`/`height` for inline SVGs.

If you want the final image to be stretched to the given aspect ratio, adding `preserveAspectRatio="none"` without a `viewBox` won't yield the desired result.

```xml
<svg preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" style="width: 150px; height: 50px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" style="width: 150px; height: 50px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>

But including a `viewBox` will allow you to distort the image.

```xml
<svg viewBox="0 0 100 100" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" style="width: 150px; height: 100px;">
	<circle cx="50" cy="50" r="50" fill="red" />
</svg>
```

<p>
	<svg viewBox="0 0 100 100" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg" style="width: 150px; height: 50px;">
		<circle cx="50" cy="50" r="50" fill="red" />
	</svg>
</p>
