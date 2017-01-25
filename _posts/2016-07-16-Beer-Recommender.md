---
layout: post
title: Creating a Beer Recommendation Engine
---

**Update (1/24/17): The site is back up at [chooseabeerfor.me](http://chooseabeerfor.me)**

Beer is one of my passions. I'm an award-winning homebrewer. I've judged beer competitions. I'm an active member in my
local homebrewing clubs. I've reviewed just under 1000 unique beers on Untappd. I have several floorplan ideas for the taproom
of the craft brewery I will definitely someday own.  
I'm drinking a beer while I'm writing this post.

So clearly, when I was tasked to follow my passion for my final project at at Metis, 
it made sense for me to apply my extensive domain knowledge of that famous drink known as beer.

My goal is to create a recommendation engine for beer that is actually useful. The recommendations that currently exist on 
beer rating websites are fairly silly at this point. Generally, if you're on a beer's page, you'll get something like: 
"Oh you like Sierra Nevada Pale Ale? Well, you'll love Three Floyds Zombie Dust!"
Well, that's definitely not wrong. Sierra Nevada created the platonic ideal of an American pale ale, and Three Floyds' Zombie Dust is an amazing interpretation of the style. However, I don't think I'll fly halfway across the country to have a bearded man deeply sigh 
towards me and say "No, everybody knows you can only get that beer at 2:15pm on the third Tuesday of the month."

So clearly, just taking the beer's style and recommending the highest rated beers in that style is fairly useless. Also, while
many beers within a style are pretty similar, some styles (like IPAs or sours) have a broad range of characteristics. Plus, abiding by these defined styles doesn't really make sense in practice, as most brewers do not particularly care what style their beer falls into. It's merely a label of convenience for the consumer. 

Another flaw in most recommendation systems is their propensity to use subjective ratings to make suggestions. In the previous
example, Zombie Dust is made on a fairly small scale. It has been shown that rarer beers tend to receive higher ratings [(source)](http://punchdrink.com/articles/why-are-beer-geeks-obsessed-with-beer-ratings-beeradvocate-ratebeer-untappd-rankings/). 
Even though I fancy myself as a fairly objective reviewer, I'm just as susceptible to this phenomenon as anyone else. I feel comfortable enough in my sanity to freely admit that I waited in line nine hours to try Russian River's Pliny the Younger. It was absolutely amazing, but I also had just spent nine hours convincing myself that it would be worth it.

To combat these flaws in previous systems, I decided to use natural language processing of beer reviews to find similarity of language used to describe beers. I figured using the words that people use to describe beer would give better results than arbitrary scores or styles.

The recommender, which can be found at [ChooseABeerFor.me](http://chooseabeerfor.me), currently uses 700,000 reviews from 20,000 beers. I applied tf-idf to each of these reviews to upweight words used for particular beers and to downweight words used for many beers. This accounts for the fact that almost every beer review mentions things like malt or hops, but few mention more descriptive words like citrus or tart. Then, I applied latent semantic analysis to reduce my feature space to 500 dimensions. Taking this dataset,
I applied cosine similarity between each of the documents to find the five beers with reviews that have the most similar language. This was all put into a Flask app that is currently hosted on AWS. 

![Data Flow](/images/beer_recommender.png)

This first iteration is already producing very good results. If I search for a beer like Lost Abbey's Track #8 (a bourbon barrel
aged Belgian quadrupel with cinnamon sticks and dried chilis) you get the following results:

Beer | Description
--- | ---
Barrel-Aged Framinghammer Mole | Baltic Porter aged in bourbon barrels with cinnamon, cocoa nibs, and ancho chilis
Bourbon Barrel Aged Buffalo Sweat With Vanilla Beans & Cinnamon | Bourbon barrel stout with vanilla and cinnamon
Pump[KY]n | Bourbon barrel pumpkin ale spiced with nutmeg, cinnamon, allspice and cloves
Heavy Seas - Great'ER Pumpkin | Another barrel aged pumpkin ale
Bourbon Barrel Aged Imperial Mayan Mocha | Imperial stout aged in bourbon barrels with coffee, cinnamon, nutmeg and habaneros

This example is important because Track #8 is a beer with no truly defined style. Its base beer is a Belgian Quadrupel, but the added ingredients make it distinct from classic examples of the style. This is reflected in the results, as the recommended beers represent four different defined styles of beer. However, they're all similar in that they are all bourbon barrel aged beers with spice.

Let's try a beer from a fairly broad style, like American sours. I'll take one of my personal favorites, Sante Adairius' West Ashley
(a sour saison aged in Pinot Noir barrels with apricots):

Beer | Description
--- | ---
Cantillon Fou' Foune | A Belgian lambic with apricots
Cascade Apricot Ale | Another apricot sour
Aurelian Lure | Yet another apricot sour
Map Of The Sun | A golden sour with apricots from a similar brewery (The Rare Barrel)
Apricot bu | Berliner Weisse with apricots

Clearly, the apricot addition is a common theme amongst these beers. Also, all of the beers are sours, but there is still
representation from three different sour styles in the recommendations. In contrast, other beer sites can't decide on whether West Ashley is a Saison or an American Wild Ale and just recommend highly rated beers from those styles.

Just to toss out a little more domain knowledge, there's two main flavor components in most IPAs nowadays. This is because modern hops tend to feature a very fruity (citrus, tropical) character, or a piney and resinous character (hops are closely related to 
cannabis, a recreational drug used by recidivists and jazz musicians). So will the model be able to detect the difference between
these two fairly distinct styles?

Let's try Sixpoint's Resin IPA, which, as you may be able to tell, falls into the pine category:

Beer | Marijuana Euphemism Used in a Review
--- | ---
Industrial IPA | musty, resiny
Lupulin Maximus Imperial IPA | green and herbal notes
Odyssey Imperial IPA | weed-like
471 IPA | grassy
Whoop Pass IPA | earthy, floral

All kidding aside, these five beers are similar to Resin IPA in that they fall in to the East Coast interpretation of the style. They are all caramel heavy with an emphasis on bitterness rather than hop flavor.

And now in contrast, let's try Deschutes' Fresh Squeezed IPA, a more fruity example of the style:

Beer | Fruit descriptors
--- | ---
Limbo IPA | tropical, mango
Victory Harvest Ale | juicy, fresh, grapefruit
Space Dust IPA | juicy, grapefruit, mango
Lunch | grapefruit, mango, citrus
Broo Doo | tropical, fresh, grapefruit

These five beers are good examples of the other side of IPAs, drier beers with more emphasis on the bold fruit character of newly bred hops. There is an emphasis on freshness, as these volatile aromatics disappear quickly (which is why you should never drink an old IPA). 

Though my model does produce consistently good results, it is not without its flaws. It still recommends beers that you may not be able to obtain without some difficulty. To fix this, in a future update, I will integrate Untappd's API to filter for beers that have been checked in near your location. I'd also like to integrate this model with BeerMenus or TapHunter to choose a beer available at a particular establishment that is similar to something you know you like. 

Be sure to check it out at [ChooseABeerFor.me](http://chooseabeerfor.me) and let me know what you think!
