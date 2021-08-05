---

copyright:
  years: 2020, 2021
lastupdated: "2021-07-22"

keywords: applications in code engine, apps in code engine, http requests in code engine, deploy apps in code engine, app workloads in code engine, deploying workloads in code engine, application, app, memory, cpu, environment variables

subcollection: codeengine

---

{:DomainName: data-hd-keyref="APPDomain"}
{:DomainName: data-hd-keyref="DomainName"}
{:android: data-hd-operatingsystem="android"}
{:api: .ph data-hd-interface='api'}
{:apikey: data-credential-placeholder='apikey'}
{:app_key: data-hd-keyref="app_key"}
{:app_name: data-hd-keyref="app_name"}
{:app_secret: data-hd-keyref="app_secret"}
{:app_url: data-hd-keyref="app_url"}
{:authenticated-content: .authenticated-content}
{:beta: .beta}
{:c#: data-hd-programlang="c#"}
{:cli: .ph data-hd-interface='cli'}
{:codeblock: .codeblock}
{:curl: .ph data-hd-programlang='curl'}
{:deprecated: .deprecated}
{:dotnet-standard: .ph data-hd-programlang='dotnet-standard'}
{:download: .download}
{:external: target="_blank" .external}
{:faq: data-hd-content-type='faq'}
{:fuzzybunny: .ph data-hd-programlang='fuzzybunny'}
{:generic: data-hd-operatingsystem="generic"}
{:generic: data-hd-programlang="generic"}
{:gif: data-image-type='gif'}
{:go: .ph data-hd-programlang='go'}
{:help: data-hd-content-type='help'}
{:hide-dashboard: .hide-dashboard}
{:hide-in-docs: .hide-in-docs}
{:important: .important}
{:ios: data-hd-operatingsystem="ios"}
{:java: .ph data-hd-programlang='java'}
{:java: data-hd-programlang="java"}
{:javascript: .ph data-hd-programlang='javascript'}
{:javascript: data-hd-programlang="javascript"}
{:new_window: target="_blank"}
{:note .note}
{:note: .note}
{:objectc data-hd-programlang="objectc"}
{:org_name: data-hd-keyref="org_name"}
{:php: data-hd-programlang="php"}
{:pre: .pre}
{:preview: .preview}
{:python: .ph data-hd-programlang='python'}
{:python: data-hd-programlang="python"}
{:route: data-hd-keyref="route"}
{:row-headers: .row-headers}
{:ruby: .ph data-hd-programlang='ruby'}
{:ruby: data-hd-programlang="ruby"}
{:runtime: architecture="runtime"}
{:runtimeIcon: .runtimeIcon}
{:runtimeIconList: .runtimeIconList}
{:runtimeLink: .runtimeLink}
{:runtimeTitle: .runtimeTitle}
{:screen: .screen}
{:script: data-hd-video='script'}
{:service: architecture="service"}
{:service_instance_name: data-hd-keyref="service_instance_name"}
{:service_name: data-hd-keyref="service_name"}
{:shortdesc: .shortdesc}
{:space_name: data-hd-keyref="space_name"}
{:step: data-tutorial-type='step'}
{:subsection: outputclass="subsection"}
{:support: data-reuse='support'}
{:swift: .ph data-hd-programlang='swift'}
{:swift: data-hd-programlang="swift"}
{:table: .aria-labeledby="caption"}
{:term: .term}
{:terraform: .ph data-hd-interface='terraform'}
{:tip: .tip}
{:tooling-url: data-tooling-url-placeholder='tooling-url'}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tsSymptoms: .tsSymptoms}
{:tutorial: data-hd-content-type='tutorial'}
{:ui: .ph data-hd-interface='ui'}
{:unity: .ph data-hd-programlang='unity'}
{:url: data-credential-placeholder='url'}
{:user_ID: data-hd-keyref="user_ID"}
{:vbnet: .ph data-hd-programlang='vb.net'}
{:video: .video}


# Automatically injected environment variables
{: #inside-env-vars}
When you deploy an application or run a job, {{site.data.keyword.codeengineshort}} automatically injects certain environment variables into the app or the job.


## Automatically injected environment variables for apps
{: #inside-env-vars-app}
	
When you deploy an application, {{site.data.keyword.codeengineshort}} automatically injects certain environment variables into the app. The following table lists automatically injected environment variables into each instance of your deployed app. The following examples of automatically injected environment variables are based on an app that is named `myapp`, which references the {{site.data.keyword.codeengineshort}} sample image, `ibmcom/codeengine`.

The first 3 environment variables, `CE_APP`, `CE_DOMAIN`, and `CE_SUBDOMAIN` are used to construct the URL of an application, `https://CE_APP.CE_SUBDOMAIN.CE_DOMAIN`. For example, if `CE_APP=myapp`, `CE_SUBDOMAIN=01234567-abcd` and `CE_DOMAIN=us-south.codeengine.dev.appdomain.cloud`, your application external URL is `https://myapp.01234567-abcd.us-south.codeengine.dev.appdomain.cloud`. The private URL of your application is `appName.CE_SUBDOMAIN`, or `myapp.01234567-abcd`.

| Environment variable | Description | Example |
|--------------------|-------------------------------------------------------|--------------|
| `CE_APP`           | The name of the application.                                          | `CE_APP=myapp` |
| `CE_DOMAIN`        | The domain name portion of the URL of the application (and project).  | `CE_DOMAIN=us-south.codeengine.dev.appdomain.cloud` |
| `CE_SUBDOMAIN`     | The subdomain portion of the URL associated with the application (and project). If you are familiar with Kubernetes, CE_SUBDOMAIN maps to the Kubernetes namespace associated with your project. | `CE_SUBDOMAIN=01234567-abcd` |
| `HOME`             | Your home directory that is running the app.                          | `HOME=/root` |
| `HOSTNAME`         | The name of instance that your app is deployed to.                    | `HOSTNAME=myapp-00001-deployment-65684cffd-xng7z` | 
| `K_CONFIGURATION`  | The name of your app.                                                 | `K_CONFIGURATION=myapp` |
| `K_REVISION`       | The name of the current revision of your deployed app.                | `K_REVISION=myapp-00001` |
| `K_SERVICE`        | The name of your app.                                                 | `K_SERVICE=myapp` |
| `PATH`             | The list of directories in which the system looks for executables.    | `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` |
| `PORT`             | The port that your app listens to receive HTTP requests.              | `PORT=8080` |
| `PWD`              | The current working directory.                                        | `PWD=/` |
{: caption="Automatically injected environment variables when deploying {{site.data.keyword.codeengineshort}} apps"}

Note that you can override the `PORT` variable by deploying your app from the console and specifying the **Listening port** value or by using the CLI and setting the `--port` option.

## Automatically injected environment variables for jobs
{: #inside-env-vars-jobs}
	
When you run a job, {{site.data.keyword.codeengineshort}} automatically injects certain environment variables into the job run instance. The following table lists automatically injected environment variables into each instance of your running job. The following examples of automatically injected environment variables are based on a job that is named `myjob`, which references the {{site.data.keyword.codeengineshort}} sample image, `ibmcom/codeengine`.

| Environment variable | Description | Example |
|----------------|---------------------|---------|
| `CE_DOMAIN`    | The domain name of the project.                                           | `CE_DOMAIN=us-south.codeengine.appdomain.cloud` |
| `CE_JOB`       | The name of the defined job configuration that was used for the job run.  | `CE_JOB=myjobdef` |
| `CE_JOBRUN`    | The name of the job run.                                                  | `CE_JOBRUN=myjob-jobrun-f5kxz` |
| `CE_SUBDOMAIN` | The subdomain associated with the project in which your job is running. If you are familiar with Kubernetes, `CE_SUBDOMAIN` maps to the Kubernetes namespace that is associated with your project.                       | `CE_SUBDOMAIN=01234567-abcd` |
| `HOME`         | Your home directory that is running the job.                              | `HOME=/root` |
| `HOSTNAME`     | The name of instance that your app is deployed to.                        | `HOSTNAME=myjob-jobrun-6bgmg-0-0` |
| `JOB_INDEX`    | The index of a specific job run instance.                                 | `JOB_INDEX=1` |
| `PATH`         | The list of directories in which the system looks for executables.        | `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` |
| `PWD`          | The current working directory.                                            | `PWD=/` |
{: caption="Automatically injected environment variables when deploying {{site.data.keyword.codeengineshort}} jobs"}

Note that each job run instance gets its own index from the array of indices that were specified when the job was created. The `JOB_INDEX` environment variable contains the index value.

While the job itself doesn't have a URL associated with it, the `CE_DOMAIN` and `CE_SUBDOMAIN` values might be useful if you need to reference an applications that is running in the same project. The full external URL of this application is `appName.CE_SUBDOMAIN.CE_DOMAIN`. To reference the private URL of an application, use `appName.CE_SUBDOMAIN`.