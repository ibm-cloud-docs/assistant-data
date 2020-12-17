---

copyright:
  years: 2015, 2020
lastupdated: "2020-12-16"

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

- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.5](#install-150-on-301)
- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.0.1](#install-150-on-301)

## System requirements
: #install-150-reqs}

| Resource | Large  | Medium |  Small |
|----------|--------|--------|--------|
| vCPU     |     35 |     25 |     15 |
| Memory   | 250 GB | 180 GB | 100 GB |
| Storage  | 284 Gi | 284 Gi | 284 Gi |
{: caption="System requirements" caption-side="top"}

The following table shows details for a typical deployment.

| Nodes | VPCs | Memory per node |
|-------|------|-----------------|
|     4 |   16 |          128 GB |
{: caption="Deployment details" caption-side="top"}

## Installing on IBM Cloud Pak for Data 3.5
{: #install-150-on-35}

The overall steps are specified here as a preview. For the complete procedure, see the installation instructions on the [{{site.data.keyword.icp4dfull_notm}} knowledge center 3.5](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-install.html){: external}

The installation process takes 1 or 2 hours. These example commands are for installing in a cluster with internet access.

1.  Copy images to your cluster. For example:

    ```
    ./cpd-cli adm --repo repo.yaml --assembly watson-assistant --namespace zen --apply
    ```
    {: codeblock}

    To see a sample `repo.yaml` that installs assistant only, see [Setting up the cluster](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-svc-install-adm.html){: external}

1.  Get the Cloud Pak layer images. For example:

    ```
    ./cpd-cli install --assembly lite --namespace zen --repo repo.yaml \
    --storageclass portworx-shared-gp3 --target-registry-username=ocadmin \
    --target-registry-password=$(oc whoami -t) --cluster-pull-prefix $(oc registry info)/zen \ --transfer-image-to $(oc get routes docker-registry -n default -o=template={{.spec.host}})/zen
    ```
    {: codeblock}

    If you see `CrashLoopBackoff` messages, you can ingore them. Give the pods time to recover and the installation will complete.

1.  Get the Events service images.

    The Events service is required if you want to use the Analytics feature.

    Follow the instructions [here](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/common-svcs.html){: external}. If you have the option to install only the Events service, do so.

1.  Install the EDB operator to get Postgres resources. For example:

    ```
    ./cpd-cli install --assembly edb-operator --optional-modules edb-pg-base:x86_64 \
    --namespace zen --repo repo.yaml --cluster-pull-prefix $(oc registry info \
    --internal)/zen --transfer-image-to=$(oc registry info)/zen \
    --ask-push-registry-credentials --insecure-skip-tls-verify
    ```
    {: codeblock}

1.  Get the images for the service. For example:

    ```
    ./cpd-cli install --assembly watson-assistant-operator \
    --optional-modules watson-assistant-operand-ibm-events-operator:x86_64 \
    --override install-override.yaml --namespace zen --repo repo.yaml \
    --cluster-pull-prefix $(oc registry info --internal)/zen \
    --transfer-image-to=$(oc registry info)/zen --ask-push-registry-credentials \
    --insecure-skip-tls-verify
    ```
    {: codeblock}

    For details about the override file, see [Creating an override file](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-svc-override.html){: external}

1.  Install the service. For example:

    ```
    ./cpd-cli  install --assembly watson-assistant --instance wa001 \
    --override install-override.yaml --namespace zen --repo repo.yaml \
    --storageclass portworx-watson-assistant-sc \
    --cluster-pull-prefix $(oc registry info --internal)/zen \
    --transfer-image-to=$(oc registry info)/zen --ask-push-registry-credentials \
    --insecure-skip-tls-verify
    ```
    {: codeblock}

## Installing on IBM Cloud Pak for Data 3.0.1
{: #install-150-on-301}

The latest release of {{site.data.keyword.conversationshort}} uses operators. (An operator is a method of packaging, deploying, and managing a Kubernetes-native application in a Red Hat OpenShift environment.) You must use the cpd-cli command line that is available only with {{site.data.keyword.icp4dfull_notm}} 3.5 to install it.

To install {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} on {{site.data.keyword.icp4dfull_notm}} 3.0.1, complete the following steps:

1.  Install {{site.data.keyword.icp4dfull_notm}} 3.0.1 by following the instruction in the [{{site.data.keyword.icp4dfull_notm}} 3.0.1 knowledge center](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/install.html){: external}.

1.  Get the {{site.data.keyword.icp4dfull_notm}} 3.5 version of the command line interface by following the instructions in the [{{site.data.keyword.icp4dfull_notm}} 3.5 knowledge center](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/installation-files.html){: external}. Complete steps 1 through 2a only of the procedure for obtaining the installation files.

1.  Run the following command to instruct {{site.data.keyword.icp4dfull_notm}} to use the new (3.5) command line interface version.

    ```
    ./cpd-cli operator-upgrade -n $PROJECT
    ```
    {: codeblock}

    where $PROJECT is the project where the {{site.data.keyword.icp4dfull_notm}} control plane to be upgraded is deployed.

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

## Uninstalling on IBM Cloud Pak for Data 3.5
{: #install-150-uninstall-35}

The steps to uninstall are outlined here:

1.  **Optional**: From the web client, remove any provisioned instances of you service. 

    This step is a useful check for you to see what instances will be removed when you uninstall the service.

    - Log in to the Cloud Pak for Data web client as an administrator.
    - From the menu, select *Services > Instances*.
    - Filter the list to show only watson-assistant service instances.
    - Delete all of the instances of the service.

1.  Change to the directory where you placed the Cloud Pak for Data command-line interface, and then log in to your Red Hat OpenShift cluster as a project administrator:

    ```
    oc login ${OpenShift_URL:port}
    ```
    {: codeblock}

1.  To uninstall Watson Assistant, you must run the installer multiple times to install the following assemblies in this order:

    - **watson-assistant**: The assistant assembly installs the service. When you uninstall this assembly, you must specify the `--instance Instance_name` parameter, which represents the deployment name. For example:

      ```
      ./cpd-cli uninstall --assembly  watson-assistant --instance wa001 -n zen
      ```
      {: codeblock}

    - **watson-assistant-operator**: The assistant operator is a prerequisite for Watson Assistant that installs resources that are required by the service.

      ```
      ./cpd-cli uninstall --assembly  watson-assistant-operator -n zen
      ```
      {: codeblock}

1.  Remove the persistent volume claims that are created by the service.

    - Use the following command to get a list of claims that are used by the service:

      ```
      oc get pvc -l icpdsupport/addOnId=assistant
      ```
      {: codeblock}

    - Run the following command for each volume claim to delete them one at a time:

      ```
      oc delete pvc ${pvc name}
      ```
      {: codeblock}

## Next steps
{: #install-150-next-steps}

Use the {{site.data.keyword.conversationshort}} product user interface to build training data and a dialog that can be used by your assistant.

- To learn more about the service first, read the [overview](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-index).
- To see how it works for yourself, follow the steps in the [getting started tutorial](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-getting-started).
- For help managing the cluster, see [Managing the cluster](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-manage).
