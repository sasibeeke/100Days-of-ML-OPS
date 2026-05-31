The xFusionCorp Industries data science team compares multiple training runs with different hyperparameters using DVC experiments.<br>
Run three experiments that vary the n_estimators hyperparameter, identify the best-performing one, and promote it to the tracked workspace.<br>


1) A project exists at /root/code/fraud-detection/ with a parameterised DVC pipeline already in place.<br>
   params.yaml contains n_estimators: 100 and the baseline pipeline has been run once.<br>

2) Run three DVC experiments, each with a different value for n_estimators across a reasonable range (for example 50, 200, and 500).<br>
   Each experiment should produce a fresh metrics.json.<br>

3) Compare the experiments and choose the one whose f1_score is the highest.<br>

4) Apply the chosen experiment to the workspace so its n_estimators, metrics.json, and models/model.pkl become the tracked state.<br>

  The DVC extension's EXPERIMENTS section under the DVC view lists every experiment alongside its parameters and metrics, supports running fresh experiments <br>
  through the + action, and applies a selected experiment to the workspace from the right-click menu—every operation in this lab can be performed either through <br>
  the extension UI or with the equivalent dvc exp commands.<br>
  =>  I have apply values thorugh cmd . Run experiments with different n_estimators values.<br>
```bash
  # dvc exp run -S n_estimators=50
  # dvc exp run -S n_estimators=200
  # dvc exp run -S n_estimators=500
```
List the experiments with results<br>
```te
# dvc exp show
```
Identify the experiment with the highest f1_score in my case this is experiemnt exp-d4e5f6 & apply the best experiment.<br>
```re
# dvc exp apply exp-d4e5f6
```
This updates:<br>
params.yaml
metrics.json
models/model.pkl  to match the selected experiment.<br>

Verify workspace<br>
```yu
# cat params.yaml
# cat metrics.json
# dvc status
```
The workspace should now reflect the best experiment's results.
