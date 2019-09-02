---
layout:     post
title:      A One Week Website - A Division of Graduate Student Labor
date:       2019-7-18 12:00:00
summary:    Making website for our group on short notice. What I learned and how it turned out.
categories: Research
---

Check out the website I made here: [Madhavan Lab Website](http://madhavanlab.org/)

There are many misconceptions about what a PhD entails. With Physics, it varies from people thinking that I do what they hated in high school to what they saw in the latest science fiction movie. In reality, the answer is "Whatever my advisor told me to do". This is of course a joke, but the best humor is often uncomfortably close to the reality.

Last week my advisor told me we need a new group website. This baton had been passed a few times within the group, finally landing in my hands. But this time was different. Funding proposals were due in a week. It had to be done.

There was a small University of Illinois Wordpress powered publishing site, but after a few hours I realized nothing good was going to come from it. I had previously looked into making a GitHub pages for myself, so I decided to make one for the lab. I already had made a GitHub account for some code base within the lab.

I started by watching tutorials from [Mike Dane](https://www.youtube.com/watch?v=T1itpPvFWHI&list=PLLAZ4kZ9dFpOPV5C5Ay0pHaa0RJFhcmcB) on using Jekyll to create statically generated websites. When hosting via GitHub pages, it can use Jekyll to compile your files into a website. In order to use Jekyll you need Ruby installed, so I did that.

From these videos I learned about themes. Themes are premade Jekyll website templates, very helpful for getting started. When you first make a website in Jekyll, you run:
```shell
jekyll new my-website
```
And a folder called `my-website` will have a basic website with the theme `minima`. This is a clean theme that is helpful for getting started and working in Jekyll. To see the website, run:
```shell
bundle exec jekyll serve
```
This will run bundle to load all the necessary gems and then put your website up on a localhost. You can see the website in the url `localhost:port` where port is the number in the IP address after 127.0.0.1.

However for our group website, I wanted the focus to be more on images than text content. So I chose the [alembic](https://alembic.darn.es/) theme. This theme has a nice banner picture for each page and a navigation bar on the top. Both features I was looking for. If you want to find a Jekyll theme, [here are some free ones](https://jekyllthemes.io/free).

Some themes need to be forked (i.e. downloaded from GitHub to your `my-website` folder) while others can be added within the `_config.yml` file. If you ever make a website with Jekyll, this is where you will add plugins and define things you need throughout the site. Often each individual theme will have a format for this file where you can edit preexisting variables.

The place where Jekyll excels is in the integration with Markdown. Markdown is a language that allows for easy text to HTML conversion. If you want to use HTML, you can put it directly in a `.md` file and it will work as expected. Before trying to make the group website I did not know any HTML, so this was very nice. Often there is a `_posts` file in a theme where Markdown content, like blog posts, can go and show up within a feed or something similar.  

So I add content and I go to my advisor. She wants it to look like another groups, professionally done with a nice image carousel on the homepage. The theme doesn't have a image carousel. So I search and find [this tutorial](https://jekylltools.github.io/jekyll-ideal-image-slider-include/examples/). I don't know javascript, but manage to make it work. A little more cut and paste than I'm proud of but it's up and I put some images in.

Then the font is too big. This requires me to learn some CSS and Sassline since it is used for this project. I try the fontsize in the `_variables.scss` file as mentioned in the Sassline tutorial, but it turns out it is a known bug. I copy and paste a fix and it works well enough.

Getting a personalized domain was surprisingly easy. We got `www.madhavanlab.org` for only twelve dollars a year. All I needed to do was change the domain namge in the Github repository and add some custom DNS files on Google domains page. I followed the tutorial found [here](https://medium.com/employbl/launch-a-website-with-a-custom-url-using-github-pages-and-google-domains-3dd8d90cc33b).

At this point I was pretty happy with the website, five days after the initial request. The current website looks similar to it did at day five, just with some content updates and newer pictures.

While making this website was a lot of work in a small amount of time, I generally enjoyed the experience and it helped give the understanding to make this personal website. So even though being a PhD involves a lot of stuff besides original research, it has made me a quicker learner and given me the confidence to solve problems where I have little to no experience.
