The xFusionCorp Industries ML team keeps different dataset and model versions on different Git branches so that the team can roll between versions cleanly. <br>
Tag the current state as v1.0, produce a v2-improved branch based on a newer dataset, and confirm that switching back restores the original data.<br>


1) A project exists at /root/code/fraud-detection/ with a working DVC pipeline and the baseline data/raw/transactions.csv already tracked.<br>

2) An improved dataset has been pre-staged at /root/code/fraud-detection/data/raw/transactions_v2.csv and is visible in the file explorer. Do not delete this file.<br>

3) On the main branch, tag the current state as v1.0.<br>

4) Create a new branch named v2-improved. Replace the tracked dataset with the contents of the v2 file, re-track it with DVC, re-run the pipeline, and commit the changes.<br>

5) Switch back to the main branch and use dvc checkout to restore the v1 dataset on disk. The restored content must match the hash recorded by the v1.0 tag.<br>

The DVC extension's DVC TRACKED section in the EXPLORER panel will reflect the current branch's tracked state—it should show different dataset hashes on main and 
v2-improved.<br>
=> 
# Ensure we're on main
```test
git checkout main
```
# Tag current state as v1.0
```re
git tag v1.0
```
# Create and switch to the new branch
```qw
git checkout -b v2-improved
```

# Replace the tracked dataset with the improved version
```ty
cp data/raw/transactions_v2.csv data/raw/transactions.csv
```

# Re-track the dataset with DVC
```qvg
dvc add data/raw/transactions.csv
```

# Re-run the pipeline
```bash
dvc repro
```
Now switch back to main and restore the original tracked data:<br>
```lo
git checkout main
```
# Restore DVC-tracked files for main
```lt
dvc checkout
```
To confirm that the restored content matches the version recorded by the v1.0 tag:<br>

# Compare current main version against the tagged version
```yu
git diff v1.0 -- data/raw/transactions.csv.dvc
```
No output from the above command means the DVC metadata matches the tagged v1.0 state.<br>

You can also inspect the DVC hashes directly:<br>
# Current main hash
```jk
grep md5 data/raw/transactions.csv.dvc
```

# Hash stored in v1.0 tag
```jsj
git show v1.0:data/raw/transactions.csv.dvc | grep md5
```

The MD5 values should be identical on main after dvc checkout.<br>

# Commit the updated dataset and model artifacts
```uo
git add .
git commit -m "Use improved dataset and retrain model"
```
