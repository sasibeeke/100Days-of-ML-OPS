The xFusionCorp Industries ML team uses uv and lockfiles to keep Python dependencies reproducible across machines. A teammate has left behind a requirements.in specification that does not match the team's standard. Correct it and compile it into a pinned lockfile.

A high-level dependency specification exists at /root/code/fraud-detection/requirements.in. uv is already installed.

The corrected specification must meet the following requirements:

It lists exactly these four top-level packages: scikit-learn, mlflow, pandas, and numpy;
every package carries a version constraint that uv can actually satisfy against PyPI.
Review the existing requirements.in, and correct everything that does not match the requirements above.

From the project directory, compile the corrected specification into a pinned lockfile:

   uv pip compile requirements.in -o requirements.txt

The resulting requirements.txt must pin each of the four top-level packages to an exact version using ==, and must also include the transitive dependencies that uv resolved.

=> first check the version of uv
   # uv --version
   Then do the change in requirements.in accroding to requirement
   # vi requirements.in 
    # Fraud detection project dependencies
    scikit-learn
    mlflow
    pandas
    numpy
   After this perform the following command 
   # uv pip compile requirements.in -o requirements.txt
   after this it gives requirements.txt with exact match versions of packages with transitive dependencies.  