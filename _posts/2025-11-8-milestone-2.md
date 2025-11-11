---
layout: post
title: IFT6758 Demo Post
use_math: true
---

## Feature Engineering I
Below is a histogram of shot counts by distance separated into two classes, goals and no-goals. From the graph, we can see that most shots are taken close to the net between 0 and 30 ft. The histogram is heavily concentrated to the left, which can be explained by players shooting more often when close to the goal. Furthermore, goals are much more frequent at close distances. The orange bars representing the "goals" are highest within the first 20 ft, meaning shots taken closer to the net are much more likely to score. Following this trend, we can assume that shots from further distance have a lower rate of scoring in the net.

![Screenshot](milestone2_histogram_by_distance.png)

Below is a histogram of shot counts by angle separated into two classes, goals and no-goals. From the graph, we can see that most shots occur at small angles between 0 to 25 degrees from the net. The histogram is heavily concentrated to the left, which can be explained by players overwhelmingly shooting when they are facing the net directly. The goal probability decreases with angle as orange bars drop off very quickly as the angle increases. This means that shots taken from the side or at sharp angles almost never result in goals.

![Screenshot](milestone2_histogram_by_angle.png)

Below is a 2d histogram or heatmap showing the join distribution of hocket shot events by both distance and angle from the net. This figure shows us what we have previously concludedl that most shots occur at small distances and small angles. the bright yellow cluster shows that players overwhelmingly shoot from close to the net and near the centerline. In the heatmap, the further you move along the x and y axis, the less shots you see player take.

![Screenshot](milestone2_2d_histogram_distance_vs_angle.png)

The figure below shows the relationship between shot distance and goal rate and how the likelihood of scoring a goal changes as the shooter moves farther from the net. This graph supports the histogram shown before where there are less goals scored at farther distance. We can see that the goal rate drops quickly by 30 ft and scoring becomes extremely unlikely beyond 40 ft. 

![Screenshot](milestone2_goal_rate_vs_distance.png)

The figure below shows the goal rate as a function of the shooting angle and how the likelihood of scoring a goal changes depending on how far from the centerline a player is when shooting. This graph supports the histogram shown previously where there are less goals scored at a steeper angle from the net. The goal rate is highest between 0 and 5 degrees and sees a decline beyond that range. The goal rate remains consistently low for wider angles (> 20 degrees), meaning shooting from the sides rarely results in goals.

![Screenshot](milestone2_goal_rate_vs_angle.png)

The histogram below plots the distribution of goal distances separated into two classes: empty-net and non-empty net events. As expected, almost all non-empty net goals occur within 0 and 25 ft of the net, with frequency dropping significantly beyond that range. This aligns with domain knowledge that "it is incredibly rare to score a non-empty net goal on the opposing team from within your defence zone". In contrast, empty0net goals are much rarer but can occur from muchgreater distanced, since there is no goalies defending the net. This makes sense, as teams rarely remove their goaltenders during games. After inspection, there are anomalies in the dataset, for example, in game_id = '2017030156', the x/y coordinates for some of the goals are off. as we can see for the last scored goal, A.Hammond scored on the awayTeam during period 3. Which means that the net that he has to score in is located at (-89,0). However, the x/y coordinates indicate (76,5), which means it has to be a goal scored from the defensive zone. However, if you look at the highlights of the game we can see that he scored at around 14 feets. This inconsistency can be explained by the API providing inaccurate or opposite data for the x/y coordinates to what it truly is, maybe because of the different camera angles of broadcasts or they forget to change sides during different periods.

<https://www.nhl.com/gamecenter/col-vs-nsh/2018/04/22/2017030156>

![Screenshot](milestone2_goal_distances_net.png)


## Feature Engineering II

The objective of this phase was to design and construct new features through feature engineering, enhancing the model’s predictive accuracy for goal-related outcomes. Here is the list of the engineered features derived from the NHL play-by-play data.

### Table of Features

| **Feature**                    | **Description**                                                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| `game_id`                      | Unique identifier for each game.                                                                                        |
| `game_seconds`                 | Total number of seconds elapsed since the start of the game.                                                            |
| `game_period`                  | Current period of the game.                                                                                             |
| `x`, `y`                       | Coordinates of the play / shot (x-axis, y-axis)                                                                         |
| `shot_distance`                | Euclidean distance between the shot event and the nearest net (RINK UNIT)                                               |
| `shot_angle`                   | Shot angle relative to the nearest net. (DEG)                                                                           |
| `shot_type`                    | Type of shot.                                                                                                           |
| `last_event_type`              | Type of the previous play.                                                                                              |
| `last_event_x`, `last_event_y` | Coordinates of the previous play.                                                                                       |
| `time_from_last_event`         | Time difference between the two plays. (SEC)                                                                            |
| `distance_from_last_event`     | Euclidean distance between this play and the previous play.                                                             |
| `rebound`                      | Boolean indicating whether this shot is a rebound.                                                                      |
| `change_in_shot_angle`         | Difference in shooting angle between consecutive shot events.                                                           |
| `speed`                        | Estimated puck speed between consecutive events. (RINK UNIT/SEC)                                                        |
| `friendly_skaters`,            | Number of skaters on the ice for the event-owner team.                                                                  |
| `opposing_skaters`             | Number of skaters on the ice for the opposing team.                                                                     |
| `pp_time_seconds`              | Number of seconds elapsed since the start of the current **power-play** A power-play is a numeric advantage on the ice. |

### Dataframe Artifact

The final filtered dataset for the **Winnipeg vs Washington** game `2017021065` was logged to the platform **Weights & Biases**.

🔗 [**Dataframe Artifact**](https://wandb.ai/alexandre-fournier-3-universite-de-montreal/ift6758-shot-prediction/artifacts/dataset/wpg_v_wsh_2017021065)

## Baseline Models
