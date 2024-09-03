 **Architecture**  :-



![](images/storage/Actual%20Cassandra%20Arch.png)
*  **Kafka**  : Backend API will push the data for every update in the submission doc to Kafka topic. 

    Topics :   

      <env>.ml.project.submissions

      <env>.ml.observation.raw

     <env>.ml.survey.raw


    1.  Few samples with different status are shown below  

    i) Status : started 

    ii) Status : inProgress

    iii) Status : submitted/completed



    

      PFA : [Kafka Events](https://docs.google.com/document/d/12osMSYBpUmg3GI1Ufqq1hJiMfA507jYYAwyeRr5azgQ/edit#heading=h.5fc7vs732672)


*  **RealTime Script :**  Faust Python streaming




1. Read the events from kafka on real-time


1. Process and Transform the data based on the requirement for reports and csv’s


1. Query the Cassandra DB for the processed submission id’s, If exists update the data else insert the new processed data




*  **Exception Handling :-** 




1. In-case if Kafka is Down(secor-backup), how do we recover the lost data → Handle the Exception


1. In-case if cassandra DB is down → Handle the Exception


1. Exception Handling across all type’s of data (KeyError , NullPointerException ….) and throw the Exceptions in real-time on slack




*  **Cassandra**  : We use Cassandra DB to store our ML Raw Transactional Data, Table name : ml-project-raw






*  **OnDemand - CassandraExhaust - Custom - DataProduct Job** :

     **Logic :-** 




1. Get the JobRequest from the Postgres (params,type)


1. Get the Cassandra Query Config from the postgres dataset_metadata table


1. Dynamic Date(Start and End date) Range in the cassandra query


1. Dynamic Filter Update in the cassandra query


1. Execution of Cassandra Query


1. Replace “unknown” to “Null”


1. Sort the DataFrame Based on the given columns


1. Store the data as a zipped csv in the azure cloud storage, zip file should be encrypted with the password if the request contains password


1. Update the Azure(Cloud Storage) Url in the postgres back for the same requestId


1. User will download from the Program Dashboard Portal




*  **Schedular for the Scala Data Product Jobs :** - [https://github.com/Sunbird-Obsrv/sunbird-data-pipeline/blob/release-5.1.0/ansible/roles/data-products-deploy/templates/model-config.j2#L138-L141](https://github.com/Sunbird-Obsrv/sunbird-data-pipeline/blob/release-5.1.0/ansible/roles/data-products-deploy/templates/model-config.j2#L138-L141)

    


*  **Cassandra/Druid Config**  - dataset_metadata PostgreSQL Table Schema :-




```
dataset_id
dataset_sub_id
dataset_config
visibility
dataset_type
version
authorized_roles
available_from
sample_request
sample_response
validation_json
druid_query
limits
supported_formats
exhaust_type
```
 **Note :-**  We would require a changes to this PostgreSQL table :-

1. Renaming the column name from  **druid_query**  →  **query** , so we can store both druid and Cassandra generic query

2. Update the getDataSetDetails Function in OnDemandBaseExhaustJob.scala to select  **query**  col → sunbird-core-data-products git repository

3. Modify the below API to support the  **query**  config column:

 **API End Point**  :- /api/dataset/v1/add

 **Method**  :- POST


*  **Sample Cassandra Query Config**  :-




```
{
  "id": "ekstep.analytics.dataset.add",
  "ver": "1.0",
  "ts": "2016-12-07T12:40:40+05:30",
  "params": {
    "msgid": "4f04da60-1e24-4d31-aa7b-1daf91c46341"
  },
  "request": {
    "dataset": "cassandra-dataset",
    "datasetSubId": "ml-task-detail-exhaust",
    "datasetConfig": {
      "type": "ml-task-detail-exhaust",
      "params": {
        "programId": "program-1",
        "solutionId": "solution-1"
      }
    },
    "datasetType": "cassandra",
    "visibility": "private",
    "version": "v1",
    "authorizedRoles": [
      "PROGRAM_MANAGER",
      "PROGRAM_DESIGNER"
    ],
    "validationJson": {
      "type": "object",
      "properties": {
        "tag": {
          "id": "http://api.ekstep.org/dataexhaust/request/tag",
          "type": "string"
        },
        "dataset": {
          "id": "http://api.ekstep.org/dataexhaust/request/dataset",
          "type": "string"
        },
        "datasetSubId": {
          "id": "http://api.ekstep.org/dataexhaust/request/datasetSubId",
          "type": "string"
        },
        "requestedBy": {
          "id": "http://api.ekstep.org/dataexhaust/request/requestedBy",
          "type": "string"
        },
        "encryptionKey": {
          "id": "http://api.ekstep.org/dataexhaust/request/encryptionKey",
          "type": "string"
        },
        "datasetConfig": {
          "id": "http://api.ekstep.org/dataexhaust/request/datasetConfig",
          "type": "object"
        }
      },
      "required": [
        "tag",
        "dataset",
        "datasetConfig"
      ]
    },
    "cassandraQuery": {
      "id": "ml-task-detail-exhaust",
      "labels": {
        "block_name": "Block",
        "project_title_editable": "Project Title",
        "task_evidence": "Task Evidence",
        "user_type": "User Type",
        "designation": "User sub type",
        "school_code": "School ID",
        "project_duration": "Project Duration",
        "status_of_project": "Project Status",
        "sub_task": "Sub-Tasks",
        "tasks": "Tasks",
        "project_id": "Project ID",
        "project_description": "Project Objective",
        "program_externalId": "Program ID",
        "organisation_name": "Org Name",
        "createdBy": "UUID",
        "area_of_improvement": "Category",
        "school_name": "School Name",
        "board_name": "Declared Board",
        "district_name": "District",
        "program_name": "Program Name",
        "state_name": "Declared State",
        "task_remarks": "Task Remarks",
        "project_evidence": "Project Evidence",
        "project_remarks": "Project Remarks"
      }, 
      "db_name": "ml_transactional_data",
      "table_name": "ml_project",
      "columns": [
        "createdBy",
        "user_type",
        "designation",
        "state_name",
        "district_name",
        "block_name",
        "school_name",
        "school_code",
        "board_name",
        "organisation_name",
        "program_name",
        "program_externalId",
        "project_id",
        "project_title_editable",
        "project_description",
        "area_of_improvement",
        "project_duration",
        "status_of_project",
        "tasks",
        "sub_task",
        "task_evidence",
        "task_remarks",
        "project_evidence",
        "project_remarks"
      ],
      "columnOrder": true,
      "sort": [
        "UUID",
        "Program ID",
        "Project ID",
        "Tasks"
      ]
    },
    "supportedFormats": [
      "csv",
      "zip"
    ],
    "cloud_storage":{"type":"S3(Azure,GCP,Oracle)","storage_account":"xyz","bucket_name(container_name)":"abc","base_url":"http://s3-REGION-.amazonaws.com/BUCKET-NAME/KEY"}
    "exhaustType": "OnDemand"
  }
}
```
Few Transformation and Manipulation logic need to be handled :-


* Label Mapping


* Column Ordering


* Sorting the data based on the given sort columns


* Evidence Link Creation based on the FileSourcePath Data present in the cassandra and base_url from the cassandra query config







 **Cassandra Schema :-** 


```
CREATE KEYSPACE IF NOT EXISTS manage_learn WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': '1'
 };



CREATE TABLE IF NOT EXISTS manage_learn.ml_project (
       id text PRIMARY KEY,     
       program_id text,
       solution_id text,
       organisation_id text,
       project_id text,
       state_externalId text,
       block_externalId text,
       district_externalId text,
       cluster_externalId text,
       school_externalId text,
       task_id text,
       sub_task_id text,
       project_title text,
       project_goal text,
) WITH bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '64', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND dclocal_read_repair_chance = 0.1
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair_chance = 0.0
    AND speculative_retry = '99PERCENTILE';
```

## Requirements for Program Dashboard
For Projects -  users are currently using the below detailed reports. The [full list can be found here](https://github.com/shikshalokam/ml-analytics-service/blob/release-5.1.0/druid_data_product_query_config.txt).

Flow:
* Flatten data is stored in the Cassandra DB


* Scala data-product is used to capture, aggregrate, sort and filter the data


* The required data is stored in the cloud and is shared with the user





|  **Name**  |  **Columns**  |  **Sort**  | 
|  --- |  --- |  --- | 
|  **Project: Task detail**  _Shows the detailed information of submitted tasks details._  A single filter is used. status_of_project = 'submitted' Datasource: sl-projectLabels used:
```json
"block_name":"Block","project_title_editable":"Project Title","task_evidence":"Task Evidence","user_type":"User Type","designation":"User sub type","school_code":"School ID","project_duration":"Project Duration","status_of_project":"Project Status","sub_task":"Sub-Tasks","tasks":"Tasks","project_id":"Project ID","project_description":"Project Objective","program_externalId":"Program ID","organisation_name":"Org Name","createdBy":"UUID","area_of_improvement":"Category","school_name":"School Name","board_name":"Declared Board","district_name":"District","program_name":"Program Name","state_name":"Declared State","task_remarks":"Task Remarks","project_evidence":"Project Evidence","project_remarks":"Project Remarks","project_created_date":"Project start date of the user","project_completed_date":"Project completion date of the user"
```
 | "createdBy","user_type","designation","state_name","district_name","block_name","school_name","school_code","board_name","organisation_name","program_name","program_externalId","project_id","project_title_editable","project_description","area_of_improvement","project_created_date","project_completed_date","project_duration","status_of_project","tasks","sub_task","task_evidence","task_remarks","project_evidence","project_remarks" | In ascending order (based on labels):"UUID","Program ID","Project ID","Tasks" | 
|  **Project: Status detail**  _Shows the status of any tasks_ No filter is used.Datasource: sl-projectLabels used:
```json
"block_name":"Block","board_name":"Declared Board","project_title_editable":"Project Title","project_completed_date":"Project completion date of the user","user_type":"User Type","designation":"User sub type","school_code":"School ID","project_created_date":"Project start date of the user","project_last_sync":" Last Synced date","project_duration":"Project Duration","status_of_project":"Project Status","project_id":"Project ID","project_description":"Project Objective","program_externalId":"Program ID","organisation_name":"Org Name","createdBy":"UUID","school_name":"School Name","district_name":"District","program_name":"Program Name","certificate_status_customised":"Certificate Status"
```
 | "createdBy","user_type","designation","state_name","district_name","block_name","school_name","school_code","board_name","organisation_name","program_name","program_externalId","project_id","project_title_editable","project_description","project_created_date","project_completed_date","project_duration","project_last_sync","status_of_project","certificate_status_customised" | In ascending order (based on labels):"UUID","Program ID","Project ID" | 
|  **Project: Filtered-task detail**  _Shows a basic information of submitted tasks details._ A single filter is used. status_of_project = 'submitted'Datasource: sl-projectLabels used:
```json
"block_name":"Block","project_title_editable":"Project Title","task_evidence":"Task Evidence","user_type":"User Type","designation":"User sub type","school_code":"School ID","project_duration":"Project Duration","status_of_project":"Project Status","sub_task":"Sub-Tasks","tasks":"Tasks","project_id":"Project ID","project_description":"Project Objective","program_externalId":"Program ID","organisation_name":"Org Name","createdBy":"UUID","area_of_improvement":"Category","school_name":"School Name","board_name":"Declared Board","district_name":"District","program_name":"Program Name","state_name":"Declared State","task_remarks":"Task Remarks","project_evidence":"Project Evidence","project_remarks":"Project Remarks","project_created_date":"Project start date of the user","project_completed_date":"Project completion date of the user"
```
 | "createdBy","user_type","designation","state_name","district_name","block_name","organisation_name","program_name","project_title_editable","project_description","project_completed_date","status_of_project","tasks","task_evidence","task_remarks","project_evidence","project_remarks","task_sequence" | "District","Block","UUID","task_sequence" | 

Enhancements and Limitations:Problem: Druid allows aggregration of data but restricts data to be updated for any given datasource- which means the datasource needs to be deleted prior to reloading it with new data, bearing a risk of failures and delays.

Solution: With the integration with Cassandra we can easily update the data without the risk of deleting the datasource while also allowing neccesary aggregrations and sorts.


## Requirements for Admin Dashboard
A detailed description about Reports are provided here: 

[https://docs.google.com/spreadsheets/d/1iUC0ZNJq_cqSeaHcWAoRP04MQ7f2I7C9T9HH0KgEpFA/edit#gid=1500982806](https://docs.google.com/spreadsheets/d/1iUC0ZNJq_cqSeaHcWAoRP04MQ7f2I7C9T9HH0KgEpFA/edit#gid=1500982806) In summary, the following metrics are currently in use:


* Sum of Unique Users 


* Sum of Unique Entities


* Sum of Unique Projects


* Sum of Unique Submissions


* Sum of Improvement Projects submitted with evidence



noteAdmin Dashboard can access a file directly stored in the Cloud Storage and display the respective values on the frontend.

Admin Dashboard can access a file directly stored in the Cloud Storage and display the respective values on the frontend.

One of the challenges currently in Admin Dashboard comes down to the fact that unique users requires aggregration through code and not by quering the datasource directly. Here are few ideas that can be implemented:


* We can use real-time query on the Cassandra to get data.


* We can also store a JSON file with the required data metrics and access them directly in the frontend without any depedency on the backend. 







*****

[[category.storage-team]] 
[[category.confluence]] 
