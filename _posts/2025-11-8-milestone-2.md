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


## Advanced Models

In this section, our goal was to expiriment with XGBoost based models to predict the probability that a shot results in a goal.

### XGBoost Baseline 

#### Training and Validation Split 

Used the Milestone 2 advanced_models setup: data from seasons 2016‑2019, booleans coerced to ints, and a 70/30 train/validation split stratified on goal. Training runs the preprocessing/XGB pipeline with grid search over tree depth, learning rate, etc., and validation metrics (ROC, goal-rate buckets, cumulative capture, calibration) are computed on the held-out 30%.

#### Logistic Regression baseline comparison

Compared with the simple Logistic Regression baselines (distance-only AUC 0.697, angle-only 0.589, distance+angle 0.703), the XGBoost baseline that ingests all engineered features reaches ROC‑AUC ≈ 0.715 and produces much steeper goal-rate curves in the top deciles (≈22% goals in the top 10% vs. ≈20% for the best LogReg). Calibration is also closer to the diagonal, especially in the mid-probability bins where XGB captures the bulk of goals while LogReg underestimates

[XGB Baseline](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/artifacts/model/question_2_all_features-model/v1)

### Hyperparameter Tunning

For the XGBoost baseline we wrap the model in the build_pipeline transformer (numeric passthrough + categorical One‑Hot) and run GridSearchCV with a 5‑fold StratifiedKFold (random_state=42) on n_estimators ∈ {100, 300}, max_depth ∈ {4, 6, 8}, learning_rate ∈ {0.05, 0.1}, subsample ∈ {0.8, 1.0}, and colsample_bytree ∈ {0.8, 1.0}. The grid is evaluated with ROC‑AUC so each candidate is scored on balanced folds before we keep the best estimator and evaluate it on the 30 % hold‑out set.

Because the assignment instructions were ambiguous about whether “Part 4 created features” should replace or complement the raw columns, we trained two pipelines: (a) all original features + the engineered ones from Part 4, and (b) only the Part 4 created features. Both used the same grid-search CV setup above. The full-feature pipeline consistently delivered higher ROC‑AUC and better lift curves, so we focused the downstream analysis on that configuration.

[XGB With Only Created Features](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/artifacts/model/question_2_created_features-model/v1)

[XGB With All Features](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/artifacts/model/question_2_all_features-model/v1)

### Feature Selection

We benchmarked four selection strategies (table below) within the identical preprocessing → XGBoost pipeline. The control row (`none`) simply feeds all 39 engineered features (see full list beneath the table) through GridSearchCV and delivers the best ROC‑AUC (0.7829 ± 0.0027). Because no column exhibits zero variance, `variance_threshold` produces an identical feature set and reproduces the same score, confirming that every signal contributes some variability worth retaining.

| strategy                   | mean_score | std_score | n_features | selected_features                                                       |
|----------------------------|------------|-----------|------------|-------------------------------------------------------------------------|
| none                       | 0.782909   | 0.002709  | 39         | [num__game_id, num__game_seconds, num__season, …]                       |
| variance_threshold         | 0.782909   | 0.002709  | 39         | [num__game_id, num__game_seconds, num__season, …]                       |
| select_from_model_logistic | 0.776633   | 0.003017  | 20         | [num__game_seconds, num__season, num__game_period, …]                   |
| select_kbest_f             | 0.766264   | 0.001894  | 20         | [num__game_seconds, num__game_period, num__shot_distance, …]            |

For the `select_from_model_logistic` row we use a logistic-regression surrogate to drop coefficients below the median, halving the space to 20 predictors centered on game state and shot context (`num__game_seconds`, `num__season`, `num__shot_distance`, etc.). That pruning costs roughly 0.006 ROC‑AUC, as seen in the table. The univariate `select_kbest_f` recipe keeps 20 features chosen via F-tests; it leans heavily on distance/angle and last-event categories and slips further to 0.766 ROC‑AUC, highlighting that ignoring multivariate interactions harms the boosted trees. Ultimately, the table shows that retaining the full engineered feature mix—either directly (`none`) or trivially via variance filtering—remains the optimal choice for capturing goal-probability nuances.

**Selected features for `none` / `variance_threshold`:**
```
['num__game_id', 'num__game_seconds', 'num__season', 'num__game_period', 'num__x', 'num__y', 'num__shot_distance', 'num__shot_angle', 'num__last_event_x', 'num__last_event_y', 'num__time_from_last_event', 'num__distance_from_last_event', 'num__rebound', 'num__change_in_shot_angle', 'num__speed', 'num__friendly_skaters', 'num__opposing_skaters', 'num__pp_time_seconds', 'cat__shot_type_backhand', 'cat__shot_type_deflected', 'cat__shot_type_slap', 'cat__shot_type_snap', 'cat__shot_type_tip-in', 'cat__shot_type_wrap-around', 'cat__shot_type_wrist', 'cat__last_event_type_blocked-shot', 'cat__last_event_type_delayed-penalty', 'cat__last_event_type_faceoff', 'cat__last_event_type_game-end', 'cat__last_event_type_giveaway', 'cat__last_event_type_goal', 'cat__last_event_type_hit', 'cat__last_event_type_missed-shot', 'cat__last_event_type_penalty', 'cat__last_event_type_period-end', 'cat__last_event_type_period-start', 'cat__last_event_type_shot-on-goal', 'cat__last_event_type_stoppage', 'cat__last_event_type_takeaway']
```

**Selected features for `select_from_model_logistic`:**
```
['num__game_seconds', 'num__season', 'num__game_period', 'num__y', 'num__shot_distance', 'num__shot_angle', 'num__last_event_x', 'num__last_event_y', 'num__time_from_last_event', 'num__distance_from_last_event', 'num__rebound', 'num__change_in_shot_angle', 'num__speed', 'num__friendly_skaters', 'num__opposing_skaters', 'num__pp_time_seconds', 'cat__shot_type_slap', 'cat__shot_type_tip-in', 'cat__last_event_type_faceoff', 'cat__last_event_type_shot-on-goal']
```

**Selected features for `select_kbest_f`:**
```
['num__game_seconds', 'num__game_period', 'num__shot_distance', 'num__shot_angle', 'num__rebound', 'num__change_in_shot_angle', 'num__speed', 'num__friendly_skaters', 'num__opposing_skaters', 'num__pp_time_seconds', 'cat__shot_type_backhand', 'cat__shot_type_deflected', 'cat__shot_type_slap', 'cat__shot_type_tip-in', 'cat__last_event_type_faceoff', 'cat__last_event_type_goal', 'cat__last_event_type_hit', 'cat__last_event_type_period-start', 'cat__last_event_type_shot-on-goal', 'cat__last_event_type_stoppage']
```
The SHAP waterfall for the top-decile shot puts concrete intuition behind the table’s numbers: `num__shot_distance` delivers the largest downward push on the log-odds (a long shot makes a goal unlikely), while `num__friendly_skaters`, `cat__shot_type_slap`, and `num__shot_angle` counterbalance with sizable positive contributions. Notice how every highlighted feature in the plot belongs to the 39-variable set retained by the `none`/`variance_threshold` rows: `cat__shot_type_wrist`, `cat__shot_type_backhand`, and `num__game_seconds` all appear in that list but many are removed once we restrict ourselves to 20 features. When `select_from_model_logistic` prunes the space to the features listed above, the SHAP figure hints at what is lost: secondary cues such as `cat__shot_type_wrist` or `num__y` still influence the boosted trees for specific plays, yet they fall below the logistic median-weight threshold and the AUC drops to 0.7766. The univariate `select_kbest_f` recipe trims even more aggressively, discarding contextual columns like `cat__shot_type_backhand` despite their localized importance in the SHAP plot, which explains the further decline to 0.7663 ROC-AUC. In short, the graph illustrates that even seemingly modest contributors from the full 39-feature roster participate in the additive explanation of real shots; stripping them out (rows 2–3 of the table) not only shrinks the feature list but also removes the very terms that help the XGBoost model capture nuanced goal probability patterns.

![Screenshot](milestone2_shap.png)


As we can see on the figures below higher-capacity XGBoost wins across all four validation views from the Milestone 2 70/30 split (2016–2019 shots, stratified on goal).

The ROC plot shows the all-feature XGBoost (orange) dominating the curve: its AUC climbs to 0.788, clearly above both the baseline (blue, 0.715) and the created-only run (green, 0.729), so it reaches higher true-positive rates for the same false-positive cost.

![Screenshot](milestone2_q5_1.png)

The goal-rate-by-percentile chart confirms that advantage in the ranking head: at the 90–100 % bucket the all-feature model converts ~33 % of shots into goals, beating the baseline ~22 % and the created-only model’s ~29 %, with the gap shrinking but persisting through the mid-deciles.

![Screenshot](milestone2_q5_2.png)

The cumulative capture curve illustrates lift across the sample: the dashed orange line amasses roughly 80 % of goals by the time it has scored the top 40 % of shots, while the blue line needs more than half the dataset to reach the same coverage, indicating stronger prioritization from the full feature set.

![Screenshot](milestone2_q5_3.png)

Finally, the reliability diagram shows calibration improvements: the orange markers sit closest to the diagonal between 0.2 and 0.6 predicted probability, meaning the all-feature model’s probabilities match observed goal frequencies better than the underconfident blue curve and the more erratic green created-only curve.

![Screenshot](milestone2_q5_4.png)

### Feature Engineering 

## Give it your best shot!

In this section, our goal was to design the best possible model to predict the probability that a shot results in a goal.

### Approaches Explored

To achieve this, we experimented with several model families and training strategies:

Random Forests: A robust ensemble model, combined with recursive feature elimination (RFECV) to reduce dimensionality and remove redundant variables.

Linear SVM with Calibration: A LinearSVC combined with a CalibratedClassifierCV to produce well-calibrated probabilities instead of raw decision margins.

SVM with PCA: Included a principal component analysis step to limit overfitting and test whether the principal components capture meaningful variance in shot features.

SVM without PCA: Served as a baseline to directly compare the effect of dimensionality reduction.

Validation Strategies:

- StratifiedKFold, to maintain class balance between goals and non-goals in each fold.

- TimeSeriesSplit, to simulate a more realistic temporal validation by preserving the chronological order of games.

Multilayer Perceptron : Tested several architectures with early stopping, regularization , and hyperparameter optimization using RandomizedSearchCV. These models proved particularly effective at capturing nonlinear interactions among shot features.

### Model Performance Summary

| Model                     | Validation Accuracy | AUC (Validation) | 
|----------------------------|--------------------:|-----------------:|
| **Random Forest**          | 0.9100             | 0.7311 | 
| **SVM (Linear, Calibrated)** | 0.9050             | 0.7224 |
| **SVM + PCA**              | 0.9050             | 0.6185 |
| **SVM (TimeSeriesSplit)**  | 0.9050             | 0.7224 |
| **MLP (Neural Network)**   | **0.9097**         | **0.7613** |


### Model Experiments

| Model                          | Wandb Run Link |
|--------------------------------|---------------|
| **Random Forest**              | [Link](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/sc0x247w?nw=nwuservissnu) |
| **SVM (Linear, Calibrated)**   | [Link](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/h1103sbc?nw=nwuservissnu) |
| **SVM + PCA**                  | [Link](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/64igwrhx?nw=nwuservissnu) |
| **SVM (TimeSeriesSplit)**      | [Link](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/zd23h07h?nw=nwuservissnu) |
| **MLP (Neural Network)**       | [Link](https://wandb.ai/IFT67582025-B2/ift6758-shot-prediction/runs/s7z3zhct?nw=nwuservissnu) |

#### Interpretation

The MLP model achieved the best overall performance, both in AUC (0.7613) and in accuracy (≈0.90), showing its ability to capture nonlinear patterns between shot features and goal probability.

The Random Forest performed strongly as well, slightly behind the MLP in AUC but still demonstrating good generalization.

The SVM models provided solid and consistent performance, though the variant with PCA underperformed , suggesting that dimensionality reduction may have removed some important discriminative information.

The TimeSeriesSplit version of SVM confirmed that temporal validation did not significantly degrade performance, indicating stability over time.

Note: While accuracy values are relatively close across models, the AUC score is the most important metric here , it measures how well each model distinguishes between goals and non-goals, regardless of the decision threshold.

### First Approach : **Random Forest**
For the first experiment, we implemented a Random Forest classifier to establish a strong nonlinear baseline for predicting the probability that a shot results in a goal. 

We integrated the classifier into a scikit-learn pipeline that included imputation of missing values, feature scaling, and categorical encoding using an OrdinalEncoder. To improve generalization and reduce dimensionality, we added a recursive feature elimination with cross-validation (RFECV) step, which automatically selected the most informative features before training.

A RandomizedSearchCV was applied over key hyperparameters,  such as the number of estimators, maximum tree depth, and minimum samples per split and leaf , using a K-Fold cross-validation strategy. 

### Second Approach: **SVM (Linear, Calibrated)**

For the second experiment, we trained a Linear SVM classifier.

This model was integrated into the same pipeline , ensuring consistent preprocessing steps . We performed a RandomizedSearchCV over the regularization parameter C using a stratifiedkfold cross-validation to maintain class balance between goals and non-goals in each fold.


### Third Approach: SVM with PCA
In this third experiment, we extended the linear SVM approach by incorporating Principal Component Analysis into the training pipeline. The objective was to evaluate whether dimensionality reduction could improve model generalization by removing redundant or noisy features.

The pipeline included the same preprocessing steps , followed by a PCA transformation. A CalibratedClassifierCV was again used to provide probabilistic outputs. We tuned both the regularization parameter (C) of the SVM and the number of PCA components using RandomizedSearchCV with a StratifiedKFold cross-validation strategy.

Although this version slightly reduced overfitting, the overall performance did not improve compared to the standard SVM. The AUC dropped marginally, suggesting that important information might have been lost during dimensionality reduction.

### Fourth Approach: SVM (TimeSeriesSplit)
For the fourth experiment, we kept the linear calibrated SVM architecture but changed the cross-validation strategy to a TimeSeriesSplit approach.
The goal was to make the validation process more realistic from a temporal perspective, ensuring that the model was always trained on earlier games and validated on later ones .

This setup is particularly relevant in sports analytics, where player performance and team dynamics evolve over time.
The preprocessing pipeline remained consistent with previous models, and the SVM regularization parameter (C) was optimized using RandomizedSearchCV with five temporal splits.

The results showed that the model maintained a similar level of accuracy and AUC as the standard cross-validation approach, indicating strong temporal stability.


### Fifth Approach : MLP (Neural Network)
For the final experiment, we implemented a Multilayer Perceptron , a type of feed-forward neural network  to explore whether a nonlinear, high-capacity model could capture more complex relationships between shot features and goal probability.

To improve generalization and avoid overfitting, we introduced several techniques:

- Regularization via the alpha parameter,

- Early stopping based on validation loss,

- Batch training with different batch sizes ,

- Hyperparameter tuning through RandomizedSearchCV across architecture size, learning rate, and regularization strength.

This model was trained within the same pipeline structure and evaluated using StratifiedKFold cross-validation to maintain class balance. The MLP achieved the highest AUC and accuracy among all tested models, indicating that its nonlinear transformations were able to capture deeper interactions between spatial and contextual shot features.

### Best Model: MLP (Neural Network)

Among all the models tested, the Multilayer Perceptron  emerged as the best-performing model in terms of both AUC and overall calibration.
Unlike the simpler linear models, the MLP was able to capture nonlinear interactions between features such as shot angle, distance, speed, and rebound , variables that interact in complex ways when determining the probability of a goal.

Thanks to regularization , early stopping, and hyperparameter tuning through RandomizedSearchCV, the model achieved strong generalization on the validation set . Its reliability curve and goal-rate analysis confirmed that it produced well-calibrated probabilities across the entire range of predicted values.

Therefore, we selected the MLP as our final model to evaluate on the holdout test set.

### ROC Curve Comparison - All Models

![ROC Curve Comparison – All Models](milestone2_roc_all_models.png)

This figure compares the ROC curves of all five models tested.
The MLP (purple curve) achieved the highest AUC (0.7613), showing the best ability to discriminate between shots that resulted in goals and those that did not.
The Random Forest followed closely (AUC = 0.7535), while the SVM models performed slightly lower, particularly the SVM + PCA, which lost some discriminative power due to dimensionality reduction.

The ROC analysis confirms that the MLP generalizes best across probability thresholds, balancing sensitivity and specificity more effectively than other models.

### Goal Rate by Predicted Probability Percentile - Comparison
![ROC Curve Comparison – All Models](milestone2_goal_rate_comparison.png)

This figure shows the goal rate (Goals / Shots) across deciles of predicted shot probability for each model.
Ideally, models should display a monotonic increase , meaning that higher predicted probabilities correspond to a higher proportion of goals.

The MLP (purple curve) and Random Forest (blue curve) exhibit the clearest and most consistent upward trends.
The SVM + PCA curve, however, appears less stable, suggesting that dimensionality reduction may have lost some discriminative information.

Overall, this analysis reinforces the conclusion that the MLP produces the most well-calibrated and discriminative predictions.

### Cumulative % of Goals - Comparison

![Cumulative % of Goals – Comparison](milestone2_cumulative_goals_comparison.png)

This cumulative goal curve illustrates how effectively each model concentrates the true goals in the top predicted probability bins.  
A more curved and steep profile indicates that the model successfully assigns higher probabilities to the shots that are actually goals.  

As shown, the MLP (purple) and Random Forest (blue) capture a higher share of goals within the top percentiles.  
The SVM + PCA remains the weakest performer, suggesting information loss during dimensionality reduction.  

### Calibration Curve (Reliability Diagram) - Comparison
![Calibration Curve (Reliability Diagram) – Comparison](milestone2_reliability_curve_comparison.png)

This reliability diagram illustrates how well each model’s predicted probabilities align with the true likelihood of scoring a goal.  
The diagonal gray line represents perfect calibration — where predicted and actual probabilities match exactly.

The Random Forest and MLP models show reasonably good calibration in the lower to mid probability range but slightly overestimate at higher probabilities.  
The SVM-based models, particularly the version with PCA, display weaker calibration, often underestimating the true goal frequency in mid to high probability regions.  




## Evaluate on test set
