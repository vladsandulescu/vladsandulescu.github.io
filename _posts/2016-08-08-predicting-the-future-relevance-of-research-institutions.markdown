---
layout: single
title: "We won the KDD Cup 2016!"
date: 2016-08-09T08:08:20+01:00
created: 2016-08-09T08:08:20+01:00
updated: 2016-08-09T08:08:20+01:00
---

**Update:**

Check out the abstract and read the [paper](https://arxiv.org/abs/1609.02728) which fully explains our approach. 

You can read the papers from the top 12 teams which participated in the KDD Cup 2016 [here](https://docs.com/alex-wade/1578/kdd-cup).

**ABSTRACT**

The world's collective knowledge is evolving through research and new scientific discoveries. It is becoming increasingly difficult to objectively rank the impact research institutes have on global advancements. However, since the funding, governmental support, staff and students quality all mirror the projected quality of the institution, it becomes essential to measure the affiliation's rating in a transparent and widely accepted way. We propose and investigate several methods to rank affiliations based on the number of their accepted papers at future academic conferences. We carry out our investigation using publicly available datasets such as the Microsoft Academic Graph, a heterogeneous graph which contains various information about academic papers. We analyze several models, starting with a simple probabilities-based method and then gradually expand our training dataset, engineer many more features and use mixed models and gradient boosted decision trees models to improve our predictions.

-------------------------

To compete in the KDD Cup is something I have been meaning to do for a few years now. To win it on the first go was simply incredible! This year, the stars aligned just right and I actually got some time to focus on this 3 months long competition. My friend and former colleague Mihai Chiru joined me and we proved once again we make one hell of a good team.

The research task was to predict the ranking of affiliations (universities, research institutions or companies) based on the number of their accepted full research papers at various academic conferences in 2016. Microsoft, the competition organizer offered a free snapshot of the Microsoft Academic Graph (MAG), a heterogenous graph containing various historical information about many academic papers.

The competition progressed great for us and in each phase we improved our predictions and overall rank. I believe our systematic way to build the models coupled with some careful feature engineering was what in the end set us apart from the other teams. 

The approach is fully described in the paper [Predicting the future relevance of research institutions - The winning solution of the KDD Cup 2016](https://arxiv.org/abs/1609.02728). I will present the paper in August in San Francisco at the [KDD Cup 2016: Towards measuring the impact of research institutions](https://kddcup2016.azurewebsites.net) workshop part of [KDD 2016](http://www.kdd.org/kdd2016/).
