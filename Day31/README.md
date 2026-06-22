The xFusionCorp Industries ML platform team maintains a config-driven training pipeline so hyperparameters can be swapped without editing Python<br>
code. A training scaffold exists at /root/code/fraud-detection/ with the trainer already in place, but the YAML config has been left in a broken<br>
state and the pipeline will not run cleanly. Your task is to correct the config so one successful training run lands on the MLflow tracking<br> 
server and the trained model ends up inside the project tree.<br>

1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the<br> 
dashboard loads with an empty fraud-detection experiment already in place.<br>

2) The project layout under /root/code/fraud-detection/:<br>
      * data/train.csv – A pre-generated 200-row synthetic binary classification dataset (columns: amount, hour, num_tx_past_day, is_fraud).
      * src/models/train.py – The config-driven trainer. This file is correct and must not be modified.
      * configs/train_config.yaml – The project's training configuration. This file has bugs.
      * models/ – Where the serialised model must land.
3) Running python3 /root/code/fraud-detection/src/models/train.py currently either fails with an estimator or column error, or succeeds while<br>
dropping the model file outside the project tree. Open configs/train_config.yaml in the VS Code editor, identify every setting that prevents<br>
a clean run, and correct it.<br>

4) The end state must include:<br>
      * A successful training run printed to stdout.
      * Exactly one new MLflow run in the fraud-detection experiment, with the estimator's hyperparameters logged as run parameters.
      * A serialised model at /root/code/fraud-detection/models/model.pkl (absolute path, inside the project tree).
      
The trainer uses a small registry of supported estimators—RandomForestClassifier, GradientBoostingClassifier, LogisticRegression. Only these<br>
exact class names resolve.<br>

=> fisrtly i run model it gives issue of unknown estimator type<br>
```test
python3 /root/code/fraud-detection/src/models/train.py 
```
ERROR: unknown estimator type 'RandomForest'. Supported: ['GradientBoostingClassifier', 'LogisticRegression', 'RandomForestClassifier']

Then i run model again after correction it gives issue of column.<br>
```ty
python3 /root/code/fraud-detection/src/models/train.py 
```
ERROR: target column 'target' not found in /root/code/fraud-detection/data/train.csv. Available columns:
['amount', 'hour', 'num_tx_past_day', 'is_fraud']<br>

Then i run again this time it runs but model is not generated at specified path so added correct model path.<br>

corrected train_config.yaml<br>
```bash
model:
  type: RandomForestClassifier
  n_estimators: 100
  max_depth: 5
  random_state: 42
data:
  train_path: /root/code/fraud-detection/data/train.csv
  target_column: is_fraud
output:
  model_path: /root/code/fraud-detection/models/model.pkl
mlflow:
  tracking_uri: http://localhost:5000
  experiment_name: fraud-detection
```
finally it runs without any error and model is generated at specified location.<br>
