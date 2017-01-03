---
layout: post
title:  "An introduction to acceptance testing"
date:   2016-12-07 19:03:42 +0000
categories: Testing
author: Adrian Clay
image: robin.jpg
---
Acceptance testing is the process of checking that what has been created meets its requirements.

For example, this is the first post on my new blog.  I have expectations for this blog, which are my requirements and as I’m building it I’ve been checking that what I’ve built meets these requirements.  I’ve outlined some requirements below.

* Adrian can create blog posts.
* Adrian can create additional content pages which aren’t displayed as blog posts.
* Adrian can use HTML to create these pages.
* Visitors can iterate through a reverse chronological list of post titles.
* Visitors can view blog posts and content pages.
* Pages can be compiled down to assets and served statically.

For each of the requirements above I can assert if the blog I have built meets them.  This has been a simple example.  I am the only person with expectations and when building the blog I only need to take into account my expectations.

When building a product, the end user will be the ultimate deciders of whether it meets their requirements.  Their acceptance of the product is most important and can be the difference between gaining a new customer or not.

Identifying requirements takes time.  Testing that what has been created meets those requirements also takes time.  As you increase the amount of acceptance testing you perform, you also increase the cost of creating the product.  As you decrease the amount of acceptance testing, you expose yourself to the risk of building something that won’t sell.

The acceptance testing process is, therefore, a prime candidate for optimization.  Automating the testing of software is attractive and easy.  To automate your testing, you create scenarios which cover the software's requirements in as much or as little detail as required.  Each scenario is built up of steps which correspond to features in the software.  Execution of the scenarios result in quick feedback, working or not.  You will still need to invest time identifying the requirements and, additionally, you have a cost for creating the automated tests.  As long as the time needed to create the automated tests is less than the time it would have taken to manually run them, then you will make a saving against those costs.  The saving is most pronounced when the tests are to be performed tens or hundreds of times.

In conclusion, acceptance testing reduces risk when developing something new.  Context dictates how much testing is required, e.g. a cancer fighting drug vs a throat lozenge.  Automating your acceptance testing can reduce its cost, resulting in higher value for money.
