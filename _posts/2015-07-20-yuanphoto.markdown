---
layout: post
title: "Display a Circular Photo in Webpage"
date: 2015-07-20 09:00:00
categories: [Web]
---

Usually an image is rectangular. However, in many cases we need to display an image in circular shape, e.g. a circular profile photo in a personal home page. So how?

One intuitive solution is to cut a circular area out of the original photo using some image processing software like Photoshop. But there is some more elegant way. Actually, just a single line of CSS code is enough.

Our tool is 'border-radius', which introduces to a rectangular object the effect of border radius, just like what we can see in iOS app icons. Now imagine we increase the border radius of a square shape, until it cover half of a square border. Obviously the square shape finally becomes a circle.

![circlephoto]({{ site.baseurl }}/images/circlephoto.png)

Now let's code. Define a class in .css file and let the 'border-radius' be 50%:

    .yuan { 
        border-radius:50%;
    }

Then in the .html where the photo should be placed, include this class:

    <img class="yuan" src="{{ site.baseurl }}/images/longmao.jpg" width="80%" height="80%">

Done! That's what I've done in the banner of this home page:

![banner]({{ site.baseurl }}/images/banner.png)

Note that here the original photo is better to be square. If it is rectangular, the result will be an ellipse.




