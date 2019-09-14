# Use Cases

## Large static dataset

I have a research dataset containing approximately 500,000,000 resources produced by an ETL pipeline. I want to import it as quickly as possible. Once imported the resources are never modified, but additional resources may be created (e.g. labeling for ML). The dataset has internal references that are mostly but not all consistent. If some or all of the dataset needs to be re-imported, I want to overwrite any resources currently in the store.

## Scalability

I have agreements with a large number of research dataset suppliers and they all implement pull capabilities in varying ways. They are all capable of producing FHIR output and saving it into ndjson, and I want to provide a consistent interface that all of them can use to deliver their datasets with minimal overhead for all parties. My data store on the other side of the interface does not support CDS and no other systems are making changes to the data store while the import datasets are being consumed.
