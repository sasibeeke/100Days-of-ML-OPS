The xFusionCorp Industries MLOps team needs the fraud-detection-v2 candidate promoted all the way from the tracking server to live inference and <br>
monitored end to end. The backing stack (PostgreSQL, SeaweedFS, MLflow tracking server, three candidate runs) is already in place. <br>
Your task is to cover the remaining lifecycle work: model promotion, serving, and health monitoring.<br>

1) The infrastructure is fully up: PostgreSQL container mlflow-db on port 5432, SeaweedFS on ports 8333 (S3 API) and 8888 (Filer UI), MLflow<br>
tracking server on port 5000 with the PostgreSQL backend and the mlflow-artifacts S3 bucket. The fraud-detection-v2 experiment contains three
candidate runs (baseline, improved, regression) with logged f1_score metrics.<br>The MLflow UI and SeaweedFS Filer buttons at the top of the lab can be
opened to view each web UI.<br>

2) The complete end state requires the following.<br>
    * A registered model named fraud-detector-v2 exists in the MLflow Model Registry.
    * A champion alias on that model points at the version sourced from the fraud-detection-v2 run with the highest f1_score.
    * An mlflow models serve process listens on port 5001, serving the champion version (--env-manager=local is the supported choice for the lab).Export<br>
    MLFLOW_TRACKING_URI=http://localhost:5000 in the serving shell so the models:/ URI can be resolved against the tracking server. The tracking
    server proxies the model download from SeaweedFS itself, so no S3 credentials are needed in the serving shell.
    * The served endpoint returns 200 on GET /health.
    * A shell script at /root/code/monitor.sh exists, is executable, probes the served model's /health endpoint once, and exits with status 0 when the
    endpoint is healthy.
3) The top run can be identified either through the MLflow UI Compare view or with a one-off MlflowClient.search_runs() call—whichever is preferable.<br>
   The registration and alias assignment are likewise available from the UI (Models tab) or the SDK.<br>

mlflow models serve is long-running; start it in the background, and ensure that the new process is listening on port 5001 before writing the <br>
monitoring script.<br>
=> 
Step 1: Open the MLflow UI<br>
Open: http://localhost:5000

Step 2: Find the best run<br>
  1) Click Experiments.
  2) Open fraud-detection-v2.
  3) You should see three runs:
     * baseline
     * improved
     * regression
  4) Compare the f1_score column.
  5) Note the run having the highest f1_score.
  
Step 3: Register the model<br>
  1) Click the best run.
  2) Scroll to the Artifacts section.
  3) Open the model artifact.
  4) Click Register Model.

If asked:<br>
```hn
  Model Name:
    fraud-detector-v2
```
Click<br>
```pl  
  Register
```
Now the model is added to the Model Registry.<br>

Step 4: Open the Model Registry<br>

Click<br>
```is
  Models
```
from the top menu.<br>
You should now see<br>
```sa
  fraud-detector-v2
```
Click it.<br>

Step 5: Set the Champion alias<br>
Inside the registered model:<br>
You'll see something similar to
```ee
  Version 1
```
depending on previous registrations.<br>
Click the version that came from the highest f1_score run.<br>

Look for<br>
```io
  Aliases

or

  Add Alias
```
Enter<br>
```yy
  champion
```
Save.<br>

Step 7: Start model serving (Terminal)<br>
```bash
export MLFLOW_TRACKING_URI=http://localhost:5000

mlflow models serve \
-m models:/fraud-detector-v2@champion \
-p 5001 \
--env-manager=local
```
Since this command keeps running, start it in another terminal or run it in the background:<br>

Step 8: Check the health endpoint<br>
```yu
  curl http://localhost:5001/health
```
Step 9:  monitoring script already present their but extension of script is sh.template so rename it to .sh & make it executable.<br>
```ge
  chmod +x /root/code/monitor.sh
``` 
Verify it<br>
```test 
/root/code/monitor.sh
echo $?
```
output is <br>
  healthy 
  0.

  
  
