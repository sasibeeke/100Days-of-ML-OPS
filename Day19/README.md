Complete the xFusionCorp Industries fraud-detection production DVC pipeline. Three stages are already wired in dvc.yaml, two remain, and the pipeline<br>
must finish as a reproducible, SeaweedFS-backed, v1.0-tagged release.<br>


1) A project exists at /root/code/ml-pipeline/ with Git and DVC initialised. The params.yaml is in place and the .dvc/config is pre-configured to push <br>
to the SeaweedFS bucket dvc-storage at http://localhost:8333.<br>

2) The ingest, validate, and preprocess stages are already declared in dvc.yaml, but one of them contains an incorrect output path that prevents dvc <br>
repro from completing. Find and fix it.<br>

3) The remaining two stages need to be added:<br>

  * train – Depends on the preprocessed dataset and scripts/train.py; reads n_estimators, max_depth, test_size, and random_seed from params.yaml;<br>
            outputs models/model.pkl and data/processed/test_split.csv; declares metrics.json as a DVC metric with cache: false.<br>
  * evaluate – Depends on models/model.pkl, data/processed/test_split.csv, and scripts/evaluate.py; outputs reports/evaluation.json declared with cache: false.<br>
4) The two scripts you need are pre-staged at /root/code/ml-pipeline/scripts-staging/train.py and scripts-staging/evaluate.py.<br>
    Copy them into scripts/ before adding the stages.<br>

5) Run the full pipeline with dvc repro, push the cache to the SeaweedFS remote with dvc push, and tag the current state as v1.0.<br>

6) Commit every change to Git so the release is fully captured.<br>

Note: Open the SeaweedFS Filer button at the top of the lab and navigate to /buckets/dvc-storage/ to confirm that the bucket holds the pushed artefacts <br>
under the files/md5/... layout.<br>

=> cat dvc.yaml
```bash
stages:
  ingest:
    cmd: python scripts/ingest.py
    deps:
      - scripts/ingest.py
      - data/raw/data.csv

  validate:
    cmd: python scripts/validate.py
    deps:
      - data/raw/data.csv
      - scripts/validate.py
    outs:
      - reports/validation.json:
          cache: false

  preprocess:
    cmd: python scripts/preprocess.py
    deps:
      - data/raw/data.csv
      - scripts/preprocess.py
    outs:
      - data/processed/cleaned.csv
```
it gives issue of in preprocess stage.
Running stage 'preprocess':                                               
> python scripts/preprocess.py
Preprocessed: 20 clean rows
ERROR: failed to reproduce 'preprocess': output 'data/processed/cleaned.csv' does not exist

corrected dvc.yaml
```bash
stages:
  ingest:
    cmd: python scripts/ingest.py
    deps:
      - scripts/ingest.py
      - data/raw/data.csv

  validate:
    cmd: python scripts/validate.py
    deps:
      - data/raw/data.csv
      - scripts/validate.py
    outs:
      - reports/validation.json:
          cache: false

  preprocess:
    cmd: python scripts/preprocess.py
    deps:
      - data/raw/data.csv
      - scripts/preprocess.py
    outs:
      - data/processed/clean.csv

  train:
    cmd: python scripts/train.py
    deps:
      - scripts/train.py
      - data/processed/clean.csv
    params:
      - n_estimators
      - max_depth
      - test_size
      - random_seed
    outs:
      - models/model.pkl
      - data/processed/test_split.csv
    metrics:
      - metrics.json:
          cache: false

  evaluate:
    cmd: python scripts/evaluate.py
    deps:
      - scripts/evaluate.py
      - models/model.pkl
      - data/processed/test_split.csv
    outs:
      - reports/evaluation.json:
          cache: false 
```
Copy the staged scripts
```ax
cp scripts-staging/train.py scripts/
cp scripts-staging/evaluate.py scripts/   
```
added this two stages 
```rf
train:
    cmd: python scripts/train.py
    deps:
      - scripts/train.py
      - data/processed/clean.csv
    params:
      - n_estimators
      - max_depth
      - test_size
      - random_seed
    outs:
      - models/model.pkl
      - data/processed/test_split.csv
    metrics:
      - metrics.json:
          cache: false

  evaluate:
    cmd: python scripts/evaluate.py
    deps:
      - scripts/evaluate.py
      - models/model.pkl
      - data/processed/test_split.csv
    outs:
      - reports/evaluation.json:
          cache: false
```
To rerun pipeline do 
```re
# dvc repro
```
Commit DVC changes
```sa
git add .
git commit -m "Complete fraud detection DVC pipeline"
```
Push artifacts to SeaweedFS
```qs
dvc push
git tag -a v1.0 -m "Version 1.0"
git tag
dvc push
```          
      

      

