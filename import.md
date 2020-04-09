# Proposal for $import Operation

## Overview
There are diverse use cases for a bulk-level FHIR data import capability -- including server backup/restore, migration of data between FHIR servers and the aggregation of data from multiple sources to create a combined data set. For some use cases, a single batch import suffices; for others, the ability to bring in incremental data over time is required.

This implementation guide complements the [FHIR Bulk Data Export Operation IG](http://build.fhir.org/ig/HL7/bulk-data/export/index.html).

Servers may wish to address the following activities as part of an import process, however, standardization of them is out of scope for the initial release of this implementation guide:
- Incorporation of incremental changes
- De-duplication or merging of data from multiple sources
- Filtering or selective inclusion of specific resources
- Authorization / access control (implementors may choose to adopt the [FHIR backend services IG](http://build.fhir.org/ig/HL7/bulk-data/authorization/index.html) for this purpose.

## Request Flow

### Bulk Data Import Kick-Off Request

`POST [fhir base]/$import`

#### Headers

- `Content-Type: application/json` (fixed value, required)
- `Accept: application/fhir+json` (fixed value, required)
- `Prefer: respond-async` (fixed value, required)

#### Body JSON Fields

- `inputFormat` (string, required)

	Servers SHALL support [Newline Delimited JSON](http://ndjson.org) with a format type of `application/fhir+ndjson` but MAY choose to support additional input formats.

- `inputSource` (uri, required)

	URI for tracking this set of imported data throughout its lifecycle. MAY be used to specify a FHIR endpoint that can by the importing system when matching references to previously imported data.

- `input` (json array, required), array of objects containing the following fields
	- `type` (string, required) FHIR resource type
	- `url` (url, required) Path to bulk data file of the type reflected in `inputFormat` containing FHIR resources

- `storageDetail` (object, optional) - Defaults to type of "https" with no parameters specified
	- `type` (string, required)
	- Parameters depending on type:

		| type   | parameters |
		|--------|------------|
		|`https` | - `contentEncoding` (array of strings, optional) |
		|`aws-s3`| |
		|`gcp-bucket`| |
		|`azure-blob`| |

```
NOTES: @nikolai: I would introduce mode: batch | transaction - which means the same as in FHIR transaction - with batch mode each loaded file will be imported without waiting for others ?!
```

#### Example Request
```
POST $import
Content-Type: application/json
Accept: application/fhir+json
Prefer: respond-async
```
```json
{
	"inputFormat": "application/fhir+ndjson",
	"inputSource": "https://other-server.example.org",
	"storageDetail": { "type": "https" },
	"input": [{
		"type": "Patient",
		"url": "https://client.example.org/patient_file_2.ndjson?sig=RHIX5Xcg0Mq2rqI3OlWT"
	},{
		"type": "Observations",
		"url": "https://client.example.org/obseration_file_19.ndjson?sig=RHIX5Xcg0Mq2rqI3OlWT"
	}]
}
```

#### Response - Success

- HTTP Status Code of `202 Accepted`
- `Content-Location` header with the absolute URL of an endpoint for subsequent status requests (polling location)
- Optionally, a FHIR OperationOutcome resource in the body

#### Response - Error

- HTTP Status Code of ```4XX``` or ```5XX```
- The body SHALL be a FHIR OperationOutcome resource in JSON format

If a server wants to prevent a client from beginning a new import before an in-progress import is completed, it SHOULD respond with a `429 Too Many Requests` status and a `Retry-After` header, following the rate-limiting advice for "Bulk Data Import Status Request" below.

---
### Bulk Data Import Delete Request

After a bulk data import request has been started, a client MAY send a DELETE request to the URL provided in the ```Content-Location``` header to cancel the request.

#### Endpoint

`DELETE [polling content location]`

#### Response - Success

- HTTP Status Code of ```202 Accepted```
- Optionally a FHIR OperationOutcome resource in the body

#### Response - Error Status

- HTTP status code of ```4XX``` or ```5XX```
- The body SHALL be a FHIR OperationOutcome resource in JSON format

---
### Bulk Data Import Status Request

After a bulk data import request has been started, the client MAY poll the status URL provided in the ```Content-Location``` header.

Clients SHOULD follow an [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) approach when polling for status. Servers SHOULD supply a [Retry-After header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) with a http date or a delay time in seconds. When provided, clients SHOULD use this information to inform the timing of future polling requests. Servers SHOULD keep an accounting of status queries received from a given client, and if a client is polling too frequently, the server SHOULD respond with a `429 Too Many Requests` status code in addition to a Retry-After header, and optionally a FHIR OperationOutcome resource with further explanation.  If excessively frequent status queries persist, the server MAY return a `429 Too Many Requests` status code and terminate the session. Other standard HTTP `4XX` as well as `5XX` status codes may be used to identify errors as mentioned.

When requesting status, the client SHOULD use an ```Accept``` header indicating a content type of  ```application/json```. In the case that errors prevent the import from completing, the server SHOULD respond with a FHIR OperationOutcome resource in JSON format.

#### Response - In-Progress Status

- HTTP Status Code of ```202 Accepted```
- Optionally, the server MAY return an ```X-Progress``` header with a text description of the status of the request that's less than 100 characters. The format of this description is at the server's discretion and may be a percentage complete value, or a more general status such as "in progress". The client MAY parse the description, display it to the user, or log it.

```
NOTES (May Connectathon):

Consider more details in interim status response, e.g. # resources processed, or failed, or % of total file bytes processed, # resources already available / visible to clients… e.g., structure may help, but may also vary across servers depending on architecture.

Shared core of a status response: pending, submitted, time remaining, etc

There's some Task-y stuff about this. Or of course Parameters...

Consider Parameters[] rather than an ad-hoc JSON payload here.

Currently there's no status in this message or, e.g.,  a set of flags for the server to indicate things like "it's okay to delete this input file now, or okay to revoke server's access now."
```

#### Response - Error Status

- HTTP status code of ```5XX```
- ```Content-Type``` header of ```application/json```
- The server SHALL return a FHIR OperationOutcome resource in JSON format

*Note*: Even if some of the requested resources cannot successfully be imported, the overall import operation MAY still succeed. In this case, the `Response.error` array of the completion response body SHALL be populated with one or more files in ndjson format (or other format if specified in kick-off request) containing FHIR `OperationOutcome` resources to indicate what went wrong (see below). In the case of a partial success, the server SHALL use a 200 status code instead of 5XX. The choice of when to determine that an import job has failed in its entirety (error status) vs returning a partial success (complete status) is left up to the implementer.

#### Response - Complete Status

- HTTP status of ```200 OK```
- ```Content-Type``` header of ```application/json```
- The server MAY return an ```Expires``` header indicating when the OperationOutcome bulk data files listed will no longer be available for access (if applicable).
- A body containing a JSON object providing metadata, and optional links to OperationOutcome bulk data files with detailed information about the operation. The files SHALL be accessible to the client at the URLs advertised. These URLs MAY be served by file servers other than a FHIR-specific server.

#### Required Fields

- `transactionTime` (FHIR instant, required)
```
NOTE: need to decide what time transactionTime represents
```

- `request` (url, required)
	
	The full URL of the original bulk data import kick-off request

- `output` (array, required)

	Array of objects that correspond 1:1 with the `input` array from request that contain the following fields:
	- `input`  (required, url) Url to corresponding file in kick-off request
	- `count` (optional, integer) The number of resources **successfully** processed, represented as a JSON number.
	- `url` (optional, url) Optional link to a file of `OperationOutcome` Resources with results of each successful FHIR resource import
	- `type` (required if `url` is included, string) Only the `OperationOutcome` resource type is currently supported, so a server SHALL generate files in the same format as bulk data import files that contain `OperationOutcome` resources.

- `error` (array, optional)

	Array of error file items that correspond 1:1 with the `input` array from request and follow the same structure as the `output` array. Errors that occurred during the import should only be included here (not in output). If no errors occurred, the server SHOULD return an empty array.  Only the `OperationOutcome` resource type is currently supported, so a server SHALL generate files in the same format as bulk data import files that contain `OperationOutcome` resources.

- `extension` (object, optional)
 
 	To support extensions, this implementation guide reserves the name `extension` and will never define a field with that name, allowing server implementations to use it to provide custom behavior and information. For example, a server may choose to provide a custom extension that contains a decryption key for encrypted ndjson files. The value of an extension element SHALL be a pre-coordinated JSON object.
    
#### Example Response

```
200 OK
Content-Type: application/json
```
```
{
	"transactionTime": "[instant]",
	"request": "[base]/$import", // do we need more context? 

	"output": [{
		"type": "OperationOutcome", // these correspond to the Patient input file
		"count": 612523,
		"inputUrl": "https://client.example.org/patient_file_2.ndjson", 
		"url": // optional link to the success results
	},{
		"type": "OperationOutcome", // these correspond to the Observation input file
		"count": 81972498,
		"inputUrl": "https://client.example.org/obseration_file_19.ndjson", 
		"url": // optional link to the success results
	}],
	"error": [{ 
		"type": "OperationOutcome", // these correspond to the Patient input file
		"count": 51,
		"inputUrl": "https://client.example.org/patient_file_2.ndjson", 
		"url": "http://serverpath2/err_file_1.ndjson"
	},{
		"type": "OperationOutcome", // these correspond to the Observation input file
		"count": 3,
		"inputUrl": "https://client.example.org/obseration_file_19.ndjson",
		"url": "http://serverpath2/err_file_1.ndjson"
	}]
}
```

### Guidance for servers

#### Preserving Resource provenance

If `Resource.meta.source` is absent, servers SHOULD populate `Resource.meta.source` of each resource with the inputSource value from the initial request.

#### Preserving Resource ids
By default, client-supplied IDs should be preserved when importing resources. If this is not possible, the server MAY re-map imported resource ids, but MUST decorate all remapped resources by appending an extension to Resource.meta:

```json
"extension": [{
	"url": "http://hl7.org/fhir/StructureDefinition/import-source",
	"valueIdentifier": [{
		"system": "[inputSource from kick-off request]",
		"identifier": "[original resource.resourceType]/[original Resource.id]"
	}]
}]
```

#### Validation and referential integrity

Servers MAY disable referential integrity checks and profile validation checks during a bulk import, and may report any issues in the "error" array of the import results.

Servers MAY choose to tolerate some number of errors (e.g., invalid entries in an ndjson file) without aborting the entire import operation. However, if a server decides to abort the entire import operation because of a fatal error, or because an error tolerance is exceeded, then the import response should include a "count" of 0 for every output entry, and any error or warning information should be conveyed in the "error" array.

When a server is importing data into a live FHIR store, it may not be able to roll back a partial import if a fatal error is encountered (e.g., if an input file is no longer available).
