---
layout: post-sidemenu-jekyll
title:  "Create Stripes Using Inkscape to Decorate Blog Post"
categories: webdev
date:   2016-06-10 02:51:15 +0700
tags: [inkscape, jekyll]
author: epsi

excerpt:
  I'm thinking of giving different color variation for each post.
  The idea is there are 12 months in a year,
  and there are rgb (red, green, blue) colors,
  both contain combination of three number.
  You can see how easy it is. 
  I don't even think about what color,
  or any aesthetic aspects. 
  It simply math.

related_link_ids: 
  - 16061225  # Elapsed Time
  - 16061409  # Webcoder Begin
  - 16060951  # Jekyll Archives  
  - 16052948  # Jekyll Related Posts

---

I'm thinking of giving different color variation for each post.
The idea is there are 12 months in a year,
and there are rgb (red, green, blue) colors,
both contain combination of three number.

I don't know if it is a good idea or not,
but I feels like my blog is too plain.
I'm not really a designer, 
so I add this colored stripes for my blog.

-- -- --

## Inkscape steps

* Create new inkscape document 360px x 200px

* Create the transparent white stripes pattern. 30x30

> <https://tucsonlabs.com/2009/03/19/patterns-with-inkscape/>

* Create a box width 30px x height 200px. 
  Duplicate each box beside for each 12 months 

* Colorize each month boxes with RGB.

* Create a box of 360px x 200px
  Fill with white stripes pattern.
  Lower the transparency for about 30% or 40% to make a soft pastel effect.

* Create a vertical gradient over the layers from #ffffff88 to #ffffff00

* Export each month box objects to separate png images 
  from 01.png to 12.png.

* You can even export a transparent png overlay 00.png,
  and still preserve your CSS color.

-- -- --

## Detail on Colors in Inkscape

The first six colors combination is easy:

* rgb(255,0,0)
* rgb(255,255,0)
* rgb(0,255,0)
* rgb(0,255,255)
* rgb(0,0,255)
* rgb(255,0,255)

The next six colors, is combination of above colors.

* rgb(255,127,0)
* rgb(127,255,0)
* rgb(0,255,127)
* rgb(0,127,255)
* rgb(127,0,255)
* rgb(255,0,127)

Of course you can utilize your own color scheme that is suitable for your site.

![Inkscape Stripe All Colors][image-inkscape-stripes]{: .img-responsive }

You can see how easy it is. 
I don't even think about what color,
or any aesthetic aspects. 
It simply math.

-- -- --

## Glossy

With about similar method, you can apply it to glossy design.
Glossy design has been popularized by Vista a decade ago.

* Copy Inkscape above, and reduced page document to 360px x 80px

* Select all by mouse, and resized all object's heights from 200px to 80px.

* Create a white (<code>$fff</code>) box on top half of the page (40px x 360px)
  And reduced the transparency to about 4% or 10%.

* Export to each month to png.



![Inkscape Glossy All Colors][image-inkscape-glossy]{: .img-responsive }

-- -- --

## Website steps

* CSS

{% highlight css %}
.stripe-01 {
    background: transparent url(/images/stripe/01.png) repeat-x left top;
}
{% endhighlight %}

* HTML (sample using Jekyll)

{% highlight html %}
<article class="post well stripe-{{ page.date | date: '%m' }}">
{% endhighlight %}

![Jekyll Stripe Background][image-jekyll-stripes]{: .img-responsive }

-- -- --

You can see the result on both figure above.

* Inkscape SVG document
* Decoration in blog post

I don't know how long would I put this stripe stuff on my blog.
It looks childish, and I lost the boldness of the design.
I wish I have a better decoration idea than just this plain pages.

Thank you for reading



[//]: <> ( -- -- -- links below -- -- -- )

[image-inkscape-stripes]: {{ site.url }}/assets/posts/webdev/2016/06/inkscape-stripe-all-colors.png
[image-inkscape-glossy]: {{ site.url }}/assets/posts/webdev/2016/06/inkscape-glossy-all-colors.png
[image-jekyll-stripes]: {{ site.url }}/assets/posts/webdev/2016/06/jekyll-stripe-background.png


