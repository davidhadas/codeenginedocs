---

copyright:
  years: 2022
lastupdated: "2022-08-02"

keywords: security, zero trust, runtime security, workload security, workload security, serverless security, Gaurd, code engine application security, code engine service

subcollection: codeengine

content-type: tutorial
completion-time: 5m 

---

{{site.data.keyword.attribute-definition-list}}

# Securing your workload with Guard
{: #runtimesecuritytut}
{: toc-content-type="tutorial"}
{: toc-completion-time="5m"}

[Guard](https://github.com/IBM/workload-security-guard/){: external} is an open source, workload runtime security solution, well equipped to protect serverless services.
{: shortdesc}

In this tutorial we will protect a sample service that we will deploy on code engine. The steps to protect your own services are similar.

Before you begin:

- [Set up your {{site.data.keyword.codeengineshort}} CLI environment](/docs/codeengine?topic=codeengine-install-cli).
- [Create and work with a project](/docs/codeengine?topic=codeengine-manage-project).

All {{site.data.keyword.codeengineshort}} users are required to have a Pay-as-you-Go account. Tutorials might incur costs. Use the Cost Estimator to generate a cost estimate based on your projected usage. For more information, see [{{site.data.keyword.codeengineshort}} pricing](/docs/codeengine?topic=codeengine-pricing).
{: note}

Steps


1. Deploy ***guard-learner*** (this needs to be repeated on every project where guard is used)
2. Deploy one or more applications

## Deploy guard-learner
{: #guard-learner}
{: step}

Guard-learner is service that auto learns guard security micro-rules. It is helpful to streamline the security of code engine serverless services.  

- Create a guard-learner code-engine service

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
    [https://guard-learner.p8rrxs4rezl.us-south.codeengine.appdomain.cloud](http://guard-learner.p8rrxs4rezl.svc.cluster.local)
    ```
    {: screen}
    
Notes:
* Ensure to protect your service and not expose it to potential offenders by using project level visibility using `-v project`
* Scale the guard-learner service to continuesly run a single instance using  `--min 1 --max 1`
* You need to create the guard-learner service only once per code-engine project.

## Creating and deploying an application
{: #guard-deploy-app}
{: step}

- Create your application and give it a project visibility. It in this tutorial you will use the helloworld image as shown in [**`ibmcloud ce application create`**](/docs/codeengine?topic=codeengine-cli#cli-application-create) command. 
- In the following example, you may change the service name and set a different image and port.

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
* Ensure to protect your service and not expose it to potential offenders by using project level visibility using `-v project`
* It is advised (but not mandatory) to set the minimum scale to a single instance using  `--min 1`

## Finding the Namespace and service URLs 
{: #guard-get-parameters}
{: step}

- Extract information about the service and guard-leaner

```txt
export NAMESPACE=`ibmcloud ce project current -o json|jq -r .kube_config_context`
export GUARD_URL=`ibmcloud ce application get -n guard-learner -o url`
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

## Expose your service to the internet via guard
{: #guard-expose-app}
{: step}

- Add a guard service to securly expose your application service 

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
    
Notes:
* Ensure to provide guard with the protect namespce and the coresponsing service it protects. Each guard service protects a coresponding application service.
* Ensure to set `USE_CONFIGMAP=true` guard will maintain a config map names `guardian-${SERVICE_NAME}` with the service micro-rules
* It is advised (but not mandatory) to set the minimum scale to a single instance using  `--min 1`

## Removing this tutorial services
{: #guard-cleanup}
{: step}

You can clean up your local system by removing the `Iter8-in-Docker` container and image if you no longer need it.

```txt
ibmcloud ce application delete -n ${SERVICE_NAME}
ibmcloud ce application delete -n ${SERVICE_NAME}-guard
ibmcloud ce application delete -n guard-learner
```
{: pre}

## Security Situational Awareness
{: #guard-situational-awareness}
{: step}

Guard offers situational awarenss into the service security posture.
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
    
Notes:
* Security Alerts appear as warnings in the log file and start with teh string "SECURITY ALERT!"
* The default setup of Guard is to never block any request and to auto-learn the requests within ~5 minuetes. Consequently, it takes about 30 min for guard to learn the charactaristics of the incoming requests and build coresponding micro-rulkes. 
* After the initial learning period, Guard wil alert only when a change in behaviour is detedted. 
* Note that in the default mode of operation, Guard continues to learn any new behaviour, correct security procedures should incldue reviewing any new behaviour detected by Guard. 
* Guard has additional modes of operations that enable setting manual micro-rules and/or blocking requetss based on micro-rules - contect IBM Code Engine for further details

