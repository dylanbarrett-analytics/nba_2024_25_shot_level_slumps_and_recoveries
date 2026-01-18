# NBA 2024-25: Slump Shots vs Recovery Shots

---

## **Introduction**

When a player is in a shooting slump, a TV broadcast analyst will often say something along the lines of:

> "I'd like to see him get closer to the basket."

Due to the offensive system, the opposing defense's strategy, and spacing in general, this is easier said than done; however, it still raises the question:

> Do players actually take closer shots when they recover from shooting slumps?

This study tests this idea using shot-by-shot data.

---

## **Table of Contents**

1. [Introduction](#introduction)
2. [About the Data](#about-the-data)
3. [Project Goals](#project-goals)
4. [Tools Used](#tools-used)
5. [Project Files](#project-files)
6. [Notebook 01: Data Acquisition](#notebook-01-data-acquisition)
7. [Notebook 02: Data Cleaning & Preparation](#notebook-02-data-cleaning--preparation)
8. [Notebook 03: Slump & Recovery Detection](#notebook-03-slump--recovery-detection)
9. [Notebook 04: Shot-Level Analysis](#notebook-04-shot-level-analysis)
10. [Tableau Dashboard](#tableau-dashboard)
11. [Conclusion](#conclusion)

---

## **About the Data**

This analysis uses shot-by-shot (i.e., shot-level) data from the 2024-25 NBA regular season. This was sourced from the **NBA API** (via Python), the official data interface that powers statistics on NBA.com.

All shots were processed chronologically within each **player-game**.

> "Player-game" refers to a single player's shot attempts within one game. For example, if a player takes 16 shots in a game, those 16 shots form one *player-game* sequence.

---

## **Project Goals**

1. **Identify shooting slumps and recoveries** at shot-by-shot level.
2. **Measure how shot selection changes** from slump → recovery.

---

## **Tools Used**

- ![Python](https://img.shields.io/badge/Python-orange) (via **Jupyter Notebook**): Data loading, cleaning, merging, and analysis
- ![Tableau](https://img.shields.io/badge/Tableau-blue) Dashboard design and final visualizations

---

## **Project Files** (Fix definitions)

### **Jupyter Notebooks**
- [`01_acquire_shot_level_data.ipynb`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/notebooks/01_acquire_shot_level_data.ipynb)
Raw collection (via NBA API) of 2024-25 shot-by-shot data
- [`02_clean_shot_level_data.ipynb`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/notebooks/02_clean_shot_level_data.ipynb)
Cleaning and consolidation of 2024-25 shot-by-shot data
- [`03_detect_slumps_recoveries.ipynb`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/notebooks/03_detect_slumps_recoveries.ipynb)
Includes the state logic that classifies every shot as either "neutral", "slump", or "recovered" based on recent shot outcomes
- [`04_shot_level_analysis.ipynb`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/notebooks/04_shot_level_analysis.ipynb)
Analysis of how shot distance and location changes from slump shots to recovery shots

### **CSV Exports**
- [`all_player_shots_slump_or_recovery.csv`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/csv/all_player_shots_slump_or_recovery.csv)
Finalized shot-level (shot-by-shot) database (only "slump" or "recovered" rows, no "neutral" rows)
- [`player_shot_summary_final.csv`](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/csv/player_shot_summary_final.csv)
Player-level metrics that summarize shot distance and location adjustments (from slump → recovery)

### **Documentation**
- `README.md`: Project documentation

---

## **Notebook 01: Data Acquisition**

All 30 NBA team IDs were extracted from the NBA API endpoint `teams`. Then all shot-by-shot data for the 2024-25 regular season was pulled from the endpoint `ShotChartDetail`, using these 30 team IDs as input parameters. This resulted in a shot-level dataset of **219,527 shots**.

---

## **Notebook 02: Data Cleaning & Preparation**

This shot-level dataset was validated through sanity checks, including the confirmation of a correct chronological shot order for each player-game. After typical cleaning steps such as removing nonessential columns and ensuring data types were correct, the dataset was sorted so that this correct shot sequence was present within every player-game.

---

## **Notebook 03: Slump & Recovery Detection**

A shot index was created so that every shot in every player-game was labeled in sequential order.

> For example, if a player-game has 16 shot attempts, those shots are labeled 1, 2, 3, ..., 14, 15, 16.

Next, a **state-based detection function** was applied to every shot from every player-game. Every shot now had a "state" label (or phase label) and there were three possible states for each shot:

> - **Neutral:** the default shot state
> - **Slump:** shots taken while the player is in a shooting slump
> - **Recovered:** the single shot that confirms a recovery from a shooting slump

A **slump state** occurs when a player **misses three consecutive shots (0-for-3)**. While in a slump state, all subsequent shots remain labeled as *slump shots* until a recovery condition is met.

A **recovered state** occurs when a player (in a slump state) either **makes two consecutive shots (2-for-2)** or **makes two of the previous three shots (2-for-3)**. The shot that triggers this recovered state is called the *recovery shot*. Only this particular shot is part of the recovery phase. The very next shot (after a recovery) reverts back to **neutral state**.

> **Note:** A slump does not carry over from one game to the next. If a player enters a slump state and never recovers (in that single game), their shooting state resets to "neutral" when the next game begins.

> **Note:** The first two shots of every game are *neutral shots* because it is not possible to enter a slump until at least three shots occur (i.e., need an 0-for-3 stretch for a slump to trigger).

---

## **Notebook 04: Shot-Level Analysis**

With this state logic implemented, it was now possible to compare slump shots with recovery shots.

### **Average Shot Distance (Global-Level)**

For league-wide averages across 566 total players, **recovery shots were taken from closer distances (11.51 ft)** than slump shots (14.93 ft).

This **average shot distance change of -3.42 ft** confirms that players typically got closer to the basket when they broke out of a slump.

> These average shot distances come from league-wide totals of 43,180 slump shots and 6,398 recovery shots.

#### **Average Shot Distance (Player-Level)**

Before studying player-by-player metrics, a filter was implemented so that **only players with at least 300 made shots** across the season were included. With this, insignificant smaller sample sizes were excluded.

![Top Players - Closer to Basket](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/images/top_players_closer_to_basket.png)

Out of 117 qualified players, **110 players (94%!) got closer to the basket** when recovering from a shooting slump.

![Top Players - Away from Basket](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/images/top_players_away_from_basket.png)

Only **7 players (6%) got farther from the basket** when recovering from a shooting slump, and as seen above, these average shot distance changes were very small.

### **Most Common Shot Zones (Global-Level)**

Across all 566 players:

![Most Common Shot Zones - Slump](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/images/most_common_shot_zones_slump.png)

![Most Common Shot Zones - Recovery](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/images/most_common_shot_zones_recovery.png)

From slump → recovery, the **only shot zone** that showed an **increase in percentage of shots** was the **restricted area** (i.e., the area below the basket). This increase was **14.59 percentage points**, which is significant.

The second largest slump → recovery shift was with Above the Break 3s (i.e., non-corner three-point attempts). This decrease was 8.68 percentage points, which stands out as well.

Overall, these shot zone shifts are further proof that players ultimately got closer to the basket when recovering from shooting slumps.

---

### **Tableau Dashboard**

![Dashboard Screenshot](https://github.com/dylanbarrett-analytics/nba_2024_25_shot_level_slumps_and_recoveries/blob/main/images/nba_2024_25_slump_recovery_shots_dashboard.png)

Looking at the *shot location comparison* visual, notice how, for recovery shots, there is a lot of empty space on the floor between the three-point line and the key... way more space than the slump shot plot. For recovery shots, shots were more concentrated inside the key (especially under the basket).

From slump → recovery, there is a noticeable drop in both long three-point attempts and long two-point attempts.

---

## **Conclusion**

For the 2024-25 NBA regular season, this study confirms that **recoveries from shooting slumps** were associated with a meaningful shift toward **shorter shot distances and locations**. 

This does not guarantee that simply taking closer shots will result in breaking out of a slump, but the results absolutely suggest that **successful recoveries coincide with simpler, higher-percentage shot attempts**.

So when a TV broadcast analyst notices a player struggling with his shot and says something like:

> "I'd like to see him get closer to the basket."

the analytics back up that intuition.

---

## **Tableau Dashboard Link**

🔗 [View the Dashboard on Tableau Public](https://public.tableau.com/app/profile/dylan.barrett1539/viz/NBAShotDistanceChanges--InProgress/Dashboard)

---

<p align="center">
  This study was for exploratory purposes only.
</p>

---

<sub> I would like to use up-to-date statistics from the 2025-26 NBA regular season, but at the same time, I'd prefer to use full-season statistics from last season, so that the sample size is large enough to find the most significant insights.
