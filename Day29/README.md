The xFusionCorp Industries ML platform team rolled out a shared MLflow deployment backed by a production-style tracking store (PostgreSQL) 
and a production-style artefact store (SeaweedFS, S3-compatible). The MLflow server is running and metadata writes to PostgreSQL succeed, 
but a teammate noticed that no artefact ever lands in the mlflow-artifacts SeaweedFS bucket — the connection from MLflow to SeaweedFS is 
not wired correctly. Diagnose what is broken in the MLflow server's startup configuration, fix it, and re-run the pre-staged smoke-test so 
one round trip lands metadata in PostgreSQL and the model artefact in SeaweedFS.<br>

1) The pre-staged state:<br>

   * PostgreSQL container mlflow-db is running on port 5432 (database mlflow, credentials mlflow / mlflow123).
   * SeaweedFS is running on port 8333 (S3 API) / 8888 (Filer UI), credentials weedadmin / weedadmin123, with a pre-created bucket mlflow-artifacts.
   * MLflow tracking server is running on port 5000 and was launched by /root/code/start-mlflow.sh. Its log is at /tmp/mlflow.log.
   * Reference scripts: /root/code/start-mlflow.sh (the MLflow startup command), /root/code/restart-mlflow.sh (kills the running server and
     re-launches via start-mlflow.sh), and /root/code/log_test_run.py (the smoke-test that exercises one full round trip).

2) Run the smoke-test once to observe the failure:<br>
   ```bash
   python3 /root/code/log_test_run.py
   ```
   
The MLflow run appears in the MLflow UI (the metadata write to PostgreSQL succeeds), but the model artefact upload step raises an error 
because the MLflow server cannot reach the SeaweedFS bucket. The SeaweedFS Filer confirms /buckets/mlflow-artifacts/ is still empty.<br>

3) Inspect /root/code/start-mlflow.sh and reconcile its environment with the SeaweedFS endpoint. Save the file, then restart the MLflow server:<br>
   ```test
   bash /root/code/restart-mlflow.sh
   ```
4) Re-run the smoke-test:<br>
   ```gf
   python3 /root/code/log_test_run.py
   ``` 
5) The end state must include:<br>
    * The test-remote experiment exists on the MLflow server with at least one successful run, visible in the MLflow UI.
    * The mlflow-artifacts bucket on SeaweedFS holds the run's model artefact (MLmodel + model.pkl), visible in the SeaweedFS Filer under
       /buckets/mlflow-artifacts/.
    * The PostgreSQL mlflow database holds the MLflow schema (the run's metadata).
PostgreSQL listens on port 5432 with a binary protocol — it is not reachable from a web browser. Use docker exec mlflow-db psql -U mlflow -d mlflow<br>
for manual inspection if needed.<br>
=> 
first i  Run the smoke test and observe the failure<br>
```yu
  python3 /root/code/log_test_run.py
```
then to see exact erro i run the command & i get The AWS Access Key Id you provided does not exist in our records.<br>
```il
grep -i -E "error|exception|s3|bucket|endpoint" /tmp/mlflow.log
```
here i don't have endopoint url  MLFLOW_S3_ENDPOINT_URL.i added in start-mlflow.sh<br>
```bash
#!/bin/bash
# Start the MLflow tracking server with the production-style wiring:
# - PostgreSQL backend for run metadata
# - SeaweedFS (S3-compatible) for artefact storage
# - host/CORS flags so the MLflow UI button works through the lab proxy
set -e

export AWS_ACCESS_KEY_ID=weedadmin
export AWS_SECRET_ACCESS_KEY=weedadmin123
export MLFLOW_S3_ENDPOINT_URL=http://localhost:8333

exec mlflow server \
  --backend-store-uri postgresql://mlflow:mlflow123@localhost:5432/mlflow \
  --artifacts-destination s3://mlflow-artifacts \
  --host 0.0.0.0 --port 5000 \
  --allowed-hosts '*' --cors-allowed-origins '*'
```
then run the following command to restart mlflow server<br> 
```io
bash /root/code/restart-mlflow.sh
```
After that run again the smoke test<br>
 ```kl
 python3 /root/code/log_test_run.py
 ```
 This is runs succesfully <br>
 I verified using:<br>
   1) test-remote experiment exist on mlflow server experiment section.
   2) then in SeaweedFS section check the mlflow-artifacts have artifacts MLmodel + model.pkl under /buckets/mlflow-artifacts/
   3) check the mlflow database holds the mlflow schema by using 
       docker exec mlflow-db psql -U mlflow -d mlflow
        \dt
        SELECT * FROM experiments;
        SELECT * FROM runs;
        \q   
