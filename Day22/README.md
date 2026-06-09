The xFusionCorp Industries ML platform team is onboarding two new ML projects and needs each one organised under its own MLflow experiment rather <br>
than sharing the Default experiment. Your task is to register both experiments through the MLflow UI and tag them with the owning team.<br>


1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to view the dashboard.<br>
   One seeded experiment (legacy-models) is listed alongside the platform-created Default—both act as reference material and must not be modified.<br>

2) Using the MLflow UI, register two new experiments with the experiment-level metadata below. <br>
   The task is complete when both records satisfy every bullet.<br>

    * fraud-detection<br>
        Experiment-level description is a non-empty string describing the project (any phrasing).<br>
        Experiment-level tag: key team, value ml-platform.<br>
    
    * churn-prediction<br>
        Experiment-level tag: key team, value analytics.<br>
    
The result can be confirmed in the MLflow UI: both new experiments appear in the left-hand list, with the description and tags visible on each <br>
experiment's page.<br>

=>
**Step 1: Open MLflow UI<br>**

Open the MLflow tracking server UI (port 5000) using the lab’s MLflow UI button.<br>

**Create Experiment 1: fraud-detection<br>**
1) In the left sidebar, click “Experiments” (or “+ New Experiment” depending on UI version).<br>
2) Click Create Experiment.<br>
3) Fill in:<br>
    * Name: fraud-detection<br>
    * Artifact Location: leave default (unless required)<br>
4) Click Create<br>

**Add Description<br>**
1) Open the newly created fraud-detection experiment.<br>
2) Go to Overview / Settings / Edit (gear icon or “Edit Experiment”).<br>
3) Add:<br>
       * Description: something like Fraud detection ML experiment for transaction anomaly classification<br>
4) Save.<br>

**Add Tag<br>**
1) In the same experiment page, find Tags section.<br>
2) Add:<br>
    * Key: team<br>
    * Value: ml-platform<br>
3) Save.<br>

**Create Experiment 2: churn-prediction<br>**
Repeat the same steps:<br>
1) Click Create Experiment<br>
2) Name: churn-prediction<br>
3) Create the experiment<br>

**Add Tag<br>**
1) Open churn-prediction<br>
2) Go to Tags<br>
3) Add:<br>
  * Key: team<br>
  * Value: analytics<br>
4) Save.<br>

(Description is not explicitly required for this one in the task, so you can leave it blank unless the UI forces it—in that case add any short <br>
text like “Customer churn prediction model tracking”.)<br>

<img width="1599" height="868" alt="image" src="https://github.com/user-attachments/assets/d9a787cc-7935-4bbf-81e1-2062fd3c7726" />
