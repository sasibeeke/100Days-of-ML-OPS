A xFusionCorp Industries data scientist has trained three candidate models on the same problem and logged them to the model-comparison experiment.<br> 
Your task is to review the candidates side by side in the MLflow UI and explicitly mark the winning run so downstream tooling can pick it up.<br>


1) The MLflow tracking server is already running on port 5000 and the model-comparison experiment has been pre-populated with three runs, each named<br>
   after its algorithm (RandomForest, GradientBoosting, LogisticRegression) and carrying accuracy and f1_score metrics.<br>
   The runs can be viewed via the MLflow UI button → model-comparison experiment.<br>

2) Using the MLflow UI, inspect the three runs side by side and identify the winner by metrics.f1_score.<br>
      *  The run with the highest f1_score must carry a run-level tag: key production-candidate, value true.<br>
      *  Neither of the other two runs may carry a production-candidate tag.<br>
The result can be confirmed in the MLflow UI: the model-comparison experiment lists three runs, and only the top-f1_score run shows
the production-candidate tag on its detail page.<br>
=> 
This task is done entirely in the MLflow UI.<br>
**Steps**
    1) Open the MLflow UI:<br>
          http://localhost:5000
    2) Open the model-comparison experiment.
    3) In the runs table, compare the three runs:
         * RandomForest
         * GradientBoosting
         * LogisticRegression
    4) Check the f1_score metric for each run and identify the run with the highest f1_score.
    5) Click that winning run to open its details page.
    6) In the Tags section:<br>
        * Add a new tag<br>
            * Key: production-candidate
            * Value: true
    7) Verify that:<br>
          * Only the winning run has the production-candidate=true tag.
          * The other two runs do not have a production-candidate tag. If one exists, delete it.

**Validation**
After saving:<br>
  * Return to the experiment view.
  * Open each run's details page.
  * Confirm that exactly one run (the one with the highest f1_score) contains:
        production-candidate = true
  * The other two runs contain no production-candidate tag.
