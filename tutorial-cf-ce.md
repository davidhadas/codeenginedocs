---

copyright:
  years: 2022
lastupdated: "2022-03-19"

keywords: code engine, tutorial, build, source, application, buildpack, access, build run, image, cloud foundry

subcollection: codeengine

content-type: tutorial
completion-time: 30m 

---

{{site.data.keyword.attribute-definition-list}}

# Migrating Cloud Foundry applications to {{site.data.keyword.codeengineshort}}
{: #migrate-cf-ce-tutorial}
{: toc-content-type="tutorial"}
{: toc-completion-time="30m"}

{{site.data.keyword.codeenginefull}} is a fully managed, serverless platform that runs your containerized workloads. {{site.data.keyword.codeengineshort}} even builds container images for you from your source code. The {{site.data.keyword.codeengineshort}} experience is designed so that you can focus on writing code and not on the infrastructure that is needed to host it.
{: shortdesc}

{{site.data.keyword.codeengineshort}} was designed with the following key goals in mind.

- Provide a first class, simplified developer experience where the developer does not need to learn about or manage the underlying infrastructure.
- Support the modern runtime characteristics that people expect today, such as autoscaling, scaling to zero when idle, and seamless integration with managed services and security.
- Support all your Cloud-native based applications, whether they are [12-factor apps](https://12factor.net/){: external}, event-driven functions, or run-to-completion batch-jobs. If it can be containerized, {{site.data.keyword.codeengineshort}} can run it.
- Pay for only the resources that you actually use.

If you are coming from a Cloud Foundry background, many of these features sound familiar. However, {{site.data.keyword.codeengineshort}} also includes some new capabilities that allow you to go beyond even what Cloud Foundry offers, including applications that can scale to zero and incur no charges. So, after you're past the syntactical difference of the user experience, you and your applications will feel right at home in {{site.data.keyword.codeengineshort}}.

Tutorials might incur costs. Use the Cost Estimator to generate a cost estimate based on your projected usage.
{: note}

## Objectives
{: #cf-objectives}

- Learn the similarities between {{site.data.keyword.codeengineshort}} and Cloud Foundry.
- Learn the general process of deploying apps in {{site.data.keyword.codeengineshort}}.
- Deploy an application from code on your local system with {{site.data.keyword.codeengineshort}}.

## Terminology
{: #terminology}

Before getting started with deploying apps in {{site.data.keyword.codeengineshort}}, learn the basics about {{site.data.keyword.codeengineshort}}. The following table describes some high level terminology differences between Cloud Foundry and {{site.data.keyword.codeengineshort}}.

| Cloud Foundry | {{site.data.keyword.codeengineshort}} | Description |
| -------------- | -------------- | -------------- |
| `Org` and `Space` | Resource group and projects | A grouping of workloads. The specific choice of which workload goes into each grouping is user defined. "Resource groups" are an {{site.data.keyword.cloud_notm}} concept, while "projects" are {{site.data.keyword.codeengineshort}} specific. Projects provide a level of isolation between workloads. See [Creating a project](#create-project). |
| Application | Application (app) | A workload that responds to HTTP requests regardless of the semantics, or purpose behind the request be it a REST API call, a web page request, or an event. {{site.data.keyword.codeengineshort}} requires that applications include the HTTP server as part of the code. Applications automatically scale (up and down) based on the incoming load. You can configure the minimum and maximum scale if needed. By default, the application listens on port 8080 by default. You can override this behavior by using the console or with the CLI `--port` flag. See [Working with apps in {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-application-workloads). |
| N/A | Job or batch job | Any workload that does not listen for HTTP requests, but instead is invoked on demand, and then exits when completed. Batch jobs can be scaled, but they are not scaled automatically. Instead, you can specify the number of instances that you want when you run your job. Jobs are often called "run to completion" workloads. See [Working with jobs and job runs](/docs/codeengine?topic=codeengine-job-plan). |
| `cf push` | Build | Process of building a container image from source code. You can build code based on a Dockerfile or use a Paketo buildpack. Building source code in {{site.data.keyword.codeengineshort}} is a distinct step from deploying a workload in the CLI, unlike `cf push` which does it all in one step. You can build and deploy an application from a single step from the {{site.data.keyword.codeengineshort}} console. See [Planning your build](/docs/codeengine?topic=codeengine-plan-build). |
| Service binding | Service binding | Attach a workload to an {{site.data.keyword.cloud_notm}} managed service. The credentials and connection information is exposed to the workload through environment variables. The `CF_SERVICES` environment variable in Cloud Foundry is called `CE_SERVICES` in {{site.data.keyword.codeengineshort}}. See [Integrating {{site.data.keyword.cloud_notm}} services with service bind](/docs/codeengine?topic=codeengine-service-binding). |
| Routes and domains | N/A | Define and manage external URLs to your workloads. {{site.data.keyword.codeengineshort}} provides this capability implicitly. You can add [custom domains](/docs/codeengine?topic=codeengine-deploy-multiple-regions) through {{site.data.keyword.cis_full_notm}}. |
{: caption="Table 1. Terminology" caption-side="bottom"}

For additional terms and capabilities for {{site.data.keyword.codeengineshort}}, see [Learn about {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-about).

## Getting started with {{site.data.keyword.codeengineshort}}
{: #cf-get-started}

Now that you know the basic terminology of {{site.data.keyword.codeengineshort}} and how it compares to Cloud Foundry, let's deploy a simple `hello world` app to see how {{site.data.keyword.codeengineshort}} works. First, set up your account and install the CLI. Then, follow the steps to create a project, build your code, and deploy your app.

### Prerequisites
{: #cf-prereqs}

Before you can get started with {{site.data.keyword.codeengineshort}}, you need to set up your account and install the CLI.

- All {{site.data.keyword.codeengineshort}} users are required to have a Pay-as-you-Go account.

- While you can use {{site.data.keyword.codeengineshort}} through the console, the examples in this documentation focus on the command line. Therefore, you must install the {{site.data.keyword.codeengineshort}} CLI.

    ```txt
    ibmcloud plugin install code-engine
    ```
    {: pre}

    For more information, see [setting up the {{site.data.keyword.codeengineshort}} CLI](/docs/codeengine?topic=codeengine-install-cli). For more information about the CLI, see [{{site.data.keyword.codeengineshort}} CLI reference](/docs/codeengine?topic=codeengine-cli).

## Creating a project
{: #create-project}
{: step}

Your first step is to create a {{site.data.keyword.codeengineshort}} project. A project is similar to a Cloud Foundry "space" in that it groups related workloads into a logical collection that is meaningful to the developer. You can group workloads into different projects based on whatever criteria that makes sense to you, for example, company organization structure, dependencies between the workloads, or development versus test versus production environments. Keep in mind that workloads within a single project share a private network and are isolated within the security boundary of the project. All workloads within a project can talk to each other freely without concern of being seen by workloads outside of the cluster. If workloads in different projects want to communicate with each other, then the communication must either use the internet or an internal IBM private network. For more information, see [Options for visibility for a {{site.data.keyword.codeengineshort}} application](/docs/codeengine?topic=codeengine-application-workloads#optionsvisibility).

    
Create a project in {{site.data.keyword.codeengineshort}} called `sample`.

```txt
ibmcloud ce project create --name sample
```
{: pre}
    
**Example output**
    
```txt
Creating project 'sample'...
ID for project 'sample' is 'abcdabcd-abcd-abcd-abcd-abcd12e3456f7'.
Waiting for project 'sample' to be active...
Now selecting project 'sample'.
OK
```
{: screen}
   
Notice that your project is also selected for context, so all subsequent application-related commands are within the scope of this new `sample` project.

## Creating a directory and source code
{: #create-dir}
{: step}

1. Create a directory on your local workstation that is named `myapp` and navigate into it. In this directory, save all the files that are required to build the image and to run your app.
        
    ```txt
    mkdir myapp && cd myapp
    ```
    {: pre}
        
2. Create a file called `server.js` and copy the following source code into it.

    ```javascript
    const http = require('http');


    http.createServer(function (request, response) {
      response.writeHead(200, {'Content-Type': 'text/plain'});
      response.end( "Hello world\n" );
    }).listen(8080);
    ```
    {: pre}
    
        
## Creating a container registry namespace
{: #create-cr-namespace}
{: step}

In Cloud Foundry, your application is packaged and provided to the hosting environment automatically. In {{site.data.keyword.codeengineshort}}, you must build or package your source code into a container image and then store that container image in a namespace, or folder, in a registry that {{site.data.keyword.codeengineshort}} can access. This example uses {{site.data.keyword.registryshort}}. 

1. Set your {{site.data.keyword.registryshort}} to the same region that you created your project in. For example, to set your region to `us-south`, run the following command.

    ```txt
    ibmcloud cr region-set us-south
    ```
    {: pre}
    
    **Example output**
        
    ```txt
    The region is set to 'us-south', the registry is 'us.icr.io'.
    ```
    {: screen}
    

2. Create your registry namespace. The following command creates a namespace called `myapp-images`. However, as namespace names must be globally unique, add your name or initials to it. 

    ```txt
    ibmcloud cr namespace-add myapp-images<INITIALS>
    ```
    {: pre}
        
    **Example output**
        
    ```txt
    Adding namespace 'myapp-images' in resource group 'default' for account John Doe's Account in registry us.icr.io...
    Successfully added namespace 'myapp-images'
    OK
    ```
    {: screen}
    
For more information, see [add registry access documentation](/docs/codeengine?topic=codeengine-add-registry#add-registry-access-ce).    
 
## Creating an API key
{: #create-apikey}
{: step}

Set up access to your namespace by creating an API key. 

```txt
ibmcloud iam api-key-create myapp-icr-apikey
```
{: pre}

**Example output**

```txt
Creating API key myapp-icr-apikey under 7f9dc5344476457f2c0f53244a246d44 as johndoe@example.com...
Please preserve the API key! It cannot be retrieved after it's created.
OK
API key myapp-icr-apikey was created


                 
ID            ApiKey-d688eb25-2b35-4641-a75f-079d8f627b8a   
Name          myapp-icr-apikey   
Description      
Created At    2022-02-04T15:02+0000   
API Key       LAzmpcrTc6OCIgiqxuwH5-frnTFiDth6bkiuGN19X9hP   
Locked        false  
```
{: screen}

## Storing access to your registry in {{site.data.keyword.codeengineshort}}
{: #store-registry-access}
{: step}

Store access to your registry in {{site.data.keyword.codeengineshort}}. Registry access is used to store access credentials for container registries. Replace the `password` value of `LAzmpcrTc6OCIgiqxuwH5-frnTFiDth6bkiuGN19X9hP` with the `API Key` value returned in the previous command.

```txt
ibmcloud ce registry create --name icr --password LAzmpcrTc6OCIgiqxuwH5-frnTFiDth6bkiuGN19X9hP --server us.icr.io
```
{: pre}

**Example output**

```txt
Creating image registry access secret 'icr'...
OK
```
{: screen}

For more information, see [add registry access documentation](/docs/codeengine?topic=codeengine-add-registry#add-registry-access-ce).

## Creating a build definition for your application
{: #create-build-def}
{: step}

Create a build, which defines how you want the source code to be built and where to store the image.

```txt
ibmcloud ce build create --name myapp-build --image us.icr.io/myapp-images<INITIAL>/myapp --registry-secret icr --strategy buildpacks --build-type local
```
{: pre}

**Example output**

```txt
Creating build 'myapp-build'...
OK
```
{: screen}

Let's briefly discuss the options.

`--name` 
:    Specifies the name of the build definition. You can use this name to refer back to the build definition when you want to run the build.

`--image`
:    Specifies the location in {{site.data.keyword.registryshort}} to store the resulting image.

`--registry-secret` 
:    The registry access credentials that you created in the previous step.

`--strategy`
:    Defines the method that Code Engine uses to build the image. This build uses ` buildpack`, a method similar to Cloud Foundry. Another valid type is `dockerfile`. For more information, see [Choose a build strategy](/docs/codeengine?topic=codeengine-plan-build#build-strategy).

`--build-type`
:    Indicates whether the source code can be found locally or in a Git repository. In this example, use `local`. For more information, see [Prepare your source location](/docs/codeengine?topic=codeengine-plan-build#build-plan-repo).

## Running your build
{: #run-build-submit}
{: step}

After your create your build definition, submit your build run. This example references the previously created build definition and provides the location for the source code for this specific build in the current directory (`.`). The `--wait` option indicates for build run to wait for the build to finish rather than running the command in the background.

```txt
ibmcloud ce buildrun submit --build myapp-build --source . --wait
```
{: pre}

**Example output**

```txt
Getting build 'myapp-build'
Packaging files to upload from source path '.'...
Submitting build run 'myapp-build-run-220204-150228492'...
Waiting for build run to complete...
Build run status: 'Running'
Build run completed successfully.
Run 'ibmcloud ce buildrun get -n myapp-build-run-220204-150228492' to check the build run status.
OK
```
{: screen}

## Deploying your application
{: #deploy-application}
{: step}

Now that you created your container image, you can deploy our application. Notice that since the registry that you are using is {{site.data.keyword.registryshort}}, which is a private registry, you must pass in the registry credentials so that {{site.data.keyword.codeengineshort}} can access and download your image.

```txt
ibmcloud ce app create --name myapp --image us.icr.io/myapp-images/myapp --registry-secret icr
```
{: pre}

**Example output**

```txt
Creating application 'myapp'...
The Route is still working to reflect the latest desired specification.
Configuration 'myapp' is waiting for a Revision to become ready.
Ingress has not yet been reconciled.
Waiting for load balancer to be ready.
Run 'ibmcloud ce application get -n myapp' to check the application status.
OK


https://myapp.bqos0nxl0jo.us-south.codeengine.appdomain.cloud
```
{: screen}

The `--name` option specifies the name for the application.  At the end of the output is the URL that you can use to connect to your app.

And that's it. You now have an internet-facing application. The code in the application itself is the same as what is used for a Cloud Foundry application, it's just the {{site.data.keyword.codeengineshort}} commands that are slightly different.

With {{site.data.keyword.codeengineshort}}, you automatically get many of the same features as Cloud Foundry, such as autoscaling and blue-green roll-out of updates, but you'll also enjoy the benefits of newer features such as scaling down-to-zero, ensuring that you are not charged if your application is not active.

Now that you've deployed a sample application to {{site.data.keyword.codeengineshort}}, learn more details about how to migrate your existing workloads from Cloud Foundry to {{site.data.keyword.codeengineshort}}.


## What types of workloads are available with {{site.data.keyword.codeengineshort}}?
{: #workloads}

{{site.data.keyword.codeengineshort}} supports two types of workloads: Applications and Batch Jobs. 

An application, or app, runs your code to serve HTTP requests. In addition to traditional HTTP requests, {{site.data.keyword.codeenginefull}} also supports applications that use WebSockets as their communications protocol. The number of running instances of an app are automatically scaled up or down (to zero) based on incoming workloads and your configuration settings. An app contains one or more revisions. A revision represents an immutable version of the configuration properties of the app. Each update of an app configuration property creates a new revision of the app. 

A job runs one or more instances of your executable code. Unlike applications, which handle HTTP requests, jobs are designed to run one time and exit. When you create a job, you can specify workload configuration information that is used each time that the job is run.

### Determining the type of workloads that you want
{: #determine-type}

Most Cloud Foundry applications can migrated to be {{site.data.keyword.codeengineshort}} applications. However, if your Cloud Foundry application does not wait for an incoming HTTP request, then jobs are probably the better choice. For more discussion and examples, see [Planning for {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-plan-codeengine).

## I use manifest files. Are there similar options available with {{site.data.keyword.codeengineshort}}?
{: #manifest}

If you use manifest files for your Cloud Foundry applications, map your manifest attributes to the corresponding {{site.data.keyword.codeengineshort}} features or CLI option.

| Manifest attribute | {{site.data.keyword.codeengineshort}} equivalent on the `ibmcloud ce app create` command |
| -------------- | -------------- |
| command | `--command` option |
| disk_quota | Implicitly set by {{site.data.keyword.codeengineshort}}. |
| docker | Not needed in {{site.data.keyword.codeengineshort}}. |
| health-check-http-endpoint | Not needed in {{site.data.keyword.codeengineshort}}. By default a TCP probe is used to know when the application is healthy and ready. |
| health-check-invocation-timeout | Not needed in {{site.data.keyword.codeengineshort}}. |
| instances | `--min-scale` and `--max-scale` options |
| memory | `--memory` option |
| metadata | Not supported at this time. |
| no-route | With the `--cluster-local` option, the application is still accessible from other workloads within the project, but there is not an internet-facing accessible URL associated with the application. |
| path | Not applicable at this time. |
| processes | Not needed in {{site.data.keyword.codeengineshort}}. The application can create additional processes at runtime. |
| random-route | Not needed in {{site.data.keyword.codeengineshort}}. Each project has a unique subdomain and since the application name is part of the URL, the URL is guaranteed to be unique. |
| routes | Custom routes are not supported at this time, but you can use IBM Cloud Internet Service (CIS) to front-end your application with a custom domain. |
| sidecars | Not supported at this time. |
| stack | Implicitly managed by {{site.data.keyword.codeengineshort}}. |
| timeout | Not needed in {{site.data.keyword.codeengineshort}}. |
| Environment variables | `-env` option |
| Services | See the [**`ibmcloud ce app bind`**](/docs/codeengine?topic=codeengine-cli#cli-application-bind) command. |
{: caption="Table 1. Markdown coding for tables" caption-side="bottom"}

{{site.data.keyword.codeengineshort}} supports many options that are not available in Cloud Foundry, such as how autoscaling is managed. See [Working with apps in {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-application-workloads) and [Configuring application scaling](/docs/codeengine?topic=codeengine-app-scale).

## I know how to deploy an app with Cloud Foundry. What do I need to know to deploy an app in {{site.data.keyword.codeengineshort}}?
{: #deployment}

If you know how to deploy an app with Cloud Foundry, find what you need to know to deploy an app in {{site.data.keyword.codeengineshort}}.

### Push code
{: #push}

With {{site.data.keyword.codeengineshort}}, you can build your code that is sourced in a Git repository or from a local system (CLI-only). Additionally, with Cloud Foundry, you can build and deploy your app in a single step (`cf push`). While you can deploy an app or job in a single step from the {{site.data.keyword.codeengineshort}} console, when you use the CLI, the process requires several steps. First, you must define and run your build to create a container image in a container registry. Then, you can deploy that image as an application or job. 

### Deployment context
{: #deploy-context}

Cloud Foundry requires an `Org` and a `Space` to push your code into an application. All Cloud Foundry users, by default receive an `Org` and `Space` that are created for them. However, to add new ones, you must set up an `Org` and `Space` target similar to the following example.

```txt
ibmcloud cf create-org MyOrg
ibmcloud target -o <ORGNAME>
ibmcloud target -s dev
```
{: codeblock}

When you then deploy an app with Cloud Foundry, that `Org` and `Space` is targeted for the deployment.

{{site.data.keyword.codeengineshort}} uses the concept of an IBM Cloud resource group and a {{site.data.keyword.codeengineshort}} project.  

```txt
ibmcloud target -g <RESOURCE-GROUP>
ibmcloud ce project create --name <PROJECTNAME>
```
{: codeblock}

These commands not only create a project, but "targets" it as well. All subsequent {{site.data.keyword.codeengineshort}} commands run in the context of this project until a different project is targeted by using the **`project select`** command. For more information, see [Managing projects](/docs/codeengine?topic=codeengine-manage-project).

### Logs
{: #logs}

{{site.data.keyword.codeengineshort}} provides logs for apps, jobs, and builds to help you determine what happened when deployments do not run properly. You can find logs by running commands similar to the following examples.

```
ibmcloud ce app logs -n <APPNAME>
ibmcloud ce jobrun logs -n <JOBRUN-NAME>
ibmcloud ce buildrun logs -n <BUILDRUN_NAME>
```
{: codeblock}

You can also use the {{site.data.keyword.la_full_notm}} service, which is available for more long-term persistence of log messages. For more information, see [Viewing logs](/docs/codeengine?topic=codeengine-view-logs).

### Creating a service
{: #create-service}

Creating a new instance of a managed service is similar in Cloud Foundry and {{site.data.keyword.codeengineshort}}.

To create a new service to use with your Cloud Foundry application, use the following command.

```
ibmcloud cf create-service cloudantNoSQLDB lite myNameCloudant
```
{: codeblock}

To create a new service to use with {{site.data.keyword.codeengineshort}} applications, 

```
ibmcloud resource service-instance-create myNameCOS cloud-object-storage lite global
```
{: codeblock}

### Service binding
{: #service-binding}

After you create your application, you can then "bind" your application to the service. 

With Cloud Foundry, run the following command.

```
ibmcloud cf bind-service appName instanceName
```
{: codeblock}

With {{site.data.keyword.codeengineshort}},

```
ibmcloud ce app bind --name appName --service-instance instanceName
```
{: codeblock}

The service instance credentials (coordinates) are injected into the app (or job) by using environment variables. The Cloud Foundry equivalent of `VCAP_SERVICES` in {{site.data.keyword.codeengineshort}} is `CE_SERVICES`. For more information, see [Integrating IBM Cloud service with service binding](/docs/codeengine?topic=codeengine-service-binding).

### Updating an app or job
{: #update-app-job}

After you create your application or job, you can update properties of your workload by using the update command. For example, to update an application in {{site.data.keyword.codeengineshort}}, 

```
ibmcloud ce app update --name <APPNAME> ...
```
{: codeblock}

You can update any of the properties that are available when you create an application or job. For more information, see the following topics.

- [Working with applications](/docs/codeengine?topic=codeengine-application-workloads)
- [Updating an app](/docs/codeengine?topic=codeengine-update-app)
- [Working with jobs and job runs](/docs/codeengine?topic=codeengine-job-plan)
- [Updating a job](/docs/codeengine?topic=codeengine-update-job)
- [**`app update`**](/docs/codeengine?topic=codeengine-cli#cli-application-update) command
- [**`job update`**](/docs/codeengine?topic=codeengine-cli#cli-job-update) command


### Runtime support
{: #runtime}

{{site.data.keyword.codeengineshort}} supports many of the runtimes Cloud Foundry supports. For a list of supported runtimes, see [Cloud Native Buildpacks](/docs/codeengine?topic=codeengine-plan-build#build-buildpack-strat). If you want use a runtime that is not supported, for example, Swift or Liberty, you can package your app as a container image yourself and [deploy that image in {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-deploy-app) without building the image directly from {{site.data.keyword.codeengineshort}}.

## I know Cloud Foundry commands. What are the {{site.data.keyword.codeengineshort}} equivalents?
{: #operations}

Find and compare the {{site.data.keyword.codeengineshort}} CLI command that is equivalent to your Cloud Foundry commands.

### Connect to the deployment API region commands
{: #deployment-api}

Cloud Foundry
:    `ibmcloud target --cf-api api.us-south.cf.cloud.ibm.com`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud target -r [region]`

### Create a deployment location for your workload commands
{: #deployment-place}

Cloud Foundry
:    `ibmcloud cf create-org [orgName]`
:    `ibmcloud target -o [orgName]`
:    `ibmcloud target -s [spaceName]`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce project create --name [projectName]`

You can also create a project from the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

### List your apps and jobs commands
{: #list-workload}

Cloud Foundry
:    `ibmcloud cf apps`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce app list`
:    `ibmcloud ce job list`

You can also view your applications and jobs from the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

### Deploy a image as an app or job commands
{: #deploy-commands}

Cloud Foundry
:    `ibmcloud cf push`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce build create` Define a build
:    `ibmcloud ce build submit`  Run a build
:    `ibmcloud ce app create` Deploy image as an app
:    `ibmcloud ce job create` Create a batch job

You can perform these tasks from a single web page in the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

### Bind a Service to a workload
{: #binding-commands}

Cloud Foundry
:    `ibmcloud cf bind-service [AppName] [ServiceInstance]https://cli.cloudfoundry.org/en-US/v6/bind-service.html`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce app bind --name [AppName] --service-instance [ServinceInstance]`
:    `ibmcloud ce job bind --name [JobName] --service-instance [ServinceInstance]`


### Update applications or jobs commands
{: #update-commands}

Cloud Foundry
:    `ibmcloud cf scale ...`
:    `ibmcloud cf set-env ...`
:    `ibmcloud cf push ...`
:    `ibmcloud cf delete ...`

For example, to scale your application to 2 instances, 1 G of memory, and restart it, run `cf scale APP_NAME -i 2 -m 1G -f`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce app update ...`
:    `ibmcloud ce job update ...`

You can also update your applications and jobs from the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

For information about what you can update with {{site.data.keyword.codeengineshort}}, see the following topics.

- [Working with applications](/docs/codeengine?topic=codeengine-application-workloads)
- [Updating an app](/docs/codeengine?topic=codeengine-update-app)
- [Working with jobs and job runs](/docs/codeengine?topic=codeengine-job-plan)
- [Updating a job](/docs/codeengine?topic=codeengine-update-job)
- [**`app update`**](/docs/codeengine?topic=codeengine-cli#cli-application-update) command
- [**`job update`**](/docs/codeengine?topic=codeengine-cli#cli-job-update) command


### Control applications and jobs commands
{: #control-commands}

Cloud Foundry
:    `ibmcloud cf start AppName` to start your application
:    `ibmcloud cf stop AppName` to stop your application

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce app create ...`
:    `ibmcloud ce app delete ...`
:    `ibmcloud ce job create ...`
:    `ibmcloud ce job delete ...`

You can also control your applications and jobs from the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

### Get information about apps commands
{: #get-info-commands}

Cloud Foundry
:    `ibmcloud cf app AppName`

{{site.data.keyword.codeengineshort}}
:    `ibmcloud ce app get -n <APPNAME>`
:    `ibmcloud ce job get -n <JOBNAME>`

You can also view information about your applications and jobs from the [{{site.data.keyword.codeengineshort}} console](https://cloud.ibm.com/codeengine/overview){: external}.

### Specifying a buildpack version
{: #specify-buildpack}

Cloud Foundry
:    `ibmcloud cf push -b https://github.com/cloudfoundry/nodejs-buildpack.git#v1.6.56`

{{site.data.keyword.codeengineshort}}
:    Not supported

## More information
{: #more-info}

- [{{site.data.keyword.codeengineshort}} Documentation](/docs/codeengine?topic=codeengine-getting-started)
- [{{site.data.keyword.codeengineshort}} Thumbnail Tutorial](https://github.com/IBM/CodeEngine/tree/main/thumbnail){: external}
- [Using Cloud Native buildpacks with {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-build-app-tutorial) tutorial
- [Running Batch Jobs in {{site.data.keyword.codeengineshort}}](/docs/codeengine?topic=codeengine-run-job-tutorial)
- [Blog: Migrating from Cloud Foundry to {{site.data.keyword.codeengineshort}}](https://www.ibm.com/cloud/blog/migrating-from-cloud-foundry-to-code-engine){: external}
- [Video: IBM {{site.data.keyword.codeengineshort}} 201](https://www.youtube.com/watch?v=zKLf11tZOY4){: external}
- [IBM Developer: Code Engine](https://developer.ibm.com/?q=%22code%20engine%22){: external}