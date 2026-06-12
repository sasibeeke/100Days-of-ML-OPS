The xFusionCorp Industries ML platform team needs two trained candidates promoted through the MLflow Model Registry so the ops side can track which<br>
model version is serving production traffic. Both runs already exist in the fraud-detection experiment. Your task is to register both as versions of 
a new fraud-detector model, add a model-level description, and assign challenger and champion aliases—all through the MLflow UI.<br>


The MLflow tracking server is already running on port 5000 and two runs are pre-populated in the fraud-detection experiment: a baseline run<br>
(n_estimators=100, max_depth=5, f1_score=0.80) and an improved run (n_estimators=200, max_depth=10, f1_score=0.89). Both runs can be opened via the <br>
MLflow UI button → fraud-detection experiment.<br>

Using the MLflow UI, reach the end state below. The order (baseline first, improved second) matters because MLflow assigns version numbers <br>
sequentially within a registered model.<br>

A registered model named fraud-detector exists in the Model Registry.<br>
  1) The registered model carries a non-empty description that references the word fraud (any phrasing; for example Fraud detection model<br>
     for xFusionCorp transactions).<br>
  3) Version 1 of fraud-detector is the baseline run and carries the alias challenger.<br>
  4) Version 2 of fraud-detector is the improved run and carries the alias champion.<br>
  
The result can be confirmed by opening Model registry → fraud-detector in the MLflow UI. Two versions are listed, the description is shown at the <br>
top of the model page, and the alias column (or the Aliases field on each version) indicates challenger on v1 and champion on v2.<br>
=> 
**1. Open the MLflow UI**
      Open: http://localhost:5000
Go to the fraud-detection experiment.<br>

**2. Register the baseline run first (creates Version 1)**
Locate the run with:<br>

Parameter	    Value
n_estimators	100
max_depth	    5
f1_score	    0.80

Open the run.<br>
  1) Find the logged model/artifact section.<br>
  2) Click Register Model.<br>
  3) Choose Create New Model.<br>
  4) Enter:<br>
        fraud-detector
  5) Save/Register.<br>
      This creates:
        fraud-detector v1

**3. Register the improved run second (creates Version 2)**

Return to the experiment.<br>
Locate the run with:<br>

Parameter	    Value
n_estimators	200
max_depth	    10
f1_score	    0.89

Open the run.<br>
  1) Click Register Model.
  2) Select Existing Registered Model.
  3) Choose:
        fraud-detector

  4) Register.
      This creates:
        fraud-detector v2

Because the baseline was registered first, MLflow assigns:<br>
v1 = baseline
v2 = improved

4. Add a model description<br>
Navigate to:

Model Registry → fraud-detector

Click the description/edit (pencil) icon.<br>
Enter any non-empty description containing the word fraud, for example:<br>

  Fraud detection model for xFusionCorp transactions.

Save.<br>

5. Assign alias to Version 1<br>
Open Version 1.

In the Aliases section:<br>

  challenger

Save.<br>

6. Assign alias to Version 2<br>
Open Version 2.

In the Aliases section:<br>

  champion

Save.<br>

Final verification<br>

In Model Registry → fraud-detector, you should see:<br>
v1	baseline (100,5,0.80)	challenger
v2	improved (200,10,0.89)	champion

And at the top of the model page:<br>
Fraud detection model for xFusionCorp transactions.(or any description containing the word fraud).<br>
