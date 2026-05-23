A teammate has added the transactions dataset to the xFusionCorp Industries fraud-detection repository, but it was committed directly to Git instead of being tracked with DVC. Bring the repository in line with the team standard—every dataset under data/ must be tracked by DVC, not by Git.


A project exists at /root/code/fraud-detection/ with DVC already initialised. The dataset data/raw/transactions.csv is currently tracked by Git, and the team standard requires DVC to own it instead.

Stop Git from tracking the dataset without deleting it from disk.

Track the same dataset with DVC so a .dvc pointer file is produced and data/raw/.gitignore excludes the dataset itself.

Stage the new .dvc pointer and the new .gitignore, then record a Git commit with the message Track transactions dataset with DVC.

Once tracking is moved to DVC, the DVC TRACKED section in the EXPLORER panel will list the dataset, confirming the extension recognises it as a DVC-managed file.

=> Go to the project directory 
```bash 
# cd /root/code/fraud-detection
To removed file from git tracking 
# git rm --cached data/raw/transactions.csv

To track files by dvc use this 
# dvc add data/raw/transactions.csv
after the previous command now dvc takes the ownership of the dataset.
It automatically creates both  point file .csv.dvc and  updates or creates .gitignore files .

Stage the .dvc pointer and the new .gitignore 
# git add data/raw/transactions.csv.dvc data/raw/.gitignore

Commit with message 
# git commit -m "Track transactions dataset with DVC"
```
DVC TRACKED bottom section in the explorer panel you will see the list of dataset.
