# Use Cases

## Large static dataset

I have a research dataset containing approximately 500,000,000 resources produced by an ETL pipeline. I want to import it as quickly as possible. Once imported the resources are never modified, but additional resources may be created (e.g. labeling for ML). The dataset has internal references that are mostly but not all consistent. If some or all of the dataset needs to be re-imported, I want to overwrite any resources currently in the store.

## Patient history import

[Castor EDC](https://www.castoredc.com/) allows the import of patient data from subjects included in a research study from the EHR via a FHIR endpoint. This includes both patient data from when the study is under way, but can also amount to large volumes of historic patient data. Currently these are retrieved via batch-bundles, which suffices for single-patient updates, but possibly the bulk import could be used for larger scale data exchanges, for example when importing data for a large retrospective cohort or registry study.

## Update a slave server for trial uses

Synchronization of a clinical with a clinical trial server. The clinical trial server can be used to without effecting the source database and will be updated regularly. E.g. with nightly jobs based on an $export with the _since parameter

## Synchronize a product with the EMR

A clinical product with its own database synchronizes its data with an EMR. Updating the EMR with data and updating its internal data with EMR data. 

## Import for offline processing

Retrieve a dump of medical data for offline processing, e.g. for measure calculation, de-identification, report generation, AI development, KPI calculation,  …

## Submitting Quality Reporting Data

A hospital system wants to send quality reporting data to a receiving system such as CMS. The hospital system has calculated their quality scores for a measure or set of measures for their covered patients. For each patient, the hosptial system has produced a Bundle containing a MeasureReport resource that identifies the score for the measure, as well as the relevant patient resources used to calculate the measure score.
