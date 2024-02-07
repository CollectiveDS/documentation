# Guide to explain and create CDSHUB jobs

![Alt text](/../resources/images/deployment_jenkins_dashboard.png?raw=true "Dashboard Image")


## What is a job?

A job refers to a function that is executed at a certain time.  These can range from fetching (pulling data from an external source) and storing them in our AWS S3 bucket, using data we have in our S3 bucket to build a CSV and reupload in a new S3 area, etc.

## Job execution process

The job is defined in the MySQL database in the `cdshub/job_schedule` table with this format:
  
| job_name        	| payload_size 	| created_on          	| last_run_on         	| active 	| cron                    	|
|-----------------	|--------------	|---------------------	|---------------------	|--------	|-------------------------	|
| nielsen_reports 	| 1            	| 2022-01-11 15:49:54 	| 2023-01-21 12:00:00 	| 1      	| 0 0 12 * 7,14,21,28 * * 	|'

This table holds the name of the job, the cron schedule with which to run the job, the max number of entities in the job's payload, and whether or not the job is active. 

When a `job` is triggered by its `cron` schedule, the job is immediately sent to the `network` queue, which determines the job's payloads and queue (via `cdshub.service.queue`).

The `cdshub.server.network/network-callback` function determines the correct entities to use as payloads for the given job name, looks up those entities in the DB (if necessary), and spawns jobs using `cdshub.service.queue/spawn-msg`, which automatically determines the correct queue for the job.

The `cdshub.server.*` namespaces for fetch servers call a `cdshub.service.queue/register-jobs` function when defining job name -> callback fn hash-maps, which updates the job name -> queue mapping. Two functions, `cdshub.service.queue/spawn-msgs` and `cdshub.service.queue/spawn-msg` take a job name and one or more payloads, look up the appropriate queue from the job name -> queue mapping, builds messages for the job, and sends them to the appropriate queue. Moving a job from one server to another should only require moving the job from one `cdshub.server.*/jobs` hash-map to another; the job name -> queue mapping will be automatically updated and calls to `cdshub.service.queue/spawn-msg*` will automatically send the jobs to the appropriate queue.



## How to create a job

This assumes you have a function defined that the server queues can call.

1) Use `cdshub.service.queue/list-queues` to find whichever queue sounds like it fits your function the best
1) Add your function to the queue's callback map found in `cdshub.server.{QUEUE_NAME}`.  The names vary, e.g. `misc-callback`, `run-report`, etc. but they all have a similar structure that is easy to find.  Typically is a `case` statment where the `job_name` is the case under test.  Add your `job_name` as part of the `case` statement and call your function in the body when the job matches.
1) Make a SQL command to add your job to the `job_schedule` table, include it in the pull request of your ticket, and wait until peer review passes to execute and modify the prod database
    ``` 
    INSERT INTO job_schedule (job_name, payload_size, active, cron) VALUES ('{JOB_NAME}', {PAYLOAD_SIZE}, 1, '{CRON}');
    ```