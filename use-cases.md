# Use Cases

## Large static dataset

I have a research dataset containing approximately 500,000,000 resources produced by an ETL pipeline. I want to import it as quickly as possible. Once imported the resources are never modified, but additional resources may be created (e.g. labeling for ML). The dataset has internal references that are mostly but not all consistent. If some or all of the dataset needs to be re-imported, I want to overwrite any resources currently in the store.

#### Seeding test data
We'd like to use import to allow clients to efficiently seed test data. The import operation would be best-effort, and invalid resources would be skipped in the import process. Imported resources would not be accessible via the REST API until the import operation had completed.