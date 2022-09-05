---
layout: single
title: "The winning solution to the KDD Cup 2016 competition - Predicting the future relevance of research institutions"
date: 2016-09-28T10:08:20+01:00
created: 2016-09-28T10:08:20+01:00
updated: 2016-09-28T10:08:20+01:00
---

The ACM SIGKDD Conference on Knowledge Discovery and Data Mining, or short KDD is one of the largest premier academic conference on data science and large scale data mining and machine learning. Many well-known researchers choose to publish their research at [KDD](http://www.kdd.org/kdd2016/) and the conference attracts most of the big names in the machine learning community. The [KDD Cup](https://kddcup2016.azurewebsites.net) challenge is held in conjunction with the conference and every year it gathers many participants competing to win the award. This year the top three teams received $10,000, $6500 and $3500 respectively. More than 500 teams registered for the competition which comprised of three stages, each time-boxed to one month each.

To compete in the KDD Cup is something I have been meaning to do for a few years now. To win it on the first go was simply incredible! This year, the stars aligned just right and I actually got some time to focus on this 3 months long competition. My friend and former colleague Mihai Chiru joined me and we proved once again we make one hell of a good team. The competition progressed great for us and in each phase we improved our predictions and overall rank. 

**The task?**
The research task was to predict the ranking of affiliations (universities, research institutions or companies) based on the number of their accepted full research papers at 8 future academic conferences in 2016. Each research paper is written by a number of authors, each affiliated to an university or company. The authors of a paper receive equal scores (1/number of paper authors) for each accepted paper. Then the score for each of the affiliations is simply the sum of these author scores. The affiliations are ranked according to these scores (also called 'relevance' by the competition organizers) and this represents the predicted ranking in the competition.

**Ok, but how were the teams evaluated?**
The evaluation metric used to rank the teams was NDCG@20, a typical ranking evaluation metric. Basically a team would get a lower score if its ranking was different compared to the true ranking once the accepted full research papers would be published at the chosen conference. You can find more details about how NDCG@20 is computed together with a straightforward example [here](https://kddcup2016.azurewebsites.net/Rules).

**What data did we use?**
Microsoft, the competition organizer offered a free snapshot of the [Microsoft Academic Graph (MAG)](https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/), a heterogeneous graph containing various historical information about many academic papers. From this giant graph we extracted information about all the papers ever published at the chosen conferences, about all their authors and the affiliations the authors belonged to. 

**So how did we do it?**
First things first we looked at our data. We checked for any obvious trends for the accepted papers made by top affiliations to each conference. We consider a top affiliation one which had a large number of accepted papers at a conference in the last five years. The assumption is that for large conferences at least, the top 20 places each year will belong to more prolific affiliations, likely to have participated in the past to the conference. We also choose to focus more on the first 20 places because the evaluation metric used in the competition is NDCG@20. We have explored the dataset in a lot of ways but will only mention the most important finding in this article.

In Figure 1 we plot the number of full research papers accepted at the KDD conference between 2011 and 2015 for the top 20 affiliations. The length of each line maps the range of the number of accepted papers for the affiliation and the mean number of papers is marked by the larger dot on each line. The plot shows the mean number of papers across all years could be a good predictor to how an affiliation will score in the future.

![center](/figs/2016-09-28-the-winning-solution-to-the-kdd-cup-2016/plot-yearly_papers_counts-1.pdf)

*Figure 1: Full range and mean value of the number of accepted full research papers for top 20 affiliations at KDD between 2011 and 2015*

Second, all good predictive models need their trustworthy baseline. In the first phase of the competition we aim to build a solid baseline model. For this, we compute the probabilities that full research papers belong to affiliations, based on their number of accepted papers across all past five years. We then rank the affiliations according to these probabilities and this becomes our baseline model against which we will compare all our other models.

In the second phase and also in the final phase of the competition, we experiment with two classes of models: [mixed models](https://en.wikipedia.org/wiki/Mixed_model) and [gradient boosted decision trees (GBDT)](http://xgboost.readthedocs.io/en/latest/model.html). The former is more interpretable while the latter has more predictive power. We set the relevance of each affiliation as the target of our predictions in whatever models we try. The relevance is, if you remember, the sum of the fractional contributions by all the authors of an affiliation for a conference in a year. You can imagine now our dataset is a large matrix where the columns are our features (I'll explain which features we use in just a bit) and the rows are the observations. Basically each row holds the features values for each conference and each affiliation and each year. 

More data is *always* better, so the next thing we try is to increase the dataset size by using information from more years and conferences related to each of the conferences we are interested in making predictions for. What does 'related' mean? Most researchers publish their work at different conferences. However they specialize in a specific area and so the conferences they publish at have to be more or less similar at least in a few respects. We use authors and keywords from the papers in MAG to cluster similar conferences together. It is a straightforward way to grow the dataset even more. The intuition behind this is the information from related conferences will enforce the patterns discovered by the models, because prolific affiliations are prolific across all conferences they submit to, not just at one of them. We compute the Jaccard similarity for both authors and keywords for any pair of conferences in the MAG. From this, we can determine which conferences are for example most similar to KDD in terms of common authors and common papers’ keywords. We experimented with different numbers of related conferences allows us to expand our training dataset immensely and greatly improve our predictions.

Ok, with the dataset in place comes the fun part: **feature engineering**.
We have experimented with many other features, but will only mention here the ones which worked best for us. We created features meant to capture each affiliation's long and short-term relevance trends:
- Stats of all previous relevance scores (std, sum, mean, median, min, max)
- Previous relevance scores computed in windows from previous year up to 4 years ago
- Stats of previous relevance scores (std, sum, mean, median, min, max) computed in windows from previous year up to 4 years ago
- Drift trend of previous relevance scores
- Exponential weighted moving average of previous relevance scores with estimated smoothing parameter
- Exponential weighted moving average of previous relevance scores, computed with a fixed smoothing parameter

**Dataset+Features+Baseline+Tuning=Profit**
Final step in any competition is to polish your predictions through tuning. In the final phase of the competition the organizers chose 3 well-known conferences for validation: [FSE](http://www.cs.ucdavis.edu/fse2016/), [MM](http://www.acmmm.org/2016/) and [MOBICOM](https://www.sigmobile.org/mobicom/2016/). We search for the features configuration for which the GBDT model gives the best predictions for each of the conferences. We perform a grid search on different combinations of features and numbers of related conferences. Thus the training dataset was of course different for each of the 3 conferences. Although the final feature sets was different overall between the conferences, some of the features do well across all conferences: the exponential smoothing features improved the final predictions for all of them.

Figure 2 shows the corresponding results of the best features configurations for each conference. We used the tuning process to chose the best feature sets for each of the conferences, such that all the scores of the GBDT model are above the probabilities model baseline.

![center](/figs/2016-09-28-the-winning-solution-to-the-kdd-cup-2016/plot-benchmark_simple_probabilities-1.pdf)

*Figure 2: Results for the best configuration of the engineered features*

**Conclusions and tips for ML competitions**
This was my first shot at the KDD Cup competition and it couldn't have ended better. I believe our systematic way to build the models coupled with some careful feature engineering was what in the end set us apart from the other teams. So here are some pointers I have for you to have in mind. You probably read all of them before but they cannot be stressed enough.

- Tidy up your data and explore it! I mean really explore it, plot it like crazy. Spend a lot of time on doing this because you will get that extra intuition to create awesome features. You can squeeze some more performance by tuning your models or stacking them and the re-stacking them and then doing a final ensemble of it all, you know, the [Kaggle](https://www.kaggle.com) way. But well thought out features will get you a much more elegant win. I don't think a single company puts ensembles of 20 models into production.
- Set up your own validation procedure and baseline model. You will compare all your other models with the baseline and this how you will progress. Flooding the leaderboards in order to test your model is going to get you nowhere.
- Try simple models first and move on to more complicated ones gradually. Simple models are interpretable and can help you spot new features.
- Teamwork is very important because it acts as a natural ensemble. You will not get all the ideas yourself. 
- Eyes on the prize! Don't give up no matter how bad you do in the competition, because you will at least learn something new.

That's it.

P.S.1. The approach is fully described in the paper [Predicting the future relevance of research institutions - The winning solution of the KDD Cup 2016](https://arxiv.org/abs/1609.02728). Parts of this article were shamelessly taken from the paper. Some things were only superficially mentioned in this article. So I encourage you to read the entire paper for a full overview of the solution.

P.S.2. This entire article was also published on the [Adform Engineering Blog](http://engineering.adform.com/blog/data-science/the-winning-solution-to-the-kdd-cup-2016-competition-predicting-the-future-relevance-of-research-institutions/)
