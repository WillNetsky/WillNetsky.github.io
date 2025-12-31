---
layout: post
title: Fantasy Sumo Predictions
---

I started watching sumo in 2020. It's been great, it's a fun thing to have on in the background. I've gotten to see the end of the GOAT Hakuho's career, Terunofuji's Yokozuna run, and now the rise of new guys. Other than being drawn to the sound of meat slapping for eight hours straight for two weeks every two months, I also enjoy the different structures of sumo. All of the separate divisions and the East/West standings make it particularly interesting from a data analysis perspective.

Of course, as the title suggests, I've gotten into fantasy sumo.

## The League Format 

> You'll have 5 wrestlers on your team, each from a different rank set:
> 
> 1 from the yokozuna/ozeki ranks
> 1 from the sekiwake/komusubi ranks
> 1 from maegashira 1-5
> 1 from maegashira 6-10
> 1 from maegashira 11+
> 
> Multiple people can have the same wrestler, but there are limits to keep it interesting. Therefore, when submitting your picks, choose 3 guys from each set.
> 
> Points will be granted in the following ways:
> 
> 1 pt. for each win
> 5 pts. for the championship or yusho
> 3 pts for the runner-up, or jun-yusho
> 3 pts for a special prize, or sansho
> 2 pts for a gold star victory, or kinboshi
> 1 pt. for a winning record, or kachi-koshi
> -.5 pt. for a losing record, or make-koshi
> -.5 pt. for your second trade
> -1 pt. for every trade beyond the first 2

So a relatively simple format. We pick three wrestlers from each of the tiers and they get a score based on how well they do. I haven't really done much trading once the league began automatically trading wrestlers who got hurt, so we'll ignore that for now in this project. Though, it could be an interesting thing to track during the tournaments and would give me a reason to create a live dashboard. But we'll save that for another time.

## The Data

Our predictions are built on a rich dataset scraped from the incredible [SumoDB](https://sumodb.sumogames.de/). The data comes in two main forms:

1.  **Banzuke Data:** This is the official ranking sheet for each tournament. It gives us a wrestler's rank, stable, age, height, and weight. This is our foundational data.
2.  **Match History Data:** This is a detailed, day-by-day log of every match, including the winner, loser, and the *kimarite* (winning technique).

Combining these two sources allows us to build a deep understanding of each wrestler.

## The Secret Sauce: Feature Engineering

A machine learning model is only as good as the data it's fed. Raw stats aren't enough. The real magic is in **feature engineering**—creating new, insightful features that capture the nuances of a wrestler's career and abilities.

Here are some of the key features that power the model:

#### 1. Beyond Basic Rank
Rank is everything in sumo, but we treat it with sophistication. Instead of just a number, we calculate:
*   **`absolute_rank_score`**: A single "power score" that represents a wrestler's standing across all six divisions.
*   **`rank_gap`**: The difference between a wrestler's current rank and their career-best rank. This tells us if they're at their peak, in a slump, or on the rise.

#### 2. Momentum is Key
A wrestler's recent form is a huge predictor of future success.
*   **`kachi_koshi_streak`**: How many consecutive tournaments has this wrestler had a winning record? A long streak indicates confidence and form.
*   **`was_kyujo_last_basho`**: A simple flag to show if a wrestler is returning from an injury-related absence, a major performance factor.

#### 3. Context and Competition
No wrestler competes in a vacuum.
*   **`heya_strength`**: We measure the average rank of a wrestler's stablemates. A stronger stable means better training partners.
*   **`division_strength`**: We also measure the average rank of their opponents in the division. This tells the model how tough the competition is for that specific tournament.

#### 4. The "Moneyball" Stats
This is where we get into advanced analytics by using the detailed match history.
*   **`oshi_ratio`**: The ratio of a wrestler's wins by "pushing/thrusting" vs. "grappling/belt" techniques. This creates a "fighter style" profile. Is he a pusher or a grappler?
*   **`avg_h2h_win_pct`**: The most powerful feature of all. For each wrestler, we calculate their historical head-to-head win percentage against every other opponent they will face in the upcoming tournament.

## The "Committee of Experts": A Stacking Model

No single machine learning model is perfect. To get the best results, we use an ensemble technique called **stacking**.

1.  **Base Models:** We first train three powerful and diverse models on the data: **LightGBM**, **XGBoost**, and **Random Forest**. Each one learns the patterns in the data slightly differently.
2.  **Meta-Learner:** We then train a final, simpler model (a Ridge Regressor) whose only job is to learn from the predictions of the three base models. It acts like a "committee chairman," learning how to best weigh the "opinions" of its expert members to make a final, more accurate prediction.

## The Payoff: An Interactive Prediction Report

The final output is a dynamic HTML report that serves as the ultimate fantasy sumo cheat sheet.

[Here are the predictions for the upcoming January 2026 tournament]

By combining deep domain knowledge with modern machine learning techniques, this project transforms raw sumo data into a powerful and insightful prediction engine.
