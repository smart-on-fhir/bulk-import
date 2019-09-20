# Use Cases

## Large static dataset

I have a research dataset containing approximately 500,000,000 resources produced by an ETL pipeline. I want to import it as quickly as possible. Once imported the resources are never modified, but additional resources may be created (e.g. labeling for ML). The dataset has internal references that are mostly but not all consistent. If some or all of the dataset needs to be re-imported, I want to overwrite any resources currently in the store.

## Patient history import

[Castor EDC](https://www.castoredc.com/) allows the import of patient data from subjects included in a research study from the EHR via a FHIR endpoint. This includes both patient data from when the study is under way, but can also amount to large volumes of historic patient data. Currently these are retrieved via batch-bundles, which suffices for single-patient updates, but possibly the bulk import could be used for larger scale data exchanges, for example when importing data for a large retrospective cohort or registry study.

## Update a slave server for trial uses

Synchronization of a clinical with a clinical trial server. The clinical trial server can be used to without effecting the source database and will be updated regularly. E.g. with nightly jobs based on an $export with the `_since` parameter.

## Synchronize a product with the EMR

A clinical product with its own database synchronizes its data with an EMR. Updating the EMR with data and updating its internal data with EMR data.

## Import for offline processing

Retrieve a dump of medical data for offline processing, e.g. for measure calculation, de-identification, report generation, AI development, KPI calculation, ...

## Scalability

I have agreements with a large number of research dataset suppliers and they all implement pull capabilities in varying ways. They are all capable of producing FHIR output and saving it into ndjson, and I want to provide a consistent interface that all of them can use to deliver their datasets with minimal overhead for all parties. My data store on the other side of the interface does not support CDS and no other systems are making changes to the data store while the import datasets are being consumed.
