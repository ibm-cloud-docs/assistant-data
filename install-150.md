---

copyright:
  years: 2015, 2020
lastupdated: "2020-12-14"

subcollection: assistant-data

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:deprecated: .deprecated}
{:important: .important}
{:note: .note}
{:tip: .tip}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}
{:download: .download}
{:gif: data-image-type='gif'}

# Installing 1.5.0
{: #install-150}

For installation instructions, find the instructions for the appropriate version of {{site.data.keyword.icp4dfull_notm}}:

- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.5](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-install.html){: external} 
- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.0.1](#install-150-on-301)

## Installing on IBM Cloud Pak for Data 3.0.1
{: #install-150-on-301}

The latest release of {{site.data.keyword.conversationshort}} uses operators. You must use the cpd-cli command line that is available only with {{site.data.keyword.icp4dfull_notm}} 3.5 to install it.

To install {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} on {{site.data.keyword.icp4dfull_notm}} 3.0.1, complete the following steps:

1.  Install {{site.data.keyword.icp4dfull_notm}} 3.0.1 by following the instruction in the [{{site.data.keyword.icp4dfull_notm}} 3.0.1 knowledge center](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/install.html){: external}.

1.  Get the {{site.data.keyword.icp4dfull_notm}} 3.5 version of the command line interface by following the instructions in the [{{site.data.keyword.icp4dfull_notm}} 3.5 knowledge center](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/installation-files.html){: external}. Complete steps 1 through 2a only of the procedure for obtaining the installation files.

1.  Run the following command to instruct {{site.data.keyword.icp4dfull_notm}} to use the new (3.5) command line interface version.

    ```
    ./cpd-cli operator-upgrade -n $PROJECT
    ```
    {: codeblock}

    where $PROJECT is the name of the project or namespace where you want to install the service.

1.  Follow [the instructions](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-install.html){: external} for installing the {{site.data.keyword.conversationshort}} service on {{site.data.keyword.icp4dfull_notm}} 3.5.

## Testing the installation
{: #install-150-test}

After installing the service, run a verification test to make sure that things are working properly.

1.  Create a yaml file with a name such as `test-override.yaml` that looks as follows.

    ```yaml
    apiVersion: com.ibm.watson.watson-assistant/v1
    kind: WatsonAssistantDvt
    metadata:
      name: watson-assistant---${instance-name}
      annotations:
        oppy.ibm.com/disable-rollback: "true"
    spec:
      # Additional labels to pass down to all objects created by the operator
      labels:
        "app.kubernetes.io/instance": "watson-assistant---${instance-name}"
      version: 1.5.0
      # The name of the WA instance to target with this DVT run
      assistantInstanceName: watson-assistant---${instance-name}
      # The cucumber test tags to execute
      testTags: "@accuracy,@dialogErrors,@dialogs,@dialogV1,@dialogV1errors,@embedsearch,@entities,@folders,@fuzzy,@generic,@healthcheck,@intents,@newse,@openentities,@patterns,@prebuilt,@search,@slots,@spellcheck,@spellcheckfr,@v2assistants,@v2authorskill,@v2authorwksp,@v2healthcheck,@v2skillref,@v2snapshots,@workspaces"
      # Information specific to this cluster
      cluster:
        # :type: Cluster environment type
        # - options: 'public', 'dedicated', 'premium', 'private'
        type: "private"
        # :image_pull_secrets: pull secret names
        imagePullSecrets: []
        # TODO: These are for WA dev
        # :docker_registry_prefix: (private only) Docker registry, including namespace, to get images from
        dockerRegistryPrefix: "${registry info}"
    ```
    {: codeblock}

1.  Replace the following values in the YAML file:

    - `${instance name}`: Instance name, such as `wa001`.
    - `${registry info}`: Docker registry, which is typically different depending on the version of OpenShift you're using.

      - **4.5**: `"image-registry.openshift-image-registry.svc:5000/${namespace}"`
      - **3.11**: `"docker-registry.default.svc:5000/${namespace}"`

      where you replace `${namespace}` with the namespace, such as `zen`.

    - **3.11 only**: Change `@accuracy` to `@accuracyOS311`.

1.  Run the following command to apply the configuration changes that you specified in the yaml file. Your changes initiate a job named `dvt` which starts a pod named `dvt` to run the test.

    ```
    oc apply -f ${yaml filename}
    ```
    {: pre}

    Give the changes time to take effect.

1.  Check to see if the `dvt` job named is now available. You can run the following command to list the jobs.

    ```
    oc get job |grep dvt
    ```
    {: pre}

    When the job is available, a response like this is displayed: 
    
    ```
    watson-assistant---wa001-dvt-job    1/1   66m   5h46m
    ```
    {: codeblock}

    When the job runs it starts a pod to use for processing the test. 

1.  Check to see if the `dvt` pod is now available. You can run the following command to list the pods.

    ```
    oc get pods |grep dvt
    ```
    {: pre}

    When the pod is available, a response like this is displayed: 
    
    ```
    watson-assistant---wa001-dvt-job-gq68t   0/1   Completed   0   5h47m
    ```
    {: codeblock}

1.  To view logs from the pod while the job is running, use the following command:

    ```
    oc logs watson-assistant---wa001-dvt-job-gq68t -f
    ```
    {: pre}

1.  After the job is completed, you can check the status of the test by using the following command:

    ```
    oc logs watson-assistant---wa001-dvt-job-gq68t
    ```
    {: pre}

    When the test is successful, a message like this is displayed:

    ```
    OK. test_output/bdd/cucumber/regression_failures.txt is empty. BDD tests passed.
    ```
    {: codeblock}

1.  To rerun the test, delete the `dvt` job. A DVT job and pod will be created and started again in few minutes.

1.  To check whether the test yaml has been applied to your cluster already, use the following command:

    ```
    oc get dvt
    ```
    {: pre}

1.  To edit the configuration of the test yaml, use the following command:

    ```
    oc edit dvt watson-assistant---wa001
    ```
    {: pre}

## Troubleshooting issues
{: #install-150-ts-get-logs}

The first step to take if you hit an installation issue, such as a cluster node is not starting as expected, is to get logs from the cluster which can provide more detail.

To get log files, complete the following steps:

1.  Log into the cluster with administrator credentials.

1.  Run the following command to get a list of the jobs that are currently running in the cluster and whether the job was successful:

    ```bash
    oc get jobs
    ```
    {: pre}

1.  For any jobs that show a success status of 0, get the log file for the job by entering the following command:

    ```bash
    oc logs -f job/{job-name}
    ```
    {: pre}

1.  You can follow the logs of running pods also.

    Use this command to get the pod names:

    ```bash
    oc get pods
    ```
    {: codeblock}

1.  To follow the log for a particular pod:

    ```bash
    oc logs -f {pod-name}
    ```
    {: codeblock}

1.  If you want more detail about a pod because the pod has not started yet or is failing to start at all, for example, you can use the following command to get more details.

    ```bash
    oc describe pod {pod-name}
    ```
    {: codeblock}

## Next steps
{: #install-150-next-steps}

Use the {{site.data.keyword.conversationshort}} product user interface to build training data and a dialog that can be used by your assistant.

- To learn more about the service first, read the [overview](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-index).
- To see how it works for yourself, follow the steps in the [getting started tutorial](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-getting-started).
- For help managing the cluster, see [Managing the cluster](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-manage).
