*ADF Trigger-File–Based Ingestion Orchestration (Model Project)*

This repository demonstrates a model solution for orchestrating data ingestion when multiple upstream sources deliver data at different times and with unpredictable processing durations.
Instead of relying on a fixed schedule, the pipelines use trigger-file–based orchestration to determine when all upstream data is ready for ingestion.

*Event Trigger*

The solution uses an event-based trigger on Azure Blob Storage to start the ingestion process automatically. The trigger monitors for files whose names end with job3_completed.csv.

	•	When this file arrives, the pipeline execution begins.
	•	The pipeline will only succeed if all required upstream files are present, even after the trigger fires.
	•	This ensures data integrity, preventing incomplete ingestion when dependent files are missing.


*Pipeline Architecture*

This solution uses three pipelines:


1️) Pipeline-1 (MultiTriggerFiles pipeline)

This is the controller pipeline responsible for continuously checking file availability before triggering ingestion.

*Key Components*

	•	Pipeline Parameter – flag
    Initially set to false. Only when this flag becomes true does the execution proceed.
	•	Until Activity
    Repeats until the flag becomes true.
	•	GetMetadata (Child Items)
    Retrieves the list of files from Blob Storage.
    Child items contain file names and metadata, which become the input to the next pipeline.
	•	ExecutePipeline → Executes File Availability Pipeline
    Passes the list of child items for validation.
	•	If Condition
    Evaluates whether the File Availability pipeline returned success or fail.
    
	     •	Success: sets the until flag to true
	     •	Fail: waits 60 seconds before retrying
       
	•	ExecutePipeline → Executes DataIngestion Pipeline
    Runs only after all conditions are satisfied and the trigger files are confirmed.


2️) Pipeline-2 (File Availability Pipeline)

This pipeline validates whether all required trigger files are present.

*Key Components*

	•	Pipeline Variable (Counter)
    Used to count how many required trigger files are detected.
	•	Switch Activity
    For each incoming file, checks whether it matches one of the expected trigger files.
    If matched → counter increments.
	•	If Condition
    Checks whether the counter equals the predefined number of required files.
	   •	Equal → sets pipeline status to success
	   •	Not Equal → sets pipeline status to fail

  This status is returned to the MultiTriggerFiles pipeline for decision-making.


3️) Pipeline-3 (DataIngestion Pipeline)

This is the ingestion pipeline executed only after all trigger files are validated.

*Key Components*

	•	Lookup Activity
    Uses a query to fetch active records from a configuration table.
	•	ForEach Activity
    Iterates through each active record.
	•	Copy Activity
    Moves the respective tables from SQL Database to Azure Data Lake Storage (Gen2).

*Summary*

This model project demonstrates how to implement dependency-driven ingestion pipelines when:

	•	Multiple upstream systems deliver data at different times
	•	Processing durations are unpredictable
	•	Time-based scheduling is unreliable

Trigger-file validation ensures that ingestion runs only when all upstream data is ready, making the solution reliable, scalable, and production-friendly.
