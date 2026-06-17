The xFusionCorp Industries ML platform team packages their training runs as MLflow Projects so any engineer can reproduce them with a single mlflow<br>
run invocation. A project has been pre-staged at /root/code/trainer/, but the first run from the lab startup already failed—the MLproject file <br>
carries a subtle command-line bug. Your task is to fix the bug and then run the project end to end twice so successful runs are recorded.<br>


1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to view <br> the dashboard; 
the trainer experiment already contains a FAILED run from the automated run that the lab startup triggered against the broken project.<br>

2) /root/code/trainer/ contains:<br>
    * MLproject – The project descriptor (this file has the bug).
    * train.py – The trainer entry point. This file is correct and must not be modified.
   
3) The fix is confined to MLproject. Once repaired, two successful mlflow run invocations must be recorded in the trainer experiment:<br>
    * One explicit call: mlflow run . -e train -P n_estimators=200 -P max_depth=10 --env-manager=local (run from /root/code/trainer).
    * One default call: mlflow run . -e train --env-manager=local.

4) The end state must include:<br>
    * The MLproject command line invokes train.py with flag names that match train.py's argparse declarations.
    * The trainer experiment contains at least two FINISHED runs whose params.n_estimators values differ (one at 200, one at the default 100).
    * The original FAILED run from the startup is still present – It must not be deleted, because the lab's diagnosis trail depends on it.
The failed run's page in the MLflow UI and /tmp/mlflow-run-initial.log both carry the underlying error. Running mlflow run . <br>
manually from /root/code/trainer reproduces the same error directly in the terminal.<br>
=>
corrected MLproject<br>
```test
name: trainer

entry_points:
  train:
    parameters:
      n_estimators:
        type: int
        default: 100
      max_depth:
        type: int
        default: 5
      test_size:
        type: float
        default: 0.2
      random_seed:
        type: int
        default: 42
    command: >
      python train.py
      --n_estimators {n_estimators}
      --max_depth {max_depth}
      --test_size {test_size}
      --random_seed {random_seed}
```
I did change here previously --n_est {n_estimators} instead i replace with --n_estimators {n_estimators}<br>
```bash
command: >
      python train.py
      --n_estimators {n_estimators}
      --max_depth {max_depth}
      --test_size {test_size}
      --random_seed {random_seed}
```

Once repaired, two successful mlflow run invocations must be recorded in the trainer experiment:<br>
```te
cd /root/code/trainer

mlflow run . -e train -P n_estimators=200 -P max_depth=10 --env-manager=local

mlflow run . -e train --env-manager=local
```

Then check the experiment in the MLflow UI and ensure:<br>
  * Original FAILED run still exists.
  * Two new FINISHED runs exist.
  * One has n_estimators=200.
  * One has n_estimators=100 (default).
      
