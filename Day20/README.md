The xFusionCorp Industries ML team is adopting MLflow for experiment tracking.<br>
Your task is to bring up a local MLflow tracking server on the ML pipeline workstation so experiments can be logged from the team's training code.<br>


MLflow 3.x is pre-installed on the controlplane. Launch the tracking server in the background so that every end-state requirement below holds.<br>

1) The server is listening on port 5000 and is reachable on all interfaces.<br>

2) The backend store is a SQLite database at /root/code/mlflow-backend/mlflow.db. The database file must exist after the server has started.<br>

3) The artifact root is /root/code/mlflow-artifacts/.<br>

4) Any parent directories the server needs must be in place before it starts—MLflow will abort if the backend directory is missing.<br>

5) The MLflow UI button at the top of the lab must open a responsive dashboard in the browser. The button routes through the lab proxy, so the server<br>
   must accept requests from any origin (--cors-allowed-origins '*') and any host header (--allowed-hosts '*') to avoid proxy-related rejections.<br>

6) The server process must persist in the background so it survives terminal closure.<br>

  Once the server is running, the Default experiment can be viewed from the MLflow UI button. The experiment is empty—runs will be <br>
  logged in subsequent labs.<br>
  => firstly i checked is there any process running with give port & also mlflow-artifacts,mlflow-backend directories are not present.
  ```bash 
  # ss -tulpn | grep 5000
  # mkdir -p /root/code/mlflow-backend
  # mkdir -p /root/code/mlflow-artifacts
```
Start MLflow tracking server in background <br>
```se
  # nohup mlflow server \   
    --host 0.0.0.0 \ 
    --port 5000 \  
    --backend-store-uri sqlite:////root/code/mlflow-backend/mlflow.db  \
    --artifacts-destination /root/code/mlflow-artifacts  \
    --cors-allowed-origins "*" \
    --allowed-hosts "*" \
    > /root/mlflow-server.log 2>&1 &
```

Verify that it started:<br>
```ts
# ps -ef | grep mlflow
``
Verify the database file exists:<br>
```sq
# ls -l /root/code/mlflow-backend/mlflow.db
```
Verify the server is listening on port 5000:<br>
```yu
# ss -tulpn | grep 5000
```
At last hit the button MLflow UI button you will see the mlflow ui will displayed.
