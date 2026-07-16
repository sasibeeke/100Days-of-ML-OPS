The xFusionCorp Industries ML platform team has developed a comprehensive fraud-detection training pipeline that includes data validation, Optuna tuning across two model families, model selection against a release<br>
threshold, Model Registry registration with a release-lane alias, and a consolidated training report. All of these components are integrated behind a single make train-pipeline command. Currently, the pre-staged <br>
system does not function end-to-end, as each invocation of make train-pipeline reveals a wiring issue, and two stages contain unfinished TODO items. To prepare for the release checklist, you must address the <br>
necessary updates in the Makefile, src/select_model.py, src/register.py, and src/report.py. Your objective is to resolve the wiring issues and complete the two TODO blocks, ensuring that make train-pipeline <br>
executes successfully from start to finish, the MLflow Model Registry contains a fraud-detector version under the staging alias, and reports/training_report.json compiles all upstream artifacts.<br>

1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty fraud-detection-tuning experiment.<br>
2) The project layout under /root/code/fraud-detection/:<br>
    * data/train.csv – The 200-row synthetic binary-classification dataset the rest of the Training section uses.
    * src/validate_data.py – Schema + null-check gate. Writes reports/validation_status.json. Correct.
    * src/tune.py – Runs 10 Optuna trials across RandomForest and GradientBoosting, each logged as an MLflow run tagged with model_type + params.{n_estimators,max_depth} + metrics.f1_score + the fitted model artefact.
Correct.
    * src/select_model.py – Picks the winning run by the training metric and writes reports/selection.json. Has a wiring bug.
    * src/register.py – Registers the selected run's model as fraud-detector; the release-lane alias assignment is left as a # TODO.
    * src/report.py – Aggregates every upstream artefact into reports/training_report.json; the report assembly is left as a # TODO.
    * Makefile – train-pipeline target runs the five stages in order. Has a wiring bug.
3) The end state must include:<br>
    * make train-pipeline completes without non-zero exit.
    * The fraud-detection-tuning MLflow experiment carries at least five trial runs, each with metrics.f1_score.
    * reports/selection.json, reports/validation_status.json, and reports/training_report.json are all present. The training report carries best_model, best_params, metrics, total_trials, and validation_status keys; 
validation_status is "ok" and total_trials is an integer ≥ 5.
    * The MLflow Model Registry (MLflow UI → Models) shows a fraud-detector registered model with at least one version. That version carries the staging alias and no production alias.
Run make train-pipeline once against the scaffold as-is — the first wiring bug surfaces immediately, and each re-run reveals the next. The two # TODO blocks (the registry alias and the report assembly) do not 
crash the pipeline; they are caught by the release checklist, so complete them before expecting a clean pass.
=>  first run  it gives error <br>
```test
make train-pipeline
```
python3 src/validate_data.py
[validate] {'status': 'ok', 'rows': 200, 'columns': ['amount', 'hour', 'num_tx_past_day', 'is_fraud']}
python3 src/select_model.py
[select] no runs in experiment 'fraud-detection-tuning' — the tune stage has not produced any candidates yet.
make: *** [Makefile:8: train-pipeline] Error 1

Correct Makefile <br>
```test
.PHONY: train-pipeline clean

# xFusionCorp Industries — Fraud Detection Training Pipeline.
# Usage: make train-pipeline

train-pipeline:
	python3 src/validate_data.py
	python3 src/tune.py
	python3 src/select_model.py
	python3 src/register.py
	python3 src/report.py

clean:
	rm -rf models/ reports/
```
corrected src/select_model.py<br>
```bg
"""Stage 3 — Model selection.

Reads every run in the `fraud-detection-tuning` experiment, picks
the best candidate by the training metric, validates it against the
release threshold, and persists the selection to
`reports/selection.json` for the register stage.
"""
import json
import os
import sys

import mlflow

TRACKING_URI = "http://localhost:5000"
EXPERIMENT = "fraud-detection-tuning"
REPORTS_DIR = "/root/code/fraud-detection/reports"
SELECTION_JSON = os.path.join(REPORTS_DIR, "selection.json")

RELEASE_THRESHOLD = 0.4


def main():
    mlflow.set_tracking_uri(TRACKING_URI)
    client = mlflow.MlflowClient()
    exp = client.get_experiment_by_name(EXPERIMENT)
    if exp is None:
        sys.exit(f"[select] experiment {EXPERIMENT!r} not found.")

    runs = mlflow.search_runs(
        experiment_ids=[exp.experiment_id],
        order_by=["metrics.f1_score DESC"],
        max_results=200,
    )
    if runs.empty:
        sys.exit(
            f"[select] no runs in experiment {EXPERIMENT!r} — the tune "
            "stage has not produced any candidates yet."
        )

    best = runs.iloc[0]
    score = float(best["metrics.f1_score"])
    if score < RELEASE_THRESHOLD:
        sys.exit(
            f"[select] best candidate ({score:.4f}) is below the "
            f"release threshold ({RELEASE_THRESHOLD})."
        )

    selection = {
        "run_id": best["run_id"],
        "model_type": best.get("tags.model_type", ""),
        "f1_score": score,
    }
    os.makedirs(REPORTS_DIR, exist_ok=True)
    with open(SELECTION_JSON, "w") as f:
        json.dump(selection, f, indent=2)
    print(f"[select] {selection}")


if __name__ == "__main__":
    main()
```
corrected src/register.py<br>
```ty
"""Stage 4 — Register the selected model.

Reads the selection written by the previous stage, registers the
selected run's model as `fraud-detector` in the MLflow Model
Registry, and assigns the release-lane alias so the serving layer
can fetch the right version by name.
"""
import json
import os
import sys

import mlflow
from mlflow.tracking import MlflowClient

TRACKING_URI = "http://localhost:5000"
REPORTS_DIR = "/root/code/fraud-detection/reports"
SELECTION_JSON = os.path.join(REPORTS_DIR, "selection.json")

REGISTERED_MODEL_NAME = "fraud-detector"
# The release-lane alias the serving layer resolves by name. Per the
# release checklist, models promoted by this pipeline go to "staging".
RELEASE_ALIAS = "staging"


def main():
    if not os.path.exists(SELECTION_JSON):
        sys.exit(
            f"[register] {SELECTION_JSON} missing — the select stage "
            "has not produced a selection yet."
        )
    with open(SELECTION_JSON) as f:
        selection = json.load(f)

    mlflow.set_tracking_uri(TRACKING_URI)
    client = MlflowClient()

    model_uri = f"runs:/{selection['run_id']}/model"
    version = mlflow.register_model(model_uri, REGISTERED_MODEL_NAME)

    # TODO: assign the release-lane alias so the serving layer can fetch
    # this version by name. Point RELEASE_ALIAS at the just-registered
    # version using client.set_registered_model_alias(name, alias,
    # version) — pass REGISTERED_MODEL_NAME, RELEASE_ALIAS, and
    # version.version.
    client.set_registered_model_alias(
    REGISTERED_MODEL_NAME,
    RELEASE_ALIAS,
    version.version,
    )
    print(
        f"[register] {REGISTERED_MODEL_NAME} v{version.version} "
        f"registered (assign the {RELEASE_ALIAS!r} alias in the TODO)"
    )


if __name__ == "__main__":
    main()
```
corrected src/report.py<br>
```ty
"""Stage 5 — Training report.

Aggregates every upstream stage's output into a single JSON report
at `reports/training_report.json`. Reads:
  - `reports/validation_status.json` produced by the validate stage.
  - `reports/selection.json` produced by the select stage.
  - the MLflow experiment's run count for the total trials figure.
"""
import json
import os

import mlflow

TRACKING_URI = "http://localhost:5000"
EXPERIMENT = "fraud-detection-tuning"
REPORTS_DIR = "/root/code/fraud-detection/reports"

VALIDATION_JSON = os.path.join(REPORTS_DIR, "validation_status.json")
SELECTION_JSON = os.path.join(REPORTS_DIR, "selection.json")
TRAINING_REPORT_JSON = os.path.join(REPORTS_DIR, "training_report.json")


def main():
    with open(VALIDATION_JSON) as f:
        validation = json.load(f)
    with open(SELECTION_JSON) as f:
        selection = json.load(f)

    mlflow.set_tracking_uri(TRACKING_URI)
    client = mlflow.MlflowClient()
    exp = client.get_experiment_by_name(EXPERIMENT)
    runs = mlflow.search_runs([exp.experiment_id], max_results=500) if exp else []
    total_trials = int(len(runs)) if hasattr(runs, "__len__") else 0

    run_id = selection["run_id"]
    client = mlflow.MlflowClient()
    run = client.get_run(run_id)
    best_params = {k: v for k, v in run.data.params.items()}
    best_metrics = {k: float(v) for k, v in run.data.metrics.items()}

    # TODO: assemble the consolidated training report from the upstream
    # artefacts gathered above. Build a dict with exactly these five keys
    # and bind it to `report`:
    #   "best_model"        -> selection's model_type (selection["model_type"])
    #   "best_params"       -> best_params
    #   "metrics"           -> best_metrics
    #   "total_trials"      -> total_trials
    #   "validation_status" -> validation's status (validation["status"])
    report = {
    "best_model": selection["model_type"],
    "best_params": best_params,
    "metrics": best_metrics,
    "total_trials": total_trials,
    "validation_status": validation["status"],
}

    os.makedirs(REPORTS_DIR, exist_ok=True)
    with open(TRAINING_REPORT_JSON, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[report] {TRAINING_REPORT_JSON}")


if __name__ == "__main__":
    main()
```

After this again run <br>
```sh
make train-pipeline
```
The final successful run should produce:
✅ make train-pipeline exits successfully.
✅ reports/validation_status.json
✅ reports/selection.json
✅ reports/training_report.json
✅ MLflow experiment with at least 5 trial runs and metrics.f1_score.
✅ Registered model fraud-detector with a version under the staging alias only.
