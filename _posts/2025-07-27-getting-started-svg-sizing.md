---
layout: post
title: Getting started with SVG sizing
date: 2025-07-27 15:35 -0400
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
	}
</style>

_All images in this article will have a gray outline, which has been added using CSS and is not part of any SVG markup._

_Any circles you see that are <span style="background: indigo; border-radius: 3px; padding: 1px 4px;">indigo</span> are inline SVGs. All other colors are `img` elements referencing an SVG file._

<hr>

When you start with an SVG, understanding what actually ends up on the screen can be a little tricky. Documentation will talk about things like viewports and user space coordinates, which are indeed parts of the system that make SVG graphics so powerful, but they can make the fundamentals hard to grasp.

What helps me is thinking about SVGs in a slightly more practical, but hand-wavey, way.

## The infinite canvas

The first thing to understand about SVGs is that in the vector world where you create shapes, and give them sizes and positions, the canvas onto which you are placing those shapes is infinite. It always extends infinitely in both directions, and there is no way to change that—there are no options or attributes that change or limit the size of an SVG‘s canvas.

You may only care about a relatively small part of that infinite plane right near the origin, but the rest of the canvas will always be there.

Which is to say, the universe where your SVG shapes live has no inherent size or bounds. Whether you make a circle with a radius of 10, or a complex vector masterpiece stretching very far in all directions of the canvas, the world is the same size.

So, when an SVG gets rendered on a webpage, there **must** be a choice made about which part of the infinite canvas to show. If you’re every working with an SVG and it doesn’t seem like you’re making that choice, it’s important to understand that _something_ is making the choice. The choice has to be made, because we can’t put infinitely sized images on a webpage.

## It’s not pixels until it is

When we talk about an SVG’s canvas being infinite, we aren’t saying that it’s infinite _pixels_ wide. Vector images like SVGs do not deal in pixels. Instead, they deal in unitless sizes.

If you create a circle in an SVG with a radius of 50, `<circle cx="0" cy="0" r="50">`, this does **not** mean 50 pixels. It’s just…50. It means that this radius should be twice the size of something that’s 25, or half the size of something that’s 100.

Similarly for positions and coordinates, if you move the circle to `cx=1000`, that’s not moving it 1,000 pixels, it’s moving it twice as far as 500, and half as far as 2,000.

When we are living in the safety and comfort of vectorland, where we do our drawing, all we have to worry about are these relative sizes.

We will need to leave vectorland at some point, though. We’re creating these SVG images so that people can see them, and people will see them on devices that think in terms of pixels.

It’s not quite right to say that the _shapes_ will leave vectorland and get transformed into pixels. The shapes we define in SVG will always be vectors in their vector world. But when we ask for the SVG to show up on a webpage, the browser will have to create an (evil?) twin pixel version of each shape we’re displaying to the user.

But there is not One True Way to create a pixel twin from a vector shape. There’s going to be some sort of translation process for how unit-less sizes in vectorland should be codified into pixels in rasterworld. Remember, the circle you drew with a radius of `50` was **not** 50 pixels. You may know, for instance, that the circle should only be 8 pixels in diameter on someone’s screen, even though the artist who drew it in the SVG happened to give it a radius of 50. Someone else may draw radius 50 circle in an SVG, and display it on a website at 500 pixels wide.

This is possible because we get to control how the unit-less sizes of our shapes in vectorland get translated into pixels. We can tell the browser, for instance, that a size of 50 in vectorland should equate to 4 pixels when it’s creating the raster twin of that circle. So your radius 50 vector circle turns into a pixel circle with a radius of 4 pixels, which gives you the 8px circle on the screen that you were hoping for.

## Two steps, two choices

We’ve now talked about choices that you get to make when working with SVGs that determine what shows up on a display for some SVG. That is, besides what to actually draw on the canvas. It's helpful to think in terms of two disconnected, but related operations: decide _what_ to draw onto the infinite canvas, then decide _how_ a part of that canvas gets turned into pixels.

This post is not about that first step (drawing).

It’s about the second step, and understanding the two choices we make during that step, which we’ve started to look at already: selecting a specific portion of the canvas to show, and telling the browser what scale it should use when converting vector sizes into raster pixels.

## Selecting a region

Let’s say we have a very simple SVG:

```xml
<svg xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

We’re drawing a green circle, with radius 50, on the infinite canvas, and its center is at the origin point.

We haven’t provided any information to the browser about which region of the infinte canvas we actually want to include in a rendered version of this SVG. Remember, the browser **must** make that decision. If we don’t tell it how, it will make it’s own choice (which we’ll look at in a bit).

We want to make that choice ourselves, so let’s look at how we control that selection.

We’ll use the `viewBox` attribute on the `<svg>` tag.

Here’s an example of that: `viewBox="10 30 100 108"`. We set four values in the attribute to define our viewBox. The first two values, `10 30`, determine the starting position within the infinite canvas, and the last two values, `100 108`, dictate the size of the viewBox from that point.

The viewBox we end up with is the specific region of the infinite canvas that we want to display.

In this case, we are saying that we want the 100x108 region that’s offset from the origin by `(10, 30)`. Out of the entire infinite canvas we have to work with, that particular 100x108 region is what we care about.

The important region will change from SVG to SVG, and we get to make the choice for each one by setting a viewBox.

Let’s go back to our simple circle, and add a `viewBox`.

```xml
<svg viewBox="-50 -50 100 100" xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

Here we’re setting a viewBox to be 100x100, and is offset from the origin by `(-50, -50)`. This is a reasonable selection, given that our circle had a diameter of 100 and was centered at the origin. This viewBox selects a region around the origin that is just big enough to contain the entire circle.

We could make other choices with the viewBox for this circle, though. If we only wanted to show the right half of the circle: `viewBox="0 -50 50 100"`. Or just a sliver of the bottom: `viewBox="-50 45 100 5"`.

## Selecting a scale

Selecting a region gets us halfway there, but we still need to instruct the browser what scale it should use when converting unit-less sizes from vectorland into pixels for rasterworld.

This is also a choice that the browser must make, and will make on its own if we don’t give it enough info (we’ll look at that soon, too). But let’s keep that control for now and make the choice ourselves.

To determine the scale, we don’t provide an explicit scaling factor. So if we want our size 100 circle to be 20 pixels wide on the screen, we don’t define somewhere `scale="5"` or `scale="0.2"` or anything like that.

Instead, we tell the browser what the dimensions of the pixelized conversion of our selected region from the SVG canvas shoud be, and the browser will determine the scale from that. We do this using the `width` and `height` attributes on the `svg` element.

Let’s add width and height to our circle:

```xml
<svg viewBox="-50 -50 100 100" width="20" height="20" xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

So while we are still using viewBox to select a 100x100 sized region of the infinite canvas, we’re now also explicitly saying that we want that region to be 20x20 pixels in rasterworld. The browser then knows that we want to use a scale for all _vector sizes_ to _pixels_ of 100 to 20, or 5 to 1. Every 5 units of size in vectorland will become 1 pixel in rasterworld.

Here’s what that looks like:

<p>
	<svg viewBox="-50 -50 100 100" width="20" height="20" xmlns="http://www.w3.org/2000/svg">
		<circle cx="0" cy="0" r="50" fill="green" />
	</svg>
</p>

In this example, we’ve conveniently created a viewBox and a pixelized image size that have the same aspect ratio: both 100x100 and 20x20 are squares. This is often what you’ll want, but it’s obviously possible to use values that don’t align.

```xml
<svg viewBox="0 0 50 50" width="40" height="80" xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

<p>
	<svg viewBox="0 0 50 50" width="40" height="80" xmlns="http://www.w3.org/2000/svg">
		<circle cx="0" cy="0" r="50" fill="green" />
	</svg>
</p>

In this example, we’re using a viewBox that selects only the region of the canvas with the lower-right quarter of the circle (which is a square). Then we’ve set a size of 40x80 pixels, which is a tall rectangle.

You have options of how aspect ratio discrepencies get handled, but by default basically what happens is the region you _select_ with `viewBox` will always be present in the resulting rendered image, and the region that is actually _used_ to produce that rendered image will expand from that selected region to create a region that is the aspect ratio of the width and height (i.e., something akin to letterboxing, or `contain` in CSS). When the aspect ratios match, the region doesn’t expand at all. By the default, your desired region will end up centered within the expanded region.

## On the page

Our understanding of infinite canvases, viewBoxes, and scaling gives us all the tools we need to turn vectors into pixels in the way we want. But just because pixels exist doesn’t mean we need to use all of them. Imagine a case where you have a JPEG image that is 1920x1080, but on the page you don’t have quiet enough room for it to be that big, so you resize the `img` element to be 640x480.

This is a final presentation layout step that we can take with our SVGs as well, if we want. Here’s our circle from before; same viewBox and same scaling, but we’ve added two new attributes: `preserveAspectRatio` and `style`.

`style` is the standard CSS styles that you’re likely familiar with. In this case, we are saying that the final displayed element should actually be 60x10 pixels.

Because the size we gave for our pixelized version of the SVG was 20x20 (a square) and the final image element size is a long rectangle, we are going to get some distortion. We actually need to opt into that distortion using `preserveAspectRatio="none"`. This property makes image resizing of SVGs work very similary to raster images.

```xml
<svg viewBox="-50 -50 100 100" width="20" height="20"
	preserveAspectRatio="none"
	style="width: 60px; height: 10px;"
	xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

<p>
	<svg viewBox="-50 -50 100 100" width="20" height="20" preserveAspectRatio="none" style="width: 60px; height: 10px;" xmlns="http://www.w3.org/2000/svg">
		<circle cx="0" cy="0" r="50" fill="green" />
	</svg>
</p>

You don’t have to do it this way. Omitting `preserveAspectRatio` or using other values will change the way layout sizing of the image behaves, and you can play around with those options on your own.

## Letting the browser decide

At the beginning of the post we talked about the fact that someone must make the viewBox and scale decisions, and if you don’t do it the browser will. Are there any cases where it’s preferrable to let the browser decide?

Imagine you want to have an SVG of a circle as an asset, but it’s not intended for any specific application on your website. Sometimes you’ll use it as a tiny little dot to indicate an unread message. Other times it will be a medium sized circle behind an avatar, and maybe even a huge circle as part of a background.

In this case, it doesn’t really make sense to bake an intinsic size into the SVG itself. Really all we’re looking to do is capture the essence of a circle in the SVG, and then we’ll make decisions about how big it should be in the layout later.

So let’s bring back our basic circle:

```xml
<svg viewBox="-50 -50 100 100" xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="green" />
</svg>
```

We’re still defining a viewBox explicitly, because we do want to clearly define which region of the infinite canvas to use (the circley region). But we have omitted the width and height.

And if we place that somewhere on the page with `<img src="/images/circle.svg">`…

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_viewBox_noSize.svg"></p>

…what you probably see is a very large circle. The region of the canvas was correct, because it was well defined with our viewBox, but without any hints about what the scale of the resulting circle should be, the browser had to decide and we got something that’s not very practical.

If we give the `img` element a size like `<img src="/images/circle.svg" style="width: 8px; height: 8px;">`…

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_viewBox_noSize.svg" style="width: 8px; height: 8px;"></p>

…we get the tiny unread notification circle we were hoping for, even without an explicit scale. The browser had to scale the circle with a diameter of 100 in the SVG to something, which it did, and then it transformed the result to the 8x8 element on the page that we asked for.

What if we include a size, but no viewBox?

```xml
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
	<circle cx="0" cy="0" r="50" fill="blue" />
</svg>
```

<p><img src="/images/posts/2025-07-25-practical-svg-sizing/circle_100.svg"></p>

Even using just `<img src="/images/blue.png">`, which has no sizing info on the element, we do get a 100x100 pixel image. What we see is that the browser defaulted to a viewBox that matched the size we provided with a 1:1 scale, starting from the origin. So basically an implicit `viewBox="0 0 100 100"`. In this case, that didn’t work great because the center of our circle was at the origin, so we only got a corner.

But you could image that we could just shift `cx` and `cy` a bit so that the circle were in that implicit viewBox, and we actually would get sort of a shorthand for defining size, scale, and viewBox with few attributes required. This is why, in the wild, you will run into SVGs without a viewBox with some regularity.

## The rest

SVG is an extremely complicated technology, and this post barely scratched the surface. Even amongst the few topics we spent time looking at closely today, there are edge cases, exceptions, technical explanations, and more general models that would make things seem a lot different than how I’ve described them.

But from a place of fundamentals, if you are just getting started with SVGs, these core concepts of the canvas, selecting part of the canvas, and scaling the vectors into concrete pixels are one of the more imporant pieces to get a good feel for.
