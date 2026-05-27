The xFusionCorp Industries ML team uses DVC pipelines to keep data processing reproducible.<br>
A draft dvc.yaml exists in the fraud-detection project, but dvc repro does not complete the full pipeline.<br> 
Correct the pipeline definition so it runs cleanly end to end.<br>


1) A project exists at /root/code/fraud-detection/ with DVC initialised.<br>
Python scripts are at src/data/process_data.py and src/data/split_data.py;<br>
raw input is at data/raw/transactions.csv. Do not modify the Python files or the input data.<br>

2) The corrected pipeline must declare two stages with the following behaviour:<br>

  - process_data – Depends on <br>
    data/raw/transactions.csv and <br> src/data/process_data.py; produces <br> data/processed/clean_transactions.csv.
  - split_data – Depends on <br> data/processed/clean_transactions.csv and <br> src/data/split_data.py; produces <br> data/processed/train.csv and <br> data/processed/test.csv.<br>

3) Review the existing dvc.yaml and correct everything that prevents dvc repro from completing.<br>

4) After your changes, dvc repro must run end to end and dvc status must report no stale stages.<br>

Once the pipeline is valid, the DVC extension's PIPELINES <br> section under the DVC view will list both stages and visualise <br> the dependency graph between them.<br>
=> previous dvc.yaml<br>
 # cat dvc.yaml
 ```bash
 stages:
  process_data:
    cmd: python src/data/process.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean.csv

  split_data:
    cmd: python src/data/split_data.py
    deps:
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv
```
correctd dvc.yaml
```test
stages:
  process_data:
    cmd: python src/data/process_data.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean_transactions.csv

  split_data:
    cmd: python src/data/split_data.py
    deps:
      - data/processed/clean_transactions.csv
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv
```
Then after that run 
```tu
# dvc repo
# dvc status
```
after dvc status if it shows dvc pipelines are up to date then it works fine.
