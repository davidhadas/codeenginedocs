# Update a job
{: #update-job}

You can make changes to a defined job and run the updated job from the console or with the CLI. Changes to your job might include updating the code container image, code arguments or commands, runtime instance resources, or environment variables.
{: shortdesc}

Each time your job runs, the most current version of your referenced container image is downloaded and run. Submitted batch jobs are run in parallel, if possible. If the number or size of the submitted jobs exceeds the configured quota limits, such as maximum number of running instances, then {{site.data.keyword.codeengineshort}} queues the jobs and delays running them until enough jobs finish. For more information about quotas and limits for jobs, including memory and CPU, see [Limits and quotas for {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-limits).
{: note}

## Updating a job from the console
{: #update-job-ui}

When the job is in ready state, you can update the job. Let's update the `myjob` job that you created previously to change the container image from `ibmcom/firstjob` to `ibmcom/testjob` and then subsequently update an environment variable. When a request is sent to this [`ibmcom/testjob`](https://hub.docker.com/r/ibmcom/testjob){: external} sample job, the job reads the environment variable `TARGET` and prints `"Hello ${TARGET}!"`. If this environment variable is empty, `"Hello World!"` is returned. 

1. Navigate to your job page. 
   * From the [{{site.data.keyword.codeengineshort}} Projects page](https://cloud.ibm.com/codeengine/projects){: external}, click the name of your project. Click **Jobs** to open a listing of your jobs.   
   * From the Jobs page, click the name of the job that you want to update. 

2. To update the image reference of your job, provide the name of your image or configure an image. Update the name of the image for this job from `ibmcom/firstjob` to `ibmcom/testjob`. Click **Save**. 
3. Click **Submit job**.
4. From the Submit job pane, accept all of the default values, and click **Submit job** again to run your job.  
5. By [viewing job logs from the console](/docs/codeengine?topic=codeengine-view-logs#view-joblogs-ui) for this job, the output of the job is `Hello World!`.
6. To update the job again and add an environment variable, navigate to your job page. 
7. Click **Environment variables** to open the tab and then click **Add**. Add a literal environment variable with the name of `TARGET` with a value of `Sunshine`. The `ibmcom/testjob` outputs the message, `Hello <value_of_TARGET>!>`.
8. Click **Add**.
9. Click **Save** to save the update to the job. 
10. Click **Submit job**.
11. From the Submit job pane, this time change the CPU value to one of the valid CPU choices for the specified memory. Notice that you must use [valid memory and CPU combinations](/docs/codeengine?topic=codeengine-mem-cpu-combo). Click **Submit job** again to run this job. The system displays the status of your job on the Job details page. Notice the updated CPU value in the `Configuration` section of the job details.
12. By [viewing job logs from the console](/docs/codeengine?topic=codeengine-view-logs#view-joblogs-ui) for this job, the output of the updated job is `Hello Sunshine!`.

## Updating a job with the CLI
{: #update-job-cli}

With the CLI, you can update an existing job configuration or specify your changes on a new job run. 

### Updating a job configuration with the CLI
{: #update-jobconfig-cli}

You can update an existing job configuration with the [**`ibmcloud ce job update`**](/docs/codeengine?topic=codeengine-cli#cli-job-update) command. 

1. Use the **`job update`** command to update the `myjob` job to reference a different image. 

```
ibmcloud ce job update --name myjob --image ibmcom/testjob 
```
{: pre}

Run the `ibmcloud ce job get -n myjob` command to display details of the updated job.

**Example output**

```
Name:          myapp
ID:            abcdefgh-abcd-abcd-abcd-1a2b3c4d5e6f
Project Name:  myproject
Project ID:    01234567-abcd-abcd-abcd-abcdabcd1111
Age:           3m6s
Created:       2021-06-04T11:56:22-04:00

Image:                ibmcom/testjob
Resource Allocation:
  CPU:     1
  Memory:  4G

Runtime:
  Array Indices:       0
  Max Execution Time:  7200
  Retry Limit:         3
```
{: screen}

2. Run the [**`ibmcloud ce jobrun submit`**](/docs/codeengine?topic=codeengine-cli#cli-jobrun-submit) command to run a job that references this updated job configuration. For the **`jobrun submit`** command, use the `--job` option to reference a defined job configuration. While the `--name` option is not required if the `--job` option is specified, the following example command specifies the `--name` option to provide a name for this job run. 

```
ibmcloud ce jobrun submit --name myjobrun1 --job myjob
```
{: pre}

3. Run **`ibmcloud ce jobrun get -n myjobrun1`** to view details of this job run. Note that the referenced image is `ibmcom/testjob`, which is based on the updated job configuration.

**Example output**

```
Getting jobrun 'myjobrun1'...
Getting instances of jobrun 'myjobrun1'...
Getting events of jobrun 'myjobrun1'...
OK

Name:          myjobrun1
ID:            abcdefgh-abcd-abcd-abcd-1a2b3c4d5e6f
Project Name:  myproject
Project ID:    01234567-abcd-abcd-abcd-abcdabcd1111
Age:           3m6s
Created:       2021-06-15T11:56:22-04:00

Job Ref:              myjob
Image:                ibmcom/testjob
Resource Allocation:
  CPU:                1
  Ephemeral Storage:  4G
  Memory:             4G

Runtime:
  Array Indices:       0
  Max Execution Time:  7200
  Retry Limit:         3

Status:
  Completed:          11s
  Instance Statuses:
    Succeeded:  1
  Conditions:
    Type      Status  Last Probe  Last Transition
    Pending   True    15s         15s
    Running   True    11s         11s
    Complete  True    11s         11s

Events:
  Type    Reason     Age                Source                Messages
  Normal  Updated    16s (x3 over 20s)  batch-job-controller  Updated JobRun "myjobrun1"
  Normal  Completed  16s                batch-job-controller  JobRun completed successfully

Instances:
  Name           Running  Status     Restarts  Age
  myjobrun1-0-0  0/1      Succeeded  0         20s
```
{: screen}

### Updating a job run with the CLI  
{: #update-jobrun-cli}

You can specify changes for a job run with the [**`ibmcloud ce jobrun resubmit`**](/docs/codeengine?topic=codeengine-cli#cli-job-update) command. The **`jobrun submit`** command resubmits a job run that references a previous job run. 

1. Use the **`jobrun resubmit`** command to resubmit the `myjobrun1` job run and change the array indices from `0` to `1-4`. While the `--name` option is not required for the **`jobrun resubmit`** command, the following example command specifies the `--name` option to provide a name for this job run. 

```
ibmcloud ce jobrun resubmit -jobrun myjobrun1 --array-indices "1-4" --name myjobrunresubmit
```
{: pre}

2. Run the `ibmcloud ce jobrun get -n myjobrunresubmit` command to display details of the updated job run. Note that the value of array indices is updated for this job run.

**Example output**

```
Getting jobrun 'myjobrunresubmit'...
Getting instances of jobrun 'myjobrunresubmit'...
Getting events of jobrun 'myjobrunresubmit'...
OK

Name:          myjobrunresubmit2. 
ID:            abcdefgh-abcd-abcd-abcd-1a2b3c4d5e6f
Project Name:  myproject
Project ID:    01234567-abcd-abcd-abcd-abcdabcd1111
Age:           3m6s
Created:       2021-06-04T11:56:22-04:00

Job Ref:              myjob
Image:                ibmcom/testjob
Resource Allocation:
  CPU:                1
  Ephemeral Storage:  4G
  Memory:             4G

Runtime:
  Array Indices:       1-4
  Max Execution Time:  7200
  Retry Limit:         3

Status:
  Completed:          34s
  Instance Statuses:
    Succeeded:  4
  Conditions:
    Type      Status  Last Probe  Last Transition
    Pending   True    38s         38s
    Running   True    36s         36s
    Complete  True    34s         34s

Events:
  Type    Reason     Age                Source                Messages
  Normal  Updated    36s (x7 over 40s)  batch-job-controller  Updated JobRun "myjobrunresubmit"
  Normal  Completed  36s                batch-job-controller  JobRun completed successfully

Instances:
  Name                  Running  Status     Restarts  Age
  myjobrunresubmit-1-0  0/1      Succeeded  0         40s
  myjobrunresubmit-2-0  0/1      Succeeded  0         40s
  myjobrunresubmit-3-0  0/1      Succeeded  0         40s
  myjobrunresubmit-4-0  0/1      Succeeded  0         40s
```
{: screen}

Job runs that are submitted (or resubmitted) with the CLI that do not reference a defined job configuration are not viewable from the console. 
{: note}
