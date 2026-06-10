A xFusionCorp Industries data scientist has accumulated ten runs in the fraud-detection MLflow experiment.<br> 
Your task is to triage those runs via the MLflow UI: mark the single best-performing candidate as the shortlisted model,<br>  
and flag every clearly under-performing run for removal.<br> 


1) The MLflow tracking server is already running on port 5000, and the fraud-detection experiment has been pre-populated with ten runs. <br> 
   The runs can be viewed via the MLflow UI button → fraud-detection experiment.<br> 

2) Using the MLflow UI, complete the triage below. The end state is what is tested—the path taken through the UI is not.<br> 

    * Shortlist the best candidate. Among all runs where metrics.f1_score > 0.85, the single run with the highest f1_score must carry a <br> 
      run-level tag: key review-status, value shortlisted.<br> 

    * Reject the under-performers. Every run where metrics.f1_score < 0.75 must carry a run-level tag: key review-status, value rejected.<br> 

3) The other runs (those in the 0.75 ≤ f1 ≤ 0.85 band, and the second-best shortlisting candidate) must carry no review-status tag at all.<br>

=> 
To complete this lab in the MLflow UI:<br>
   1) Open the MLflow UI (http://localhost:5000) and select the fraud-detection experiment.<br>
   2) Sort the runs by f1_score (descending).<br>
   3) Identify:<br>
           * The single run with the highest f1_score among all runs where f1_score > 0.85.
           * All runs where f1_score < 0.75.
          
**Add the shortlisted tag**

For the best run:<br>
  1) Open the run.
  2) Add a run tag:
      * Key: review-status
      * Value: shortlisted
    
**Add the rejected tag**

For every run with f1_score < 0.75:
  1) Open the run.<br>
  2) Add a run tag:
      * Key: review-status
      * Value: rejected

**Important**
* Do not tag any other runs.
* Runs with 0.75 ≤ f1_score ≤ 0.85 must have no review-status tag.
* The second-best run above 0.85 must also have no review-status tag.
