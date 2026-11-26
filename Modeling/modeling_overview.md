# Modeling Overview

## 1. Setup
- **Data:** CDC BRFSS 2022, 427,573 adults after cleaning.
- **Target:** 1 if the person reports fewer than 7 hours of sleep. This follows the AASM recommendation of at least 7 hours for adults. There's no upper limit, because the evidence on long sleep is weaker.
- **Predictors:** 10 categorical variables. Most of them keep an explicit `Unknown` category. We dropped 3,005 rows (under 1%) with unknown education or physical activity.
- **Split:** 60/20/20 train/validation/test. We assign whole state × PSU clusters, so no cluster appears in more than one set. Class balance is similar across the three sets.
- **Models:** a constant baseline, L2 logistic regression, random forest and XGBoost. All four use the same inputs and the same split.

## 2. Why Tree Models
Logistic regression adds up each factor separately. Sleep may depend on combinations of factors, such as being young, employed and having children. Trees capture these interactions automatically.

- **Random forest:** 300 trees. Each tree is trained on a random half of the rows and considers a random subset of features at each split. The prediction is the average of the trees' probabilities.
- **XGBoost:** small trees built one after another. Each new tree corrects the current model's errors and is added with a small learning rate.

The two methods differ: random forest averages independent trees, while XGBoost stacks trees that correct each other. If neither clearly beats logistic regression, that is also a finding: the simple model is enough.

## 3. Tuning: Randomized Search
We sample 20 hyperparameter configurations per model. Usually only a few hyperparameters matter much, and random sampling tries more distinct values of those than a grid of the same cost. It also lets us draw continuous values, like the learning rate, on a log scale.

Each configuration is trained on the training set and scored on the validation set. We keep the best configuration for each model. The test set is used once, after all tuning is done.

| Model | Hyperparameter | Role |
|---|---|---|
| RF | `max_depth` | Limits tree depth |
| RF | `min_samples_leaf` | Minimum people per leaf; larger values give smoother predictions |
| RF | `max_features` | Features considered per split; keeps the trees different from each other |
| XGB | `max_depth` | Kept small, since each tree only fixes part of the error |
| XGB | `learning_rate`, `n_estimators` | Trade off against each other: a smaller step needs more trees |
| XGB | `subsample`, `colsample_bytree` | Add randomness |
| XGB | `reg_lambda` | Penalizes large leaf values |

We fix RF at 300 trees. Adding more trees only makes it more stable; it doesn't cause overfitting.

## 4. Metric: Weighted Average Precision (AP)
To compute AP, we rank people by predicted probability. Each time the ranking reaches a true short sleeper, we take the precision at that point, then average those values. A perfect ranking scores 1.

- **Why not accuracy:** About a third of people are short sleepers. Predicting "no" for everyone already gives about 67% accuracy, which means nothing. AP focuses on the positive class and needs no cutoff.
- **Why weighted:** Survey weights make each respondent count in proportion to the number of U.S. adults they represent.
- **Reference points (validation):** A constant score gets AP = 0.363, the share of short sleepers. Logistic regression gets 0.454. There is real signal, but it is limited.

## 5. Interpretation: Grouped Permutation Importance
On the validation set, we shuffle one predictor across rows and measure the drop in weighted AP. We repeat this 10 times and average.

- **Grouped:** We shuffle the original column, not its one-hot columns. This keeps every row valid and gives one number per predictor.
- **Compared with logistic coefficients:** A coefficient gives the direction of an association, and a predictor like income has several of them. Permutation importance gives one number per predictor, measures the effect on prediction quality, and works for any model.
- **Limitation:** Income, education and employment overlap. When one is shuffled, the model can partly recover it from the others, so its importance is underestimated. The results show what the model relies on, not what causes short sleep.

## 6. Significance: Cluster Bootstrap
The question is whether the best tree model's AP gain over logistic regression is real or just noise from which people landed in the test set.

We resample the test set 1,000 times and compute the AP difference each time. If the middle 95% of these differences excludes zero, the gain is unlikely to be noise. We resample whole PSU clusters rather than individual rows. People in the same cluster tend to be similar, so resampling rows would make the interval too narrow.
