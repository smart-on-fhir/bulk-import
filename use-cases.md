# Use Cases

## Large static dataset

I have a research dataset containing approximately 500,000,000 resources produced by an ETL pipeline. I want to import it as quickly as possible. Once imported the resources are never modified, but additional resources may be created (e.g. labeling for ML). The dataset has internal references that are mostly but not all consistent. If some or all of the dataset needs to be re-imported, I want to overwrite any resources currently in the store.

## Patient history import

[Castor EDC](https://www.castoredc.com/) allows the import of patient data from subjects included in a research study from the EHR via a FHIR endpoint. This includes both patient data from when the study is under way, but can also amount to large volumes of historic patient data. Currently these are retrieved via batch-bundles, which suffices for single-patient updates, but possibly the bulk import could be used for larger scale data exchanges, for example when importing data for a large retrospective cohort or registry study.
