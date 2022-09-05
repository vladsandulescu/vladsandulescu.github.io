---
layout: single
title: "Predicting what user reviews are about with LDA and gensim"
modified: 2014-09-09 21:07:06 +0200
tags: [topic modeling,lda,reviews,gensim,yelp]
---

I was rather impressed with the impressions and feedback I received for my [Opinion phrases](http://www.vladsandulescu.com/opinion-phrases/) prototype - code repository [here](https://github.com/vladsandulescu/phrases). So yesterday, I have decided to rewrite my previous post on topic prediction for short reviews using Latent Dirichlet Analysis and its implementation in [gensim](http://radimrehurek.com/gensim/ "Gensim").

I have previously worked with topic modeling for my [MSc thesis](http://www.vladsandulescu.com/opinion-spam-detection-through-semantic-similarity/) but there I used the [Semilar toolkit](http://www.semanticsimilarity.org/) and a looot of C# code. Having read many articles about gensim, I was itchy to actually try it out. 

**Why would we be interested in extracting topics from reviews?**

It is becoming increasingly difficult to handle the large number of opinions posted on review platforms and at the same time offer this information in a useful way to each user so he or she can make a decision fast whether to buy the product or not. Topic-based aggregations and short review summaries are used to group and condense what other users think about the product in order to personalize the content served to a new user and shorten the time he needs to make a buying decision.

A short example always works best. Suppose a review says:
*The mailing pack that was sent to me was very thorough and well explained,correspondence from the shop was prompt and accurate,I opted for the cheque payment method which was swift in getting to me. All in all, a fast efficient service that I had the upmost confidence in,very professionally executed and I will suggest you to my friends when there mobiles are due for recycling :-)*

Some of the topics that could come out of this review could be *delivery*, *payment method* and *customer service*.

In short, knowing what the review talks helps automatically categorize and aggregate on individual keywords and aspects mentioned in the review, assign aggregated ratings for each aspect and personalize the content served to a user. Or simply calculate the efficiency of each of the departments in a company by what people write in their reviews - in this example, the guys in the customer service department as well as the delivery guys would be pretty happy. 

**What do I need to run the code?**

You can clone the [repository](https://github.com/vladsandulescu/topics) and play with the Yelp's dataset which contains many reviews or use your own short document dataset and extract the LDA topics from it.

Get the [Yelp academic dataset](http://www.yelp.com/dataset_challenge) and import the reviews from the json file into your local MongoDB by running the **yelp/yelp-reviews.py** file. Use [MongoDB](http://www.mongodb.org/), take my word for it, you'll never write to a text file ever again! You will also need [PyMongo](http://api.mongodb.org/python/current/), [NLTK](http://www.nltk.org/), [NLTK data](http://www.nltk.org/data.html) (in Python run *import nltk*, then *nltk.download()*). I personally have these Corpora modules installed: Brown Corpus, SentiWordNet, WordNet, as well as the following Models: Treebank Part of Speech Tagger (HMM), Treebank Part of Speech Tagger (Maximum Entropy), Punkt Tokenizer Models. 
Finally, don't forget to install  [gensim](http://radimrehurek.com/gensim/ "Gensim").

OK, enough foreplay, this is how the code works. Skip to the results if you are not interested in running the prototype.

**How does the prototype work?**

Well, the main goal of the prototype of to try to extract topics from a large reviews corpus and then predict the topic distribution for a new unseen review. Please read this[ paper](http://www.yelp.com/html/pdf/YelpDatasetChallengeWinner_ImprovingRestaurants.pdf) first, before checking out the source code, as I have followed it rather closely and tried to reproduce their results. These guys won a prize in the Yelp dataset challenge and in order for me to check if I get similar results, I also experimented on the [Yelp academic dataset](http://www.yelp.com/dataset_challenge). 

Future plans include trying out the prototype on [Trustpilot](http://www.trustpilot.com) reviews, when we will open up the Consumer APIs to the world. I plan to do another blog post then, when I will explain how you can run the prototype on top of the [Trustpilot API](https://github.com/trustpilot/developers) and get nice results from it.

If you clone the repository, you will see a few python files which make up the execution pipeline: yelp/yelp-reviews.py, reviews.py, corpus.py, train.py, display.py and predict.py. I have not yet made a main class to run the entire prototype, as I expect people might want to tweak this pipeline in a number of ways. For example, some may prefer a corpus containing more than just nouns, or avoid writing to Mongo, or keep more than 10000 words, or use more/less than 50 topics and so on. 

You should just run these following files in order. 

* **yelp/yelp-reviews.py** - gets the reviews from the json file and imports them to MongoDB in a collection called *Reviews*

* **reviews.py**/ **reviews_parallel.py** - loops through all the reviews in the initial dataset and for each review it: splits the review into sentences, removes stopwords, extracts parts-of-speech tags for all the remaining tokens, stores each review, i.e. reviewId, business name, review text and (word,pos tag) pairs vector to a new MongoDB database called *Tags*, in a collection called *Reviews*. If you have many reviews, try running **reviews_parallel.py**, which uses the Python multiprocessing features to parallelize this task and use multiple processed to do the POS tagging.

* **corpus.py** - loops through all the reviews from the new MongoDB collection in the previous step, filters out all words which are not nouns, uses WordNetLemmatizer to lookup the lemma of each noun, stores each review together with nouns' lemmas to a new MongoDB collection called *Corpus*.

* **train.py** - feeds the reviews corpus created in the previous step to the gensim LDA model, keeping only the 10000 most frequent tokens and using 50 topics.

* **display.py** - loads the saved LDA model from the previous step and displays the extracted topics.

* **predict.py** - given a short text, it outputs the topics distribution. Simply lookout for the highest weights on a couple of topics and that will basically give the "basket(s)" where to place the text.

* **stopwords.txt** - [stopwords list](http://www.lextek.com/manuals/onix/stopwords2.html) created by Gerard Salton and Chris Buckley for the experimental SMART information retrieval system at Cornell University

**The resulting topics**

POS tagging the entire review corpus and training the LDA model takes considerable time, so expect to leave your laptop running over night while you dream of phis and thetas. It took ~10h on my personal laptop (Lenovo T420s with Intel i5 inside and 8GB of RAM) to do POS tagging for all 1,125,458 Yelp reviews (used reviews_parallel.py for this). I ran the LDA model for 50 topics, but feel free to choose more.

Here were the resulting 50 topics, ignore the bold words written in parenthesis for now:

**0: (food or sauces or sides)** 0.028*sauce + 0.019*meal + 0.018*meat + 0.017*salad + 0.016*food + 0.015*menu + 0.015*side + 0.015*flavor + 0.013*dish + 0.012*pork
**1: (breakfast)** 0.122*egg + 0.096*breakfast + 0.065*bacon + 0.064*juice + 0.033*sausage + 0.032*fruit + 0.024*morning + 0.023*brown + 0.023*strawberry + 0.022*crepe
**2: (restaurant owner)** 0.074*owner + 0.073*year + 0.048*family + 0.032*business + 0.029*company + 0.028*day + 0.026*month + 0.025*time + 0.024*home + 0.021*daughter
**3: (terrace or surroundings)** 0.065*park + 0.030*air + 0.028*management + 0.027*dress + 0.027*child + 0.026*parent + 0.025*training + 0.024*fire + 0.020*security + 0.020*treatment
**4: (seafood)** 0.091*shrimp + 0.090*crab + 0.077*lobster + 0.060*seafood + 0.054*nail + 0.042*salon + 0.039*leg + 0.033*coconut + 0.032*oyster + 0.031*scallop
**5: (thai food)** 0.055*soup + 0.054*rice + 0.045*roll + 0.036*noodle + 0.032*thai + 0.032*spicy + 0.029*bowl + 0.028*chicken + 0.026*dish + 0.023*beef
**6: (cafe)** 0.086*sandwich + 0.063*coffee + 0.048*tea + 0.026*place + 0.018*cup + 0.016*market + 0.015*cafe + 0.015*bread + 0.013*lunch + 0.013*order
**7: (service)** 0.068*food + 0.049*order + 0.044*time + 0.042*minute + 0.038*service + 0.034*wait + 0.030*table + 0.029*server + 0.024*drink + 0.024*waitress
**8: (dessert)** 0.078*cream + 0.071*ice + 0.059*flavor + 0.056*dessert + 0.049*cake + 0.039*chocolate + 0.021*sweet + 0.015*butter + 0.014*taste + 0.013*apple
**9: (greek food)** 0.052*topping + 0.039*yogurt + 0.034*patty + 0.033*hubby + 0.026*flavor + 0.026*sample + 0.024*gyro + 0.022*sprinkle + 0.021*coke + 0.020*greek
**10: (service)** 0.055*time + 0.037*job + 0.032*work + 0.026*hair + 0.025*experience + 0.024*class + 0.020*staff + 0.020*massage + 0.018*day + 0.017*week
**11: (mexican food)** 0.131*chip + 0.081*chili + 0.071*margarita + 0.056*fast + 0.031*dip + 0.030*enchilada + 0.026*quesadilla + 0.026*gross + 0.024*bell + 0.020*pastor
**12: (price)** 0.082*money + 0.046*% + 0.042*tip + 0.040*buck + 0.040*ticket + 0.037*price + 0.033*pay + 0.029*worth + 0.027*cost + 0.024*ride
**13: (location or not sure)** 0.061*window + 0.058*soda + 0.056*lady + 0.037*register + 0.031*ta + 0.030*man + 0.028*haha + 0.026*slaw + 0.020*secret + 0.018*wet
**14: (italian food)** 0.144*pizza + 0.038*wing + 0.031*place + 0.029*sauce + 0.026*cheese + 0.023*salad + 0.021*pasta + 0.019*slice + 0.016*brisket + 0.015*order
**15: (family place or drive-in)** 0.157*car + 0.150*kid + 0.030*drunk + 0.028*oil + 0.026*truck + 0.024*fix + 0.021*college + 0.016*vehicle + 0.016*guy + 0.013*arm
**16: (bar or sports bar)** 0.196*beer + 0.069*game + 0.049*bar + 0.047*watch + 0.038*tv + 0.034*selection + 0.033*sport + 0.017*screen + 0.017*craft + 0.014*playing
**17: (hotel or accommodation)** 0.134*room + 0.061*hotel + 0.044*stay + 0.036*pool + 0.027*view + 0.024*nice + 0.020*gym + 0.018*bathroom + 0.016*area + 0.015*night
**18: (restaurant or atmosphere)** 0.073*wine + 0.050*restaurant + 0.032*menu + 0.029*food + 0.029*glass + 0.025*experience + 0.023*service + 0.023*dinner + 0.019*nice + 0.019*date
**19: (not sure)** 0.052*son + 0.027*trust + 0.025*god + 0.024*crap + 0.023*pain + 0.023*as + 0.021*life + 0.020*heart + 0.017*finish + 0.017*word
**20: (location or not sure)** 0.057*mile + 0.052*arizona + 0.041*theater + 0.037*desert + 0.034*middle + 0.029*island + 0.028*relax + 0.028*san + 0.026*restroom + 0.022*shape
**21: (club or nightclub)** 0.064*club + 0.063*night + 0.048*girl + 0.037*floor + 0.037*party + 0.035*group + 0.033*people + 0.032*drink + 0.027*guy + 0.025*crowd
**22: (brunch or lunch)** 0.171*wife + 0.071*station + 0.058*madison + 0.051*brunch + 0.038*pricing + 0.025*sun + 0.024*frequent + 0.022*pastrami + 0.021*doughnut + 0.016*gas
**23: (casino)** 0.212*vega + 0.103*la + 0.085*strip + 0.047*casino + 0.040*trip + 0.018*aria + 0.014*bay + 0.013*hotel + 0.013*fountain + 0.011*studio
**24: (service)** 0.200*service + 0.092*star + 0.090*food + 0.066*place + 0.051*customer + 0.039*excellent + 0.035*! + 0.030*time + 0.021*price + 0.020*experience
**25: (pub or fast-food)** 0.254*dog + 0.091*hot + 0.026*pub + 0.023*community + 0.022*cashier + 0.021*way + 0.021*eats + 0.020*york + 0.019*direction + 0.019*root
**26: (not sure)** 0.087*box + 0.040*adult + 0.028*dozen + 0.027*student + 0.026*sign + 0.025*gourmet + 0.018*decoration + 0.018*shopping + 0.017*alot + 0.016*eastern
**27: (bar)** 0.120*bar + 0.085*drink + 0.050*happy + 0.045*hour + 0.043*sushi + 0.037*place + 0.035*bartender + 0.023*night + 0.019*cocktail + 0.015*menu
**28: (italian food)** 0.029*chef + 0.027*tasting + 0.024*grand + 0.022*caesar + 0.021*amazing + 0.020*linq + 0.020*italian + 0.018*superb + 0.016*garden + 0.015*al
**29: (not sure)** 0.064*bag + 0.061*attention + 0.040*detail + 0.031*men + 0.027*school + 0.024*wonderful + 0.023*korean + 0.023*found + 0.022*mark + 0.022*def
**30: (mexican food)** 0.122*taco + 0.063*bean + 0.043*salsa + 0.043*mexican + 0.034*food + 0.032*burrito + 0.029*chip + 0.027*rice + 0.026*tortilla + 0.021*corn
**31:** 0.096*waffle + 0.057*honey + 0.034*cheddar + 0.032*biscuit + 0.030*haze + 0.025*chicken + 0.024*cozy + 0.022*let + 0.022*bring + 0.021*kink
**32:** 0.033*lot + 0.027*water + 0.027*area + 0.027*) + 0.025*door + 0.023*( + 0.021*space + 0.021*parking + 0.017*people + 0.013*thing
**33:** 0.216*line + 0.054*donut + 0.041*coupon + 0.030*wait + 0.029*cute + 0.027*cooky + 0.024*candy + 0.022*bottom + 0.019*smoothie + 0.018*clothes
**34:** 0.090*phoenix + 0.077*city + 0.042*downtown + 0.037*gem + 0.026*seating + 0.025*tourist + 0.022*convenient + 0.021*joke + 0.020*pound + 0.017*tom
**35:** 0.072*lol + 0.056*mall + 0.041*dont + 0.035*omg + 0.034*country + 0.030*im + 0.029*didnt + 0.028*strip + 0.026*real + 0.025*choose
**36:** 0.159*place + 0.036*time + 0.026*cool + 0.025*people + 0.025*nice + 0.021*thing + 0.021*music + 0.020*friend + 0.019*'m + 0.018*super
**37:** 0.138*steak + 0.068*rib + 0.063*mac + 0.039*medium + 0.026*bf + 0.026*side + 0.025*rare + 0.021*filet + 0.020*cheese + 0.017*martini
**38:** 0.075*patio + 0.064*machine + 0.055*outdoor + 0.039*summer + 0.038*smell + 0.032*court + 0.032*california + 0.027*shake + 0.026*weather + 0.023*pretzel
**39:** 0.124*card + 0.080*book + 0.079*section + 0.049*credit + 0.042*gift + 0.040*dj + 0.022*pleasure + 0.019*charge + 0.018*fee + 0.017*send
**40:** 0.081*store + 0.073*location + 0.049*shop + 0.039*price + 0.031*item + 0.025*selection + 0.023*product + 0.023*employee + 0.023*buy + 0.020*staff
**41:** 0.048*az + 0.048*dirty + 0.034*forever + 0.033*pro + 0.032*con + 0.031*health + 0.027*state + 0.021*heck + 0.021*skill + 0.019*concern
**42:** 0.037*time + 0.028*customer + 0.025*call + 0.023*manager + 0.023*day + 0.020*service + 0.018*minute + 0.017*phone + 0.017*guy + 0.016*problem
**43:** 0.197*burger + 0.166*fry + 0.038*onion + 0.030*bun + 0.022*pink + 0.021*bacon + 0.021*cheese + 0.019*order + 0.018*ring + 0.015*pickle
**44:** 0.069*picture + 0.052*movie + 0.052*foot + 0.034*vip + 0.031*art + 0.030*step + 0.024*resort + 0.022*fashion + 0.021*repair + 0.020*square
**45:** 0.054*sum + 0.043*dim + 0.042*spring + 0.034*diner + 0.032*occasion + 0.029*starbucks + 0.025*bonus + 0.024*heat + 0.022*yesterday + 0.021*lola
**46:** 0.071*shot + 0.041*slider + 0.038*met + 0.038*tuesday + 0.032*doubt + 0.023*monday + 0.022*stone + 0.022*update + 0.017*oz + 0.017*run
**47:** 0.152*show + 0.050*event + 0.046*dance + 0.035*seat + 0.031*band + 0.029*stage + 0.019*fun + 0.018*time + 0.015*scene + 0.014*entertainment
**48:** 0.099*yelp + 0.094*review + 0.031*ball + 0.029*star + 0.028*sister + 0.022*yelpers + 0.017*serf + 0.016*dream + 0.015*challenge + 0.014*'m
**49:** 0.137*food + 0.071*place + 0.038*price + 0.033*lunch + 0.027*service + 0.026*buffet + 0.024*time + 0.021*quality + 0.021*restaurant + 0.019*eat

All right, they look pretty cohesive, which is a good sign. Now comes the manual topic naming step where we can assign one representative keyword to each topic. This is useful when predicting the topics of new unseen reviews. 
I have suggested some keywords based on my instant inspiration, which you can see in the round parenthesis. I got bored after half of them, but I feel I made the point. You only need to set these keywords once and *summarize* each topic. I suggested some keywords while watching over the Kung Pao Chicken and having a beer...so my keywords may not match yours! Anyway, you get the idea.

It's up to you how you choose the keywords: you can be broader or more precise about what you are interested in the topic, select the most frequent word in the topic and setting that as the keywords, etc..

**Predicting the topics of new unseen reviews**

OK, now that we have the topics, let's see how the model predicts the topics distribution for a new review:

*It's like eating with a big Italian family. Great, authentic Italian food, good advice when asked, and terrific service. With a party of 9, last minute on a Saturday night, we were sat within 15 minutes. The owner chatted with our kids, and made us feel at home. They have meat-filled raviolis, which I can never find. The Fettuccine Alfredo was delicious. We had just about every dessert on the menu. The tiramisu had only a hint of coffee, the cannoli was not overly sweet and they had this custard with wine that was so strangely good. It was an overall great experience!*

The output of the **predict.py** file given this review is: [(0, 0.063979336376367435), (2, 0.19344804518265865), (6, 0.049013217061090186), (7, 0.31535985308065378), (8, 0.074829314265223476), (14, 0.046977300077683241), (15, 0.044438343698184689), (18, 0.09128157138884592), (28, 0.085020844956249786)]

Thus, the review is characterized mostly by topics 7 (32%) and 2 (19%). The missing topics, such as 1, 3, 4, 5 and so on are all zero, that's why they are missing.
Well, what do you know, those topics are about the **service** and **restaurant owner**.

Another one:
*Either the quality has gone down or my taste buds have higher expectations than the last time I was here (about 2 years ago). Now that SF has so many delicious Italian choices where the pasta is made in-house/homemade, it was tough for me to eat the store-bought pasta.  The pasta lacked texture and flavor, and even the best sauce couldn't change my disappointment.  The gnocchi tasted better, but I just couldn't get over how cheap the pasta tasted.  I discovered another spot in North Beach called X and I was really impressed with their pasta, so that's my new go-to spot.*

Distribution: [(2, 0.049949761363727557), (14, 0.67415587326751736), (28, 0.14795291772795682), (33, 0.044461283686581303), (44, 0.044349729171608801)]

Clearly, the review is about topic 14, which is **italian food**.

Third time's the charm:
*Really superior service in general; their reputation precedes them and they deliver. It can indeed be tough to get seating, but I find them willingly accommodating when they can be, and seating at the bar can be really enjoyable, actually. So many wonderful items to choose from, but don't forget to save room for the over-the-top chocolate souffle; elegant and wondrous. Oh and hello, roast Maine lobster, mini quail and risotto with dungeness crab. De-lish.*

[(0, 0.12795812236631765), (4, 0.25125769311344842), (8, 0.097887323141830185), (17, 0.15090844416208612), (24, 0.12415345702622631), (27, 0.067834960190092219), (35, 0.06375000000000007), (41, 0.06375000000000007)]

The topics predicted are topic 4 - **seafood** and topic 24 - **service**. Right on the money again. I just picked the first couple of topics but these can be selected based on their distribution, i.e. taking all above a set threshold.

It isn't generally this sunny in Denmark though... Take a closer look at the topics and you'll notice some are hard to summarize and some are overlapping. This is where a bit of LDA tweaking can improve the results.

While this method is very simple and very effective, it still needs some polishing, but that is beyond the goal of the prototype. LDA is however one of the main techniques used in the industry to categorize text and for the most simple review tagging, it may very well be sufficient.

