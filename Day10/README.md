The xFusionCorp Industries ML team is adopting DVC so that datasets and model files are versioned separately from code. Initialise DVC inside the existing Git repository at /root/code/fraud-detection/ and record the initialisation in Git.


A Git repository already exists at /root/code/fraud-detection/ with an initial commit.

Initialise DVC inside that repository so that the standard .dvc/ control directory and .dvcignore file are created alongside the existing Git working tree.

Stage every file that DVC produces during initialisation, and record them in a new Git commit with the message Initialize DVC.

Once initialisation is complete, the DVC extension will detect the new .dvc/ directory and surface the DVC TRACKED section in the EXPLORER panel together with a DVC indicator in the bottom status bar.
=> Firstly initialize dvc 
# dvc init 
After that you check .dvc and .dvcingnore file get created
# ls -a
stage every file that dvc produced.
# git add .dvc .dvcignore

Commit with message
# git commit -m "Initialize DVC"

After this verify in vs code editor DVC TRACKED in section explorer .
