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
> 
> 1 from the sekiwake/komusubi ranks
> 
> 1 from maegashira 1-5
> 
> 1 from maegashira 6-10
> 
> 1 from maegashira 11+
> 
> 
> Multiple people can have the same wrestler, but there are limits to keep it interesting. Therefore, when submitting your picks, choose 3 guys from each set.
> 
> Points will be granted in the following ways:
> 
> 1 pt. for each win
> 
> 5 pts. for the championship or yusho
> 
> 3 pts for the runner-up, or jun-yusho
> 
> 3 pts for a special prize, or sansho
> 
> 2 pts for a gold star victory, or kinboshi
> 
> 1 pt. for a winning record, or kachi-koshi
> 
> -.5 pt. for a losing record, or make-koshi
> 
> -.5 pt. for your second trade
> 
> -1 pt. for every trade beyond the first 2
> 

So a relatively simple format. We pick three wrestlers from each of the tiers and they get a score based on how well they do. I haven't really done much trading once the league began automatically trading wrestlers who got hurt, so we'll ignore that for now in this project. Though, it could be an interesting thing to track during the tournaments and would give me a reason to create a live dashboard. But we'll save that for another time.

## The Data

Our predictions are built on a rich dataset scraped from the incredible [SumoDB](https://sumodb.sumogames.de/). SumoDB has data all the way back to the [October 1757 tournament](https://sumodb.sumogames.de/Banzuke.aspx?b=175710) which is absolutely incredible. Now, understandably, this data is not as complete as the modern data, and simply features the Makuuchi and Juryo banzukes. By 1761, SumoDB has final results, and by 1909 it has daily results and matchups. Then by 1927, kimarites (winning techniques) are starting to be added for the top divisions. The most complete data begins in the July 1991 tournament, with 2747 kimarites and bouts recorded. 

The data comes in two main forms:

1.  **Banzuke Data:** This is the official ranking sheet for each tournament. It gives us a wrestler's rank, stable, age, height, and weight. This is our foundational data.
2.  **Match History Data:** This is a detailed, day-by-day log of every match, including the winner, loser, and the *kimarite* (winning technique).

Combining these two sources allows us to build a deep understanding of each wrestler.

## Feature Engineering

Having this rich dataset from SumoDB is great, but in order to properly build a model we need feature engineering.

Here are some of the key features that power the model:

#### 1. Rank Based Features
*   **`absolute_rank_score`**: Absolute rank in the entire sumo structure
*   **`rank_gap`**: The difference between a wrestler's current rank and their career-best rank. This tells us if they're at their peak, in a slump, or on the rise.

#### 2. Momentum is Key
A wrestler's recent form is a huge predictor of future success.
*   **`kachi_koshi_streak`**: How many consecutive tournaments has this wrestler had a winning record? A long streak indicates confidence and form.
*   **`was_kyujo_last_basho`**: A simple flag to show if a wrestler is returning from an injury-related absence, a major performance factor.

#### 3. Context and Competition
No wrestler competes in a vacuum.
*   **`heya_strength`**: We measure the average rank of a wrestler's stablemates. A stronger stable means better training partners.
*   **`division_strength`**: We also measure the average rank of their opponents in the division. This tells the model how tough the competition is for that specific tournament.

#### 4. The Kimarite Features
This is where we get into advanced analytics by using the detailed match history.
*   **`oshi_ratio`**: The ratio of a wrestler's wins by "pushing/thrusting" vs. "grappling/belt" techniques. This creates a "fighter style" profile. Is he a pusher or a grappler?
*   **`avg_h2h_win_pct`**: The most powerful feature of all. For each wrestler, we calculate their historical head-to-head win percentage against every other opponent they will face in the upcoming tournament.

## The "Committee of Experts": A Stacking Model

No single machine learning model is perfect. To get the best results, we use an ensemble technique called **stacking**.

1.  **Base Models:** We first train three powerful and diverse models on the data: **LightGBM**, **XGBoost**, and **Random Forest**. Each one learns the patterns in the data slightly differently.
2.  **Meta-Learner:** We then train a final, simpler model (a Ridge Regressor) whose only job is to learn from the predictions of the three base models. It acts like a "committee chairman," learning how to best weigh the "opinions" of its expert members to make a final, more accurate prediction.

## The Payoff: An Interactive Prediction Report

```
--- Stacking Model Performance ---
  R-squared (R²): 0.1219
  Mean Absolute Error (MAE): 2.0084 wins
-----------------------------------
```

Now sure 0.12 R-squared isn't great, but I think the MAE of 2.0084 wins is definitely usable in this context. If we take these win predictions and use them to find the other fantasy sumo stats like kachi-koshis and kinboshis through probability, we should have a workable cheat seet to draft from.

The final output is a dynamic HTML report that serves as the ultimate fantasy sumo cheat sheet.

Here are the predictions for the upcoming January 2026 tournament:

    <!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><title>Fantasy Sumo Report 202601</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f4f9; color: #333; padding: 20px; }
        .container { max-width: 1200px; margin: auto; background: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1, h2 { text-align: center; color: #444; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 13px; }
        th, td { padding: 8px 10px; border: 1px solid #ddd; text-align: left; }
        th { background-color: #333; color: #fff; cursor: pointer; user-select: none; position: sticky; top: 0; }
        .filters { text-align: center; margin-bottom: 20px; }
        .filters button { padding: 8px 15px; margin: 0 5px; border: 1px solid #ccc; background: #eee; cursor: pointer; border-radius: 4px; }
        .filters button.active { background: #333; color: #fff; border-color: #333; }
        .group-yo { background-color: #ffcccc; } .group-sk { background-color: #ffedcc; }
        .group-m1-5 { background-color: #ffffcc; } .group-m6-10 { background-color: #ccffcc; }
        .group-m11 { background-color: #ccffff; } .group-j { background-color: #e6ccff; }
    </style>
    <script>
    function sortTable(n) {
      var table, rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
      table = document.getElementById("reportTable");
      switching = true; dir = "asc"; 
      while (switching) {
        switching = false; rows = table.rows;
        for (i = 1; i < (rows.length - 1); i++) {
          shouldSwitch = false;
          x = rows[i].getElementsByTagName("TD")[n];
          y = rows[i + 1].getElementsByTagName("TD")[n];
          var xContent = isNaN(parseFloat(x.innerText)) ? x.innerText.toLowerCase() : parseFloat(x.innerText);
          var yContent = isNaN(parseFloat(y.innerText)) ? y.innerText.toLowerCase() : parseFloat(y.innerText);
          if (dir == "asc") { if (xContent > yContent) { shouldSwitch = true; break; } } 
          else if (dir == "desc") { if (xContent < yContent) { shouldSwitch = true; break; } }
        }
        if (shouldSwitch) {
          rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
          switching = true; switchcount ++;      
        } else { if (switchcount == 0 && dir == "asc") { dir = "desc"; switching = true; } }
      }
    }
    function filterDivision(division) {
        var table = document.getElementById("reportTable");
        var tr = table.getElementsByTagName("tr");
        for (var i = 1; i < tr.length; i++) {
            if (division === 'all') tr[i].style.display = "";
            else tr[i].style.display = tr[i].classList.contains('division-' + division) ? "" : "none";
        }
        document.querySelectorAll('.filters button').forEach(btn => btn.classList.remove('active'));
        document.querySelector(`.filters button[onclick="filterDivision('`+division+`')"]`).classList.add('active');
    }
    window.onload = () => { document.querySelector('.filters button[onclick="filterDivision(\'all\')"]').classList.add('active'); };
    </script>
    </head>
    <body><div class="container">
        <h1>Fantasy Sumo Prediction Report</h1>
        <h2>Basho: 202601</h2>
        <div class="filters">
            <button onclick="filterDivision('all')">All</button>
            <button onclick="filterDivision('makuuchi')">Makuuchi</button>
            <button onclick="filterDivision('juryo')">Juryo</button>
        </div>
        <table id="reportTable">
            <thead>
                <tr>
                    <th onclick="sortTable(0)">Rikishi</th>
                    <th onclick="sortTable(1)">Rank</th>
                    <th onclick="sortTable(2)">Total Pts</th>
                    <th onclick="sortTable(3)">Win Pts</th>
                    <th onclick="sortTable(4)">KK/MK Pts</th>
                    <th onclick="sortTable(5)">Yusho Pts</th>
                    <th onclick="sortTable(6)">J-Yusho Pts</th>
                    <th onclick="sortTable(7)">Kinboshi Pts</th>
                    <th onclick="sortTable(8)">Actual Wins</th>
                </tr>
            </thead>
            <tbody>
        <tr class="division-makuuchi group-yo">
            <td>Onosato</td>
            <td>Y1w</td>
            <td><strong>13.89</strong></td>
            <td>10.7</td>
            <td>0.92</td>
            <td>1.77</td>
            <td>0.49</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-yo">
            <td>Aonishiki</td>
            <td>O1w</td>
            <td><strong>12.79</strong></td>
            <td>10.2</td>
            <td>0.87</td>
            <td>1.28</td>
            <td>0.44</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-yo">
            <td>Hoshoryu</td>
            <td>Y1e</td>
            <td><strong>11.57</strong></td>
            <td>9.6</td>
            <td>0.78</td>
            <td>0.81</td>
            <td>0.38</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Yoshinofuji</td>
            <td>M1w</td>
            <td><strong>10.84</strong></td>
            <td>8.1</td>
            <td>0.43</td>
            <td>0.23</td>
            <td>0.18</td>
            <td>1.90</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Asanoyama</td>
            <td>M16e</td>
            <td><strong>9.76</strong></td>
            <td>8.6</td>
            <td>0.56</td>
            <td>0.38</td>
            <td>0.21</td>
            <td>0.01</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Fujinokawa</td>
            <td>M7w</td>
            <td><strong>9.20</strong></td>
            <td>8.2</td>
            <td>0.46</td>
            <td>0.28</td>
            <td>0.17</td>
            <td>0.10</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-yo">
            <td>Kotozakura</td>
            <td>O1e</td>
            <td><strong>9.16</strong></td>
            <td>8.2</td>
            <td>0.46</td>
            <td>0.32</td>
            <td>0.19</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Wakatakakage</td>
            <td>M2w</td>
            <td><strong>9.03</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.11</td>
            <td>0.06</td>
            <td>1.37</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Hiradoumi</td>
            <td>M6e</td>
            <td><strong>8.94</strong></td>
            <td>7.9</td>
            <td>0.37</td>
            <td>0.18</td>
            <td>0.15</td>
            <td>0.34</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Takanosho</td>
            <td>M3e</td>
            <td><strong>8.77</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.11</td>
            <td>0.09</td>
            <td>1.09</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Atamifuji</td>
            <td>M4w</td>
            <td><strong>8.69</strong></td>
            <td>7.4</td>
            <td>0.22</td>
            <td>0.11</td>
            <td>0.07</td>
            <td>0.89</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kotoeiho</td>
            <td>J1e</td>
            <td><strong>8.66</strong></td>
            <td>8.2</td>
            <td>0.46</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Hakunofuji</td>
            <td>M3w</td>
            <td><strong>8.56</strong></td>
            <td>7.1</td>
            <td>0.13</td>
            <td>0.07</td>
            <td>0.06</td>
            <td>1.20</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Kotoshoho</td>
            <td>M10w</td>
            <td><strong>8.31</strong></td>
            <td>7.4</td>
            <td>0.22</td>
            <td>0.12</td>
            <td>0.08</td>
            <td>0.49</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Takerufuji</td>
            <td>J5w</td>
            <td><strong>8.27</strong></td>
            <td>7.9</td>
            <td>0.37</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Roga</td>
            <td>M9w</td>
            <td><strong>8.25</strong></td>
            <td>7.0</td>
            <td>0.10</td>
            <td>0.04</td>
            <td>0.06</td>
            <td>1.05</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Tochitaikai</td>
            <td>J7e</td>
            <td><strong>8.14</strong></td>
            <td>7.8</td>
            <td>0.34</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Hitoshi</td>
            <td>J9e</td>
            <td><strong>8.14</strong></td>
            <td>7.8</td>
            <td>0.34</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kitanowaka</td>
            <td>J8w</td>
            <td><strong>8.14</strong></td>
            <td>7.8</td>
            <td>0.34</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Oshoumi</td>
            <td>M16w</td>
            <td><strong>8.11</strong></td>
            <td>7.6</td>
            <td>0.28</td>
            <td>0.10</td>
            <td>0.11</td>
            <td>0.02</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Shodai</td>
            <td>M8e</td>
            <td><strong>8.02</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.09</td>
            <td>0.09</td>
            <td>0.35</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Shonannoumi</td>
            <td>J4w</td>
            <td><strong>8.01</strong></td>
            <td>7.7</td>
            <td>0.31</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Midorifuji</td>
            <td>M12e</td>
            <td><strong>7.97</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.13</td>
            <td>0.10</td>
            <td>0.25</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Abi</td>
            <td>M12w</td>
            <td><strong>7.92</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.10</td>
            <td>0.09</td>
            <td>0.38</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Asasuiryu</td>
            <td>J7w</td>
            <td><strong>7.88</strong></td>
            <td>7.6</td>
            <td>0.28</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Nishinoryu</td>
            <td>J6w</td>
            <td><strong>7.75</strong></td>
            <td>7.5</td>
            <td>0.25</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Wakanosho</td>
            <td>J11e</td>
            <td><strong>7.75</strong></td>
            <td>7.5</td>
            <td>0.25</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Tamawashi</td>
            <td>M5e</td>
            <td><strong>7.70</strong></td>
            <td>6.9</td>
            <td>0.07</td>
            <td>0.10</td>
            <td>0.05</td>
            <td>0.59</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Onokatsu</td>
            <td>M6w</td>
            <td><strong>7.69</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.07</td>
            <td>0.08</td>
            <td>0.05</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Asahakuryu</td>
            <td>M17e</td>
            <td><strong>7.69</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.11</td>
            <td>0.08</td>
            <td>0.02</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Oshoma</td>
            <td>M7e</td>
            <td><strong>7.69</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.11</td>
            <td>0.09</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-sk">
            <td>Kirishima</td>
            <td>S1e</td>
            <td><strong>7.68</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.12</td>
            <td>0.06</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-sk">
            <td>Oho</td>
            <td>K1e</td>
            <td><strong>7.64</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.07</td>
            <td>0.08</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Daiseizan</td>
            <td>J2e</td>
            <td><strong>7.62</strong></td>
            <td>7.4</td>
            <td>0.22</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Fujiryoga</td>
            <td>J3e</td>
            <td><strong>7.62</strong></td>
            <td>7.4</td>
            <td>0.22</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Kinbozan</td>
            <td>M8w</td>
            <td><strong>7.58</strong></td>
            <td>7.0</td>
            <td>0.10</td>
            <td>0.08</td>
            <td>0.07</td>
            <td>0.33</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Nishikifuji</td>
            <td>M11w</td>
            <td><strong>7.55</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.10</td>
            <td>0.07</td>
            <td>0.03</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Tobizaru</td>
            <td>M13e</td>
            <td><strong>7.54</strong></td>
            <td>7.0</td>
            <td>0.10</td>
            <td>0.09</td>
            <td>0.07</td>
            <td>0.28</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Fujiseiun</td>
            <td>J1w</td>
            <td><strong>7.49</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Dewanoryu</td>
            <td>J13w</td>
            <td><strong>7.49</strong></td>
            <td>7.3</td>
            <td>0.19</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Gonoyama</td>
            <td>M9e</td>
            <td><strong>7.44</strong></td>
            <td>6.9</td>
            <td>0.07</td>
            <td>0.07</td>
            <td>0.05</td>
            <td>0.34</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Ichiyamamoto</td>
            <td>M1e</td>
            <td><strong>7.42</strong></td>
            <td>5.7</td>
            <td>-0.22</td>
            <td>0.03</td>
            <td>0.01</td>
            <td>1.90</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Hatsuyama</td>
            <td>M17w</td>
            <td><strong>7.41</strong></td>
            <td>7.1</td>
            <td>0.13</td>
            <td>0.10</td>
            <td>0.07</td>
            <td>0.02</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Daieisho</td>
            <td>M4e</td>
            <td><strong>7.37</strong></td>
            <td>6.7</td>
            <td>0.02</td>
            <td>0.06</td>
            <td>0.04</td>
            <td>0.56</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kyokukaiyu</td>
            <td>J12w</td>
            <td><strong>7.36</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kotokuzan</td>
            <td>J6e</td>
            <td><strong>7.36</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kazekeno</td>
            <td>J10e</td>
            <td><strong>7.36</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kazuma</td>
            <td>J14w</td>
            <td><strong>7.36</strong></td>
            <td>7.2</td>
            <td>0.16</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Shirokuma</td>
            <td>J3w</td>
            <td><strong>7.23</strong></td>
            <td>7.1</td>
            <td>0.13</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Shishi</td>
            <td>M14e</td>
            <td><strong>7.13</strong></td>
            <td>6.9</td>
            <td>0.07</td>
            <td>0.08</td>
            <td>0.07</td>
            <td>0.01</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kayo</td>
            <td>J9w</td>
            <td><strong>7.10</strong></td>
            <td>7.0</td>
            <td>0.10</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Asakoryu</td>
            <td>M15w</td>
            <td><strong>7.09</strong></td>
            <td>6.8</td>
            <td>0.04</td>
            <td>0.09</td>
            <td>0.04</td>
            <td>0.11</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Tohakuryu</td>
            <td>J10w</td>
            <td><strong>6.97</strong></td>
            <td>6.9</td>
            <td>0.07</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Kagayaki</td>
            <td>J2w</td>
            <td><strong>6.97</strong></td>
            <td>6.9</td>
            <td>0.07</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Chiyoshoma</td>
            <td>M11e</td>
            <td><strong>6.95</strong></td>
            <td>6.6</td>
            <td>-0.01</td>
            <td>0.05</td>
            <td>0.02</td>
            <td>0.29</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Tomokaze</td>
            <td>M13w</td>
            <td><strong>6.89</strong></td>
            <td>6.7</td>
            <td>0.02</td>
            <td>0.06</td>
            <td>0.06</td>
            <td>0.05</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Mitakeumi</td>
            <td>M14w</td>
            <td><strong>6.86</strong></td>
            <td>6.6</td>
            <td>-0.01</td>
            <td>0.05</td>
            <td>0.04</td>
            <td>0.17</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m6-10">
            <td>Tokihayate</td>
            <td>M10e</td>
            <td><strong>6.85</strong></td>
            <td>6.6</td>
            <td>-0.01</td>
            <td>0.04</td>
            <td>0.03</td>
            <td>0.19</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-sk">
            <td>Takayasu</td>
            <td>S1w</td>
            <td><strong>6.81</strong></td>
            <td>6.7</td>
            <td>0.02</td>
            <td>0.05</td>
            <td>0.04</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-sk">
            <td>Wakamotoharu</td>
            <td>K1w</td>
            <td><strong>6.80</strong></td>
            <td>6.7</td>
            <td>0.02</td>
            <td>0.03</td>
            <td>0.06</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m11">
            <td>Ryuden</td>
            <td>M15e</td>
            <td><strong>6.72</strong></td>
            <td>6.6</td>
            <td>-0.01</td>
            <td>0.04</td>
            <td>0.03</td>
            <td>0.06</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Sadanoumi</td>
            <td>J4e</td>
            <td><strong>6.72</strong></td>
            <td>6.7</td>
            <td>0.02</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Hakuyozan</td>
            <td>J14e</td>
            <td><strong>6.59</strong></td>
            <td>6.6</td>
            <td>-0.01</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Tamashoho</td>
            <td>J5e</td>
            <td><strong>6.46</strong></td>
            <td>6.5</td>
            <td>-0.04</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Ura</td>
            <td>M2e</td>
            <td><strong>6.35</strong></td>
            <td>5.8</td>
            <td>-0.20</td>
            <td>0.01</td>
            <td>0.03</td>
            <td>0.71</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Hidenoumi</td>
            <td>J13e</td>
            <td><strong>6.34</strong></td>
            <td>6.4</td>
            <td>-0.06</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Tsurugisho</td>
            <td>J11w</td>
            <td><strong>6.21</strong></td>
            <td>6.3</td>
            <td>-0.09</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Nishikigi</td>
            <td>J12e</td>
            <td><strong>5.84</strong></td>
            <td>6.0</td>
            <td>-0.16</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-makuuchi group-m1-5">
            <td>Churanoumi</td>
            <td>M5w</td>
            <td><strong>5.77</strong></td>
            <td>5.9</td>
            <td>-0.18</td>
            <td>0.03</td>
            <td>0.03</td>
            <td>0.00</td>
            <td>0</td>
        </tr>
        <tr class="division-juryo group-j">
            <td>Meisei</td>
            <td>J8e</td>
            <td><strong>5.48</strong></td>
            <td>5.7</td>
            <td>-0.22</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0</td>
        </tr></tbody>
        </table>
    </div></body></html>

By combining deep domain knowledge with modern machine learning techniques, this project transforms raw sumo data into a powerful and insightful prediction engine for a particular fantasy sumo league.
