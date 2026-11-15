# CIS5450_Final_Project
# Predicting Short Sleep Duration Among U.S. Adults

CIS 5450 final project. We use the CDC's 2022 Behavioral Risk Factor Surveillance System (BRFSS) to study how demographic, socioeconomic, and lifestyle characteristics relate to self-reported short sleep, defined as fewer than seven hours per 24-hour period.

## One branch per teammate

**Do all project work on your own branch. Merge into `main` through a pull request.** Each person starts their branch from the latest `main` and pushes their changes to their own branch.

| Team member | Working branch | Main responsibility |
| --- | --- | --- |
| Elaine Chen | `elaine/data-preparation` | Data cleaning and shared modeling inputs |
| Jerry Feng | `jerry/logistic-baseline` | L2 logistic regression baseline and interpretation |
| Jiayi Qi | `jiayi/eda-and-testing` | Exploratory analysis and permutation testing |
| Chang Gao | `chang/tree-models` | Tree models and model comparison |

After cloning the repository, open it in VS Code. In the terminal, run the following from the repo root. This example uses Jerry's branch. Substitute your branch name.

```bash
git switch main
git pull --ff-only origin main
git switch -c jerry/logistic-baseline
```

Once the branch exists locally, use `git switch jerry/logistic-baseline` to return to it. When your changes are ready, select the files to commit in VS Code's Source Control panel or stage them by path with `git add`. Then commit, push your branch, and open a pull request into `main` on GitHub.

```bash
git commit -m "Describe the change"
git push -u origin jerry/logistic-baseline
```

If another teammate is changing the same notebook, coordinate before editing it to avoid conflicting cell changes.

## Run the models in VS Code

1. Install the VS Code **Python** and **Jupyter** extensions. Install the dependencies in the Python environment you will use as the notebook kernel.

   ```bash
   python -m pip install ipykernel pandas numpy matplotlib scikit-learn
   ```

2. Once the cleaned CSV has been merged into `main`, clone the repository or update your local copy. The shared model input is `Data_Cleaning/brfss_2022_clean.csv`.
3. Open your modeling notebook, select your Python kernel, and read the CSV directly from your local copy of the repository.

## Modeling notes

- Use the same cleaned data and train, validation, and test assignments for all models. The preparation notebook assigns state-nested sampling units to partitions. The cleaned CSV does not currently contain those assignments, so agree on a shared export before comparing model scores.
- Set short sleep as the positive class with `1 - enough_sleep`. Use the ten designated input features. Keep sleep duration, outcome columns, and survey-design fields out of the feature matrix, and use survey weights separately where applicable.
- Compare L2 logistic regression, random forest, and XGBoost using the same validation metrics. Keep the final test set reserved until model settings are fixed.
- Keep explanations alongside key code and outputs in each notebook. We will combine the project workflow into one annotated notebook for the final submission.

