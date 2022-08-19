---

copyright:
  years: 2022
lastupdated: "2022-08-02"

keywords: security, zero trust, runtime security, workload security, situational awareness, serverless security, Guard, code engine application security, code engine security

subcollection: codeengine

content-type: tutorial
completion-time: 5m 

---

{{site.data.keyword.attribute-definition-list}}

# Securing your application with Guard
{: #runtimesecuritytut}
{: toc-content-type="tutorial"}
{: toc-completion-time="5m"}

[Guard](http://knative.dev/security-guard){: external} is a runtime-security solution that will be offered as an option in later versions of Knative. Guard uses a per-application auto-learned set of micro rules to control the application ingress. As a result, Guard  helps identifiy application anomalities and supports Situational Awareness. Guard can also be used to block anomalous ingress. 
{: shortdesc}

In this tutorial we deploy a new helloworld application and protect it with Guard. The steps to protect your own applications are similar.

Before you begin:

- Make sure that your are able to [Deploy applications using CLI](docs/codeengine?topic=codeengine-deploy-app-tutorial).

All {{site.data.keyword.codeengineshort}} users are required to have a Pay-as-you-Go account. Tutorials might incur costs. Use the Cost Estimator to generate a cost estimate based on your projected usage. For more information, see [{{site.data.keyword.codeengineshort}} pricing](/docs/codeengine?topic=codeengine-pricing).
{: note}

Steps


1. Deploying a Guard-Learner application as a per-project Service
2. Creating and deploying a protected unexposed helloeworld application
3. Finding the namespace and service URLs 
4. Exposing the helloeworld application to the internet via Guard
5. Managing the security of the helloeworld application and gaining Situational Awareness
6. Removing this tutorial applications

## Deploy a guard-learner application as a per-project service
{: #guard-learner}
{: step}

Guard-Learner is service that auto-learns the necessery micro-rules for each Guard protected application in a project. It is helpful to streamline the deployment of new applications that seek Guard protection.   

- Deploying a Guard-Learner application

    ```txt
    ibmcloud ce application create -n guard-learner -v project --min 1 --max 1 -p 8888 -i ghcr.io/ibm/workload-security-guard/guard-learner
    ```
    {: pre}

    Example output

    ```txt
    Creating application 'guard-learner'...
    [...]
    Run 'ibmcloud ce application get -n guard-learner' to check the application status.
    OK

    http://guard-learner.p8rrxs4rezl.svc.cluster.local
    ```
    {: screen}
    
Notes:
* Ensure to protect your application and avoid exposing it to potential offenders by setting project level visibility using `-v project`
* Scale the Guard-Learner service to continuesly run a single instance using  `--min 1 --max 1`
* You need to create the Guard-Learner service only once per code-engine project.

## Creating and deploying a protected unexposed helloeworld application
{: #guard-deploy-app}
{: step}

- Create the helloworld application and give it a project visibility. 
- In the following example, you may change the application name and set a different image and port.

    ```txt
    export SERVICE_NAME="myapp"
    ibmcloud ce application create -n ${SERVICE_NAME} -v project -min 1 -p 8080 -i icr.io/codeengine/hello

    ```
    {: pre}

    Example output

    ```txt
    Creating application 'myapp'...
    [...]
    Run 'ibmcloud ce application get -n myapp' to check the application status.
    OK
    
    http://myapp.p8rrxs4rezl.svc.cluster.local
    ```
    {: screen}
    
Notes:
* Ensure to protect your service and not expose it to potential offenders by setting project level visibility using `-v project`
* It is advised (but not mandatory) to set the minimum scale to a single instance using  `--min 1`

## Finding the namespace and service URLs 
{: #guard-get-parameters}
{: step}

- Extract information about the service and guard-leaner

```txt
export NAMESPACE=`ibmcloud ce project current -o json|jq -r .kube_config_context`
export LEARNER_URL=`ibmcloud ce application get -n guard-learner -o url`
export SERVICE_URL=`ibmcloud ce application get -n ${SERVICE_NAME} -o url`

echo "The protected service name '${SERVICE_NAME}' namespace '${NAMESPACE}' url '${SERVICE_URL}'"
echo "The guard learner url is '${GUARD_URL}'"
```
{: pre}

Example output

```txt
The protected service name 'myapp' namespace 'p8rrxs4rezl' url 'http://myapp.p8rrxs4rezl.svc.cluster.local'
The guard learner url is 'http://guard-learner.p8rrxs4rezl.svc.cluster.local'
```
{: screen}

## Exposing the helloeworld application to the internet via guard
{: #guard-expose-app}
{: step}

- Add a helloeworld Guard to securly expose the helloeworld application to the internet 

```txt
ibmcloud ce application create -n ${SERVICE_NAME}-guard --min 1 -p 22000 \
        -e GUARD_URL=${GUARD_URL} \
        -e SERVICE_URL=${SERVICE_URL} \
        -e SERVICE_NAME=${SERVICE_NAME} \
        -e NAMESPACE=${NAMESPACE} \
        -e USE_CONFIGMAP=true \
        -i ghcr.io/ibm/workload-security-guard/guard-rproxy
```
{: pre}

Example output

    ```txt
    Creating application 'myapp-guard'...
    [...]
    Run 'ibmcloud ce application get -n myapp-guard' to check the application status.
    OK
    
    https://myapp-guard.p8rrxs4rezl.us-south.codeengine.appdomain.cloud
    ```
    {: screen}

You may now access the protected helloworld application via the url exposed by helloeworld Guard 


Notes:
* Deploy a Gurd for any application you seek to protect.
* Ensure to provide Guard with the project namespce and the coresponsing application name and url it protects. 
* Ensure to provide Guard with the url of the Guard-Learner. 
* Ensure to set `USE_CONFIGMAP=true` to instruct Guard to maintain the micro-rules in a configmap named `guardian-${SERVICE_NAME}` 
* It is advised (but not mandatory) to set the minimum scale of Guard to a single instance using  `--min 1`


## Managing the security of the helloeworld application and gaining Situational Awareness
{: #guard-situational-awareness}
{: step}

Guard offers situational awarenss into the application security posture.
You may see Security Alerts in the log file of guard.

```txt
ibmcloud ce application logs -n ${SERVICE_NAME}-guard 
```
{: pre}

Example output

    ```txt
    [...]
    {"level":"warn","message":"SECURITY ALERT! HttpRequest: Headers: KeyVal: Known Key X-B3-Traceid: Digits: Counter out of Range: 25"}  
    [...]
    ```
    {: screen}


Security Alerts appear as warnings in the log file and start with the string "SECURITY ALERT!". The default setup of Guard is to never block any request and auto-learn any changes in the ingress. It typically takes about 30 min for Guard to learn the charactaristics of the incoming requests and build coresponding micro-rules. After the initial learning period, Guard alerts only when a change in behaviour is detedted. 

Note that in the default setup, Guard continues to learn any new behavior and will therefore avoid reporting when the new behavior repeats.
Correct security procedures should incldue reviewing any new behavior detected by Guard. 

Guard can also be configrued to operate in other important modes of operation such as:

- Move from auto-learning to manual micro-rules management after the initial learning period
- Block requests/responses when they do not conform to the micro-rules 

Contact us in the [#code-engine channel](https://ibm-cloud-success.slack.com){: external} for further details. 

## Removing this tutorial applications
{: #guard-cleanup}
{: step}

You can clean up your local system by removing the helloeworld application, the helloeworld Guard and the Guard-Learner if you no longer need it.

```txt
ibmcloud ce application delete -n ${SERVICE_NAME}
ibmcloud ce application delete -n ${SERVICE_NAME}-guard
ibmcloud ce application delete -n guard-learner
```
{: pre}


