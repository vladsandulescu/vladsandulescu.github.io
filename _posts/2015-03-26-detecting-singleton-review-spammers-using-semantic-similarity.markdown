---
layout: single
title: "Detecting Singleton Review Spammers Using Semantic Similarity"
date: 2015-03-26T22:08:26+01:00
---

My paper "Detecting Singleton Review Spammers Using Semantic Similarity" with [Martin Ester](http://www.cs.sfu.ca/~ester/) was accepted at [Rumors and Deception in Social Media workshop](http://www.pheme.eu/events/rdsm2015/) at [WWW 2015](http://www.www2015.it/). I will present the work at WWW this year in Florence.

Check out the abstract and view the [paper](https://arxiv.org/abs/1609.02727).

**ABSTRACT**

Online reviews have increasingly become a very important resource for consumers when making purchases. Though it is becoming more and more difficult for people to make well-informed buying decisions without being deceived by fake reviews. Prior works on the opinion spam problem mostly considered classifying fake reviews using behavioral user patterns. They focused on prolific users who write more than a couple of reviews, discarding one-time reviewers. The number of singleton reviewers however is expected to be high for many review websites. While behavioral patterns are effective when dealing with elite users, for one-time reviewers, the review text needs to be exploited. In this paper we tackle the problem of detecting fake reviews written by the same person using multiple names, posting each review under a different name. We propose two methods to detect similar reviews and show the results generally outperform the vectorial similarity measures used in prior works. The first method extends the semantic similarity between words to the reviews level. The second method is based on topic modeling and exploits the similarity of the reviews topic distributions using two models: bag-of-words and bag-of-opinion-phrases. The experiments were conducted on reviews from three different datasets: Yelp (57K reviews), Trustpilot (9K reviews) and Ott dataset (800 reviews).
