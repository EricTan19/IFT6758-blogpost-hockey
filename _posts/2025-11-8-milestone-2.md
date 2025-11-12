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
The baseline logistic regression model using distance achieves a high accuracy of 91%. However, this result is misleading, as the model predicts only "no-goal" outcomes across the entire test set. From the confusion matrix below, we can see that the model only predicted samples to be "no-goals". This high-accuracy is caused by an imbalanced training set, where the number of non-goal samples is significantly more present than goals. This is an indication to use other metrics, such as F1-score, precision, and recall to further test the generalization capabilities of our models.

![Screenshot](milestone2_log_reg_distance_results.png)


The graph below shows four different ROC curve (Receiver Operating Characteristic) comparing how well different logistic regression models predict a binary outcome, either goal or non-goal, based on different input feature: distace, angle, and their combination. Each ROC curves plots the false positive rate on the x-axis and the true positive rate on the y-axis. The AUC (Area Under the Curve) summarizes the model’s overall performance: the higher the AUC, the better the model’s ability to discriminate between the two classes. The LogReg - Distance has an AUC of 0.697. The LogReg - Angle has an AUC of 0.589. The LogReg - Distance/Angle has an AUC of 0.703, and the Random baseline, which follows a uniform distribution has an AUC of 0.499. These results show that all models perform better than the random baseline, meaning that they capture some relationship between features and the target. The LogReg - Distance/Angle model performs the best as combining both features captures more information about the event of the shot.

![Screenshot](milestone2_baseline_models_roc_curves.png)

The graph below shows how well the predicted probabilities from different logistic regression models align with the actual observed goal rates. The x-axis shows shot probability percentiles and the y-axis shows the actual goal rate within each percentile. This means that shots with higher predicted probabilities correspond to higher actual goal rates. Distance from the net is the strongest predictor of goal, while the combination of distance and angle results in the best model overall. Whereas the angle feature provides limited predictive value.

![Screenshot](milestone2_baseline_models_goal_rate_predicted_probability.png)

The graph below shows how well different models capture actual goals as you move through percentiles of predicted shot probability. The curves measure how efficiently each model ranks shots from most to least dangerous. The shots are ranked by their predicted probability of being a goal on the x-axis, where 100 is the lowest probability and 0 is the highest probability (percentile). The y-axis is the fraction of all true goals capture up to that percentile. The distance and distance/angle logistic regression models show the most promising results by effectively ranking the shots by true scoring likelihood. The angle-only model performs worse, indicating that distance from the net is a more dominant feature in scoring likelihood.

![Screenshot](milestone2_baseline_models_cumulative.png)

The graph below shows how well each model's predicted probabilities match the true observed frequencies of goals. It measures how trustworthy the model's probability outputs are. The x-axis represents the mean predicted probability and the y-axis shows the actual fraction of shots that turned into goals. A perfectly calibated model would lie exactly on the diagonal line. Distance-based models are. well-calibrated, while the angle logistic regression model performs slightly worse. All logistic regression curves stay near the bottom left region of the graph, which suggests that most shots have a low change of scoring, which matches the hockey dataset.

![Screenshot](milestone2_baseline_models_calibration.png)

[Logistic regression with distance from net](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/r9a8sje4?nw=nwusererictan)

[Logistic regression with angle from net](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/jz8smqnz?nw=nwusererictan)

[Logistic regression with distance and angle from net](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/syty1dfp?nw=nwusererictan)

## Evaluate on test set
