---
layout: post
title: Creating a Beer Recommendation Engine
---

Beer is one of my passions. I'm an award-winning homebrewer. I've judged beer competitions. I'm an active member in my
local homebrewing clubs. I've reviewed just under 1000 unique beers on Untappd. I have several floorplan ideas for the taproom
of the craft brewery I will definitely someday own.
I'm drinking a beer while I'm writing this post.

So clearly, when I was tasked to do Natural Language Processing for my third data science project at Metis, 
it made sense for me to apply my extensive domain knowledge of that famous drink known as beer.

My goal is to create a recommendation engine for beer that is actually useful. The recommendations that currently exist on 
beer rating websites are fairly silly at this point. Generally, if you're on a beer's page, you'll get something like: 
"Oh you like Sierra Nevada Pale Ale? Well, you'll love Three Floyds Zombie Dust!"
Well, you're definitely not wrong. However, I don't think I'll fly halfway across the country to have a bearded man deeply sigh 
towards me and say "No, everybody knows you can only get that beer at 2:15pm on the third Tuesday of the month."

So clearly, just taking the beer's style and recommending the highest rated beers in that style is fairly useless. Also, while
many beers within a style are pretty similar, some styles (like IPAs or sours) have a broad range of characteristics. To account 
for this, I decided to compile individual reviews of beers and find key descriptive phrases within the corpus.

My first iteration of the recommender takes the top 50 beers of each style, and the top 25 reviews for each beer as a single 
document. I applied tf-idf to each of these reviews to account for the fact that almost every beer review mentions things like
malt or hops. Then, I applied latent semantic analysis to reduce my feature space to ~500 dimensions. Taking this dataset,
I applied cosine similarity between each of the documents to find the beer with a set of reviews that has the most similar
language. 

![Data Flow](/images/beer_recommender.png)

This first iteration is already producing very good results. If I search for a beer like Lost Abbey's Track #8 (a bourbon barrel
aged Belgian quadrupel with cinnamon sticks and dried chilis) you get the following results:

Beer | Description
--- | ---
Melange No. 3 | A huge (15.5% ABV) bourbon barrel aged strong ale
Bourbon Barrel Aged Angel's Share | Another big beer from Lost Abbey aged in bourbon barrels
Nor' Easter | Another dark and rich belgian ale
Kuhnhenn Bourbon Barrel Fourth Dementia | A high abv old ale aged in oak
Mother of All Storms | A big barleywine also aged in oak barrels

Every beer that is recommended is a big, boozy, rich and dark beer with distinct yeast characteristics. What is especially helpful
is that there are three different beer styles within the five recommendations. That is something you generally don't get with
other recommendation services, yet these beers are all very similar.

Let's try a beer from a fairly broad style, like American sours. I'll take one of my personal favorites, Sante Adairius' West Ashley
(a sour saison aged in Pinot Noir barrels with apricots):
Beer | Description
--- | ---
Cascade Apricot Ale | Another apricot sour
Cantillon Fou' Foune | A Belgian lambic with apricots
Imperial Apricot bu Weisse | A Berliner Weisse with apricots
Apricot bu | The same as above, but the less strong version
Valley of the Heart's Delight | Another local brewery's apricot sour

Clearly, the apricot addition is a common theme amongst these beers. Also, all of the beers are sours, but there is still
representation from three different sour styles in the recommendations.

Just to toss out a little more domain knowledge, there's two main flavor components in most IPAs nowadays. These heavily hopped
beers either feature a very fruity (citrus, tropical) character, or a piney and resinous character (hops are closely related to 
cannabis, a recreational drug used by recidivists and jazz musicians). So will the model be able to detect the difference between
these two fairly distinct styles?

Let's try Sixpoint's Resin IPA, which, as you may be able to tell, falls into the pine category:
Beer | Marijuana Euphemism Used in a Review
--- | ---
Deviant Dale's IPA | Herbal, green
G-Bot Double IPA | Skunk, resin
Gemini | Grassy
471 IPA | Floral
Green Flash Palate Wrecker | Dank


A neat little feature I threw in is if you enter text that is not the name of a beer in the database, it will treat the input as a 
description of the beer that you're looking for. So if you want a "spicy floral herbal ale with coriander", perhaps you should try
one of the following: 

* Spice Of Life
* Shiner White Wing
* Traquair Jacobite
* Verloren
* Samuel Adams Little White Rye

I'm currently working on some nifty little features for the web app before I deploy it, but I'll be sure to update when it is
ready to be used by the public!
