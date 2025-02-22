# Dataclenz Documentation

### Process Map

<img src ="charts/Dataclenz ProcessMap.drawio (1).png">


## Backend


### *Definitions*
---

**FileStatus** represents the status of an uploaded file.

   FileStatus | Value | Definition
   --- |---|---
   INACTIVE | 0 | No jobs (operations/services) are running
   QUEUED | 1 | The file is queued to run (it will run the next queued job)
   ACTIVE | 2 | A job is currently running from this file; runs until all jobs are completed or an error occurs
   COMPLETED | 3 |All jobs have been completed
   ERROR | 4 | An error occurred during processing
<br/>
 
**JobStatus** represents the status of an individual job (operation on the file).

  JobStatus | Value | Definition
  --- |---|---
  QUEUED | 0 |Ready to run (the file runs the earliest queued job)
  ACTIVE | 1 | The job's operation is currently running
  COMPLETED | 2 | The job has completed and the file data has been updated
  ERROR | 3 | An error occurred during the job, and the file data was updated
<br/>

**OperationType** defines the type of operation to perform on file data.
 *NOT IMPLEMENTED*
   OperationType | Value | Definition
   --- |---|---
   VALIDATE | 0 | Appends a single column of boolean results (e.g., checks integer validity, nulls, etc.)
   APPLY | 1 | Replaces values (e.g., replace columns, remove rows, fill zeroes)
   CALCULATE | 2 | Appends a column of calculated results (e.g., sum, concatenate)
<br/>

**ComparisonOperator** defines operators used for comparing column selections to values or another columns.
 *NOT IMPLEMENTED*
   ComparisonOperator | Value | Definition
   --- |---|---
   NONE | 0 | No comparison operator is applied
   GREATER_THAN | 1 | >
   LESS_THAN | 2 | <
   EQUAL_TO | 3 | ==
   NOT_EQUAL_TO | 4 | !=
   IN_ARRAY | 5 | Check if value is in an array
   NOT_IN_ARRAY | 6 | Check if value is not in an array
<br/>

### *Models*
---

**File** represents an uploaded file and holds both the file data and its associated job IDs.
  Key | Type | Description
  --- |---|---
  id | number | Auto-generated unique identifier for the file
  user | number | ID of the user who uploaded the file
  file_name | string | Name of the file
  created | string | ISO timestamp when the file was uploaded
  data | any | The file's data stored as JSON (e.g., an array of row objects)
  status | FileStatus | Current status of the file (using FileStatus enum)
  updated | string | ISO timestamp of the last update (e.g., after a job completes or errors)
  jobs | Job[] | List of Job s associated with this file (array of Job objects)
  active_job_id? | number *(default: null)* | (Optional) The ID of the currently running job (if any)
<br/>


**Job** represents an operation executed on file data. It includes details about which operation to perform, column selections, and comparison criteria.
  Key | Type | Description
  --- |---|---
  id | number | Unique job identifier
  file | number | ID of the File to which this job is attached
  index | number | Position of this job within the file's job queue
  status | JobStatus |Current status of the job (using JobStatus enum)
  updated | string | ISO timestamp for the last update (e.g., when a job block completes)
  operation_id | number | Identifier for the operation to execute
  operation_column_indices? | number[] *(default: null)* | Array of column indices to apply the operation (null means all columns)
  operation_comparison_operator | ComparisonOperator | Operator to compare column values (using ComparisonOperator enum - DEFAULTS TO 0 or NONE) *NOT IMPLEMENTED*
  operation_comparison_value? |string *(default: null)* |The value to compare to (if applicable) *NOT IMPLEMENTED*
  operation_comparison_column_index? | number *(default: null)* | The column index to compare to (if applicable) *NOT IMPLEMENTED*
  result_info | any | Contains result info (e.g., processed row counts, errors, etc.)
  data | any | Output data after the operation (e.g., summed values); may be empty for certain operations
<br/>


**Operation** defines an available operation that can be performed on file data.
  Key | Type | Description
  --- |---|---
  id | number | Unique identifier for the operation
  name | string | Descriptive name of the operation
<br/>

### *Routes* 
---
**Upload and create a new File**
* **POST** `/api/files/create_file/`
* Creates a new File
* Required:
  * `file`
  * authorize with token in the header OR `user_id`
* Response:
  * `File` the newly created File
  * `{"message": <error>}` if error

**Get all Files**
* **GET** `/api/files/get_files/`
* gets all Files associated with the currently logged in user or a `user_id`
* Required:
  * authorize with token in the header OR `user_id`
* Response:
  * `File[]` array of File objects
  * `{"message": <error>}` if error

**Get single File**
* **GET** `/api/files/get_file/`
* gets a single File associated with the currently logged in user or a `user_id`
* Required:
  * `file_id` the id of the file to fetch
  * authorize with token in the header OR `user_id`
* Response:
  * `File` the requested File
  * `{"message": <error>}` if error

**Queue File**
* **POST** `/api/files/queue_file/`
* sets a File's status to QUEUED
* Required:
  * `file_id` the id of the file to queue
  * authorize with token in the header OR `user_id`
* Response:
  * `File` the File if successfully queued
  * `{"message": <error>}` if error

**Create a new Job for File**
* **POST** `/api/jobs/create_job/`
* Creates a new `Job` to attach to a File (appends to jobs)
* Required:
  * `file_id` the id of the File to attach to
  * `operation_inputs` an Array with the operation parameters:
    * `operation_id` the id of the Operation
    * `column_indices[]` an Array of column indices
  * authorize with token in the header OR `user_id`
* Response:
  * `Job` the newly created Job (status is QUEUED)
  * `{"message": <error>}` if error

**Get all Operations**
* **GET** `/api/operations/get_operations/`
* gets all Operations
* Required:
  * Nothing
* Response:
  * `Operation[]` array of Operation objects
  * `{"message": <error>}` if error

**Get single Operation**
* **GET** `/api/operations/get_operation/`
* gets a single Operation
* Required:
  * `operation_id` the id of the file to fetch
* Response:
  * `Operation` the requested Operation
  * `{"message": <error>}` if error

