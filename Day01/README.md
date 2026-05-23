The xFusionCorp Industries data science team needs a standardised Python environment for their new ML project.
Set up a virtual environment with the required ML libraries on the controlplane host.

Create a Python virtual environment named ml-env under /root/code/ using python3 -m venv.
Activate the environment and install the following packages: numpy, pandas, scikit-learn, and matplotlib.

Generate a requirements.txt file using pip freeze and save it at /root/code/requirements.txt.

=> First check which version of python is present.
   ```bash 
   # python --version
   ```
   Setup a Virtual Environment.
   ```ens
   # python3 -m venv ml-env
   ```
   Activate the environment and install the suggested packages.
   ```lsm
   # source ml-env/bin/activate
   # pip install numpy pandas scikit-learn matplotlib
   ```
   Generate a requirement.txt using pip freeze.
   ```free
   # pip freeze > requirements.txt
   ```
