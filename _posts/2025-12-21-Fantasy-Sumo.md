---
layout: post
title: Moneyball for Sumo: Predicting Tournament Winners with Python
---

Fantasy sports have always been a game of statistics and gut feelings. But what if you could bring some serious data science to the dohyo? This project is an attempt to do just that: build a machine learning pipeline to predict the performance of sumo wrestlers in a fantasy league.

The goal is to go beyond simple win/loss records and create a sophisticated tool that gives a real edge in picking a winning fantasy stable.

## The Data: More Than Just Big Guys Pushing

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

It includes:
*   **Sortable Columns:** Click any header to sort the data.
*   **Division Filters:** Instantly switch between viewing all wrestlers, just the top Makuuchi division, or just the Juryo division.
*   **Color-Coded Groups:** Wrestlers are highlighted by their fantasy draft group (e.g., Yokozuna/Ozeki, Maegashira 1-5) for quick scanning.
*   **Detailed Fantasy Points:** Instead of just a final score, the report breaks down the prediction into its components: points from wins, kachi-koshi, kinboshi, and even the *probability-adjusted* points for winning the championship (Yusho).

By combining deep domain knowledge with modern machine learning techniques, this project transforms raw sumo data into a powerful and insightful prediction engine.
