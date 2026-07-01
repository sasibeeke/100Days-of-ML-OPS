The xFusionCorp Industries ML platform team is running a three-way bake-off between a RandomForest, a GradientBoosting, and a LogisticRegression<br>
candidate for fraud detection, with every candidate tracked as an MLflow run in the bakeoff experiment. Three correct trainer scripts are already<br>
in place, but the orchestrator at /root/code/fraud-detection/src/models/bakeoff.py picks the wrong winner and writes an incomplete report. <br>
Your task is to correct the orchestrator so the saved winner is the highest-F1 candidate and the report identifies which model family won.<br>


1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard <br>
loads with an empty bakeoff experiment.<br>
2) The project layout under /root/code/fraud-detection/:<br>
      * data/train.csv – The same 200-row synthetic binary-classification dataset Day 34 uses (imbalanced roughly 70 / 30).
      * src/models/train_rf.py, src/models/train_gb.py, src/models/train_lr.py – Three independent trainer scripts. Each one fits its named estimator
with 3-fold stratified CV and logs one MLflow run tagged candidate=<model family> with the mean f1_score metric and its hyperparameters. These three<br>
files are correct and need no edits.<br>
      * src/models/bakeoff.py – The orchestrator. It queries the bakeoff experiment with mlflow.search_runs(...) and writes<br>
/root/code/fraud-detection/reports/winner.json. Two specific corrections are required.<br>
4) Run each of the three trainer scripts once so every candidate is logged, open src/models/bakeoff.py in the VS Code editor, correct the two<br>
problems that keep the report from meeting the release checklist, save, and run the orchestrator.<br>
6) The end state must include:<br>
      * Three runs exist in the bakeoff MLflow experiment, one per candidate, each with tags.candidate, the candidate's hyperparameters, and<br>
        metrics.f1_score.<br>
      * A JSON file at /root/code/fraud-detection/reports/winner.json with exactly three keys: model_type (one of random_forest, gradient_boosting,<br>
        logistic_regression), run_id, and f1_score.<br>
      * The model_type, run_id, and f1_score stored in winner.json correspond to the candidate with the highest f1_score in the bakeoff experiment.<br>
The MLflow Compare view—select all three runs in the experiment's run list and click Compare—is the fastest way to eyeball which candidate won and<br>
spot-check the report.<br>

=> Go to the root directory of project <br>
```bash
   cd /root/code/fraud-detection
```
Then Run all the three files<br>
```test
  python src/models/train_rf.py
  python src/models/train_gb.py
  python src/models/train_lr.py
```
After this, the bakeoff experiment should contain three MLflow runs, one for each model:<br>
  * random_forest
  * gradient_boosting
  * logistic_regression
  
corrected bakeoff.py <br>
```nhy
"""Pick the winning candidate from the `bakeoff` MLflow experiment and
persist it at /root/code/fraud-detection/reports/winner.json.

Assumes train_rf.py, train_gb.py, and train_lr.py have each been run
at least once so the experiment contains three candidate runs.

The winner is the run with the highest metrics.f1_score. The saved
report must contain:

    {
      "model_type": "<candidate tag>",
      "run_id":     "<mlflow run id>",
      "f1_score":   <float>
    }
"""
import json
import os

import mlflow

TRACKING_URI = "http://localhost:5000"
EXPERIMENT = "bakeoff"
REPORTS_DIR = "/root/code/fraud-detection/reports"
WINNER_JSON = os.path.join(REPORTS_DIR, "winner.json")


def main():
    mlflow.set_tracking_uri(TRACKING_URI)
    client = mlflow.MlflowClient()
    exp = client.get_experiment_by_name(EXPERIMENT)
    if exp is None:
        raise SystemExit(
            f"Experiment {EXPERIMENT!r} not found. Run the three "
            "trainer scripts first."
        )

    runs = mlflow.search_runs(
        experiment_ids=[exp.experiment_id],
        order_by=["metrics.f1_score DESC"],
        max_results=10,
    )
    if runs.empty:
        raise SystemExit(
            f"No runs found in {EXPERIMENT!r}. Run the three trainer "
            "scripts first."
        )

    winner = runs.iloc[0]
    report = {
        "model_type": winner["tags.candidate"],
        "run_id": winner["run_id"],
        "f1_score": float(winner["metrics.f1_score"]),
    }

    os.makedirs(REPORTS_DIR, exist_ok=True)
    with open(WINNER_JSON, "w") as f:
        json.dump(report, f, indent=2)

    print(f"Winner written to {WINNER_JSON}: {report}")


if __name__ == "__main__":
    main()
```
Then Run the orchestrator & verify winner.json <br>
```test
python3 src/models/bakeoff.py
```
Gives which model achieved the highest F1 score.<br>
