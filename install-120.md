---

copyright:
  years: 2015, 2019
lastupdated: "2019-06-27"

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

# Installing 1.2
{: #install-120}

Use {{site.data.keyword.conversationfull}} for {{site.data.keyword.icp4dfull}} V1.2.0 to build conversational interfaces into any app, device, or channel. 
{: shortdesc}

Use this installation method if you do not have a {{site.data.keyword.icp4dfull}} V2.1.0 cluster. These instructions describe how to install {{site.data.keyword.icp4dfull}} V2.1.0, and then add {{site.data.keyword.conversationshort}} to it as an add-on. You cannot install V1.2 into a {{site.data.keyword.icpfull_notm}} standalone environment.

You can deploy {{site.data.keyword.conversationshort}} one time only in a single cluster.

## Application details
{: #install-120-wa-details}

### Microservices
{: #install-120-microservices}

Microservices are individual components that together comprise a service. {{site.data.keyword.conversationshort}} consists of the following microservices:

- **NLU (Natural Language Understanding)**: Interface for store to communicate with the back-end to initiate ML training.
- **Dialog**: Dialog runtime, or user-chat capability.
- **ed-mm**: Manages contextual entity capabilities.
- **Master**: Controls the lifecycle of underlying intent and entity models.
- **Recommends**: Supports recommendations from Watson, such as dictionary-based entity synonyms and intent conflicts. 
- **SIREG** - Manages tokenization and system entity capabilities.
- **skill-conversation**: Manages dialog skills.
- **skill-search**: Manages search skills.
- **SLAD**: Manages service training capabilities.
- **Store**: API endpoints.
- **TAS**: Manages services model inferencing.
- **UI**: Provides the developer user interface.

In addition to these microservices, the Helm chart installs the following resources:

- **PostgreSQL**: Stores training data. Includes the components keeper, sentinel, and proxy.
- **MongoDB**: Stores word vectors.
- **Redis**: Used by the {{site.data.keyword.conversationshort}} tool to store web session-related data.
- **etcd**: Manages service registration and discovery.
- **Minio**: Stores CLU models.

### Language considerations
{: #install-120-lang-reqs}

The components that are necessary to process natural languages require significant amounts of data. The base set of supported languages require 40 GB of memory per node. The following languages require that you add additional resources to support them.

Table 1. Language resource requirements

| Language | Pods for production | Pods for development | Additional memory requirements per pod | 
|----------|---------------------|----------------------|----------------------------------------|
| Chinese (Simplified or Traditional or both) | 2 | 1 | 8 GB |
| German | 2 | 1 | 1 GB |
| Japanese | 2 | 1 | 1 GB |
| Korean | 2 | 1 | 2 GB |
{: caption="Language resource requirements" caption-side="top"}

For the full list of supported languages, see [Supported languages](https://cloud.ibm.com/docs/services/assistant-data?topic=assistant-data-language-support).

## System requirements
{: #install-120-reqs-over}

Before you install the add-on, ensure that you have sufficient resources to run the add-on. The following resources are required in addition to the minimum requirements.

For details of the minimum requirements that must be met to support {{site.data.keyword.icp4dfull}} itself, see [System requirements](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/install/reqs-ent.html){: external}. 
<!--Lite sys req details? -->

### Minimum requirements
{: #install-120-reqs-minimum}

The systems that host {{site.data.keyword.conversationshort}} must meet these requirements:

- {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} can run on Intel architecture nodes only.
- CPUs must have 2.4 GHz or higher clock speed
- CPUs must support Linux SSE 4.2
- CPUs must support the AVX instruction set extension See the [Advanced Vector Extensions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions){: external} Wikipedia page for a list of CPUs that include this support (most CPUs since 2012). The service cannot function properly without AVX support.

<!--The cluster must provide the following number of Virtual Private CPUs (VPCs) to support {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} at a minimum.

- **Production**: 27 CPU with 112 GB Memory across a minimum of 4 worker nodes
- **Development**: 21 CPU with 75 GB Memory across a minimum of 1 worker node

These numbers reflect the bare minimum requirements. In a cluster environment, where CPU and memory are assigned to containers dynamically, CPU and memory resources can become stranded on nodes, leaving insufficient resources to schedule subsequent workloads. In particular, the process of training a machine learning model requires at least one node to have 4 CPUs that can be dedicated to training. This capacity is only needed when training occurs, which happens after changes are made to the training data for an assistant.
{: important}
-->
### Optimal deployment configuration for production
{: #install-120-tested-sys-reqs-prod}

Table 2. Hardware verified to support a production deployment of the add-on

| Node type | Number of nodes | CPU per node | Memory per node (GB) | Disk per node (GB) |
|-----------|-----------------|--------------|-----------------|---------------|
| worker | 4 | 8 | 64 | 500 |
{: caption="Production hardware requirements" caption-side="top"}

### Optimal deployment configuration for development
{: #install-120-tested-sys-reqs-dev}

Table 3. Hardware verified to support a development deployment of the add-on

| Node type | Number of nodes | CPU per node | Memory per node (GB) | Disk per node (GB) |
|-----------|-----------------|--------------|-----------------|---------------|
| worker | 3  | 8 | 64 | 500 |
{: caption="Non-production hardware requirements" caption-side="top"}

## Storage requirements
{: #install-120-storage-reqs}

The following table lists the storage resources that are required to support a deployment.

Table 4. Storage requirements

| Component | Number of replicas | Space per pod | Storage type |
|-----------|-----------------|--------------|
| Postgres  | 3 | 10 GB | local-storage |
| etcd      | 3 | 10 GB | local-storage |
| Minio     | 4 |  5 GB | local-storage |
| MongoDB   | 3 | 80 GB | local-storage |
{: caption="Storage requirements" caption-side="top"}

## Overview of the steps
{: #install-120-task-overview}

Follow these steps to install {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}}.

1.  [Install the cluster](#install-120-install-icp4d)
1.  [Purchase and download installation artifacts](#install-120-download-wa-icp)
1.  [Create a namespace](#install-120-create-namespace)
1.  [Upload the archive file](#install-120-upload-archive)
1.  [Extract files from the chart](#install-120-extract)
1.  [Create persistent volumes](#install-120-create-pvs)
1.  [Set up security policies](#install-120-apply-security-policy)
1.  [Create the image policy](#install-120-image-policy)
1.  [Customize the configuration](#install-120-config)
1.  [Install from the Helm chart](#install-120-load-helm-chart)
1.  [Verify that the installation was successful](#install-120-verify)
1.  [Provision an instance of the add-on](#install-120-install-add-on)
1.  [Launch the product](#install-120-launch-tool)

## Step 1: Install the cluster
{: #install-120-install-icp4d}

1.  Install {{site.data.keyword.icp4dfull_notm}}. 

    Follow the instructions to install and set it up that begin with [Installing](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/install/ovu.html){: external}.
    <!--Lite install config details? -->

1.  Review the following topics about cluster security and take steps to implement any security measures that you want to have in place before you install the add-on:

- [Security in IBM Cloud Pak for Data](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/overview/security.html){: external}

  Encryption of data at rest must be managed by the storage provider.
  {: important}

- [Use a custom SSL or TLS certificate for HTTPS connections to the web client](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/install/https-config.html){: external}
- Post-installation instructions in [Encrypting cluster data network traffic with IPsec](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.2/installing/ipsec_mesh.html#ipsec-certificate-and-key)
- Red Hat OpenShift-specific information about [encrypting traffic between nodes with IPsec](https://docs.openshift.com/container-platform/3.11/admin_guide/ipsec.html){: external}

## Step 2: Purchase and download installation artifacts
{: #install-120-download-wa-icp}

After you purchase the add-on, you download the software as a Passport Advantage archive (PPA) file. The PPA file for {{site.data.keyword.conversationshort}} contains a Helm chart and images. Helm is the Kubernetes package management system that is used for application management inside an {{site.data.keyword.icp4dfull_notm}} cluster.

1.  Purchase {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} from [Passport Advantage](https://www.ibm.com/software/passportadvantage/index.html){: external}.

1.  Use the Secure Shell protocol to log in to the master node (master-1) of your {{site.data.keyword.icp4dfull_notm}} cluster as the root user.

1.  Change to the directory where you want the installation files to be stored.

1.  Download the archive file.

    This download takes about a half hour to an hour to complete over a network connection.

## Step 3: Create a namespace
{: #install-120-create-namespace}

Create a namespace for your application. Namespaces are a way to divide cluster resources between multiple users, which can be managed by resource quota.

1.  From the Kubernetes CLI, run a command with the following syntax:

    ```bash
    kubectl create namespace {namespace-name}
    ```
    {:pre}

    For example:
    ```bash
    kubectl create namespace assistant
    ```
    {:pre}

If you have any trouble running kubectl commands, see [Enabling access to kubectl](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/install/kubectl-access.html).

## Step 4: Upload the archive file
{: #install-120-upload-archive}

After the download of the archive file from Passport Advantage is completed, load the file onto the cluster. The file must be available to the cluster before you can use Helm commands to install the add-on.

1.  From the {{site.data.keyword.icpfull_notm}} command line interface, run the following command to log in:

    ```bash
    cloudctl login -a {cluster4d-master-node}:8443 -u {admin-user-id} -p {admin password} 
    ```
    {: pre}

1.  When prompted for a namespace, provide the namespace you created in the previous step.

1.  Run the following command to upload the archive file to the cluster:

    ```bash
    cloudctl catalog load-archive --registry {cluster4d-master-node}:8500 --archive {archive-name}.tar.gz --repo local-charts
    ```
    {: pre}

    Do not include a protocol prefix (such as `https://`) with the {cluster4d-master-node} value.
    {: important}

    For example:

    ```bash
    cloudctl catalog load-archive --registry mycloud.example.com:8500 --archive {archive-name}.tar.gz --repo local-charts
    ```
    {: pre}

    This process takes a while.

    If there is a change that the terminal window connection to your cluster may be interrupted before the command completes, you can help the command complete even if you lose your connection. Pause the job, and run it it the background (bg). Then, remove the inheritance path of the interrupt (SIGHUP) signal (disown -h %{job#}) to prevent it from stopping the job prematurely.
    {: tip}

If you get the following error message, it means you included `https://` with the cluster address. Try to load the file again without it.

`Error parsing reference: "https://{cluster4d-master-node}:8500/us.icr.io/icp-common-components/wcn-addon:1.x" is not a valid repository/tag: invalid reference format`

## Step 5: Extract files from the archive
{: #install-120-extract}

After the archive file is loaded into the cluster, you can extract files from it. There are scripts provided with the chart that you can use to perform tasks such as creating persistent volumes and applying security policies.

1.  Change to the directory where you downloaded the archive file, and then use the following command to expand the archive. 

    This step is important because the `values.yaml`, which defines configuration settings for your deployment, gets populated with information during the process. If you were to extract the files without expanding them, this information would not be populated properly.

    ```bash
    wget --no-check-certificate https://{cluster4d-master-node}:8443/helm-repo/requiredAssets/ibm-watson-assistant-prod-1.2.0.tgz
    ```
    {: pre}

    If you are using a load balancer, then the {cluster4d-master-node} is the hostname for the load balancer.
    {: note}

1.  Extract the files from the Helm chart package with the following command:

    ```bash
    tar -xvf ibm-watson-assistant-prod-1.2.0.tgz
    ```
    {: pre}

    The root directory that is extracted from the package is named **ibm-watson-assistant-prod**.

## Step 6: Create persistent volumes
{: #install-120-create-pvs}

A PersistentVolume (PV) is a unit of storage in the cluster. In the same way that a node is a cluster resource, a persistent volume is also a resource in the cluster.

For an overview, see [Persistent Volumes in the Kubernetes documentation ](https://kubernetes.io/docs/concepts/storage/persistent-volumes/){: external}.

You must be a cluster administrator to create local storage volumes, and the script used to create them must be run from the master node of the cluster.
{: important}

Follow the correct procedure for your deployment.

- [Creating persistent volumes for a production environment](#install-120-create-pvs-prod)
- [Creating persistent volumes for a development environment](#install-120-create-pvs-dev)

### Creating persistent volumes for a production environment
{: #install-120-create-pvs-prod}

Consider using an {{site.data.keyword.icp4dfull_notm}} storage [add-on](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/admin/add-ons.html#add-ons__storage){: external} or a storage option that is hosted outside the cluster, such as [vSphere Cloud Provider](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.2/manage_cluster/vsphere_land.html){: external}.

### Creating persistent volumes for a development environment
{: #install-120-create-pvs-dev}

When you install the service, persistent volume claims are created for the components automatically. However, because the preferred storage class for the service is **local-storage**, you must explicitly create persistent volumes before you install the service.

A script is included in the archive file that you can use to create persistent volumes for a development deployment only. 

For more information about the commands used in the script, see [Creating a PersistentVolume](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.2/manage_cluster/create_volume.html){: external}.

1.  From the master node, change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/pre-install** subdirectory of the archive file that you extracted the product files from earlier.

1.  Run the **createLocalVolumePV.sh** script by using a command with the following sytnax:

    ```bash
    ./clusterAdministration/createLocalVolumePV.sh [--path PATH]
    ```
    {: pre}

    - `path`: Specify a directory path for the physical location of the volume. If you don't, the path `/mnt/local-storage/storage/watson/assistant` is used. The script appends a unique directory name for each volume, such as `/pv-80gb-n`, to the end of file path you specify.

1.  Use the following command to verify that the persistent volumes were created successfully.

    ```bash
    kubectl get pv
    ```
    {: pre}

    You might want to filter the list. For example, you can pipe the command to `grep gb-` because the script uses the naming convention `pv-#gb-#` for the volumes it creates.
    {: tip}

    The resulting list should look something like this:

    ```bash
    pv-10gb-1 10G RWO Recycle Available local-storage 11m
    pv-10gb-2 10G RWO Recycle Available local-storage 7m
    pv-10gb-3 10G RWO Recycle Available local-storage 7m
    pv-10gb-4 10G RWO Recycle Available local-storage 7m
    pv-10gb-5 10G RWO Recycle Available local-storage 7m
    pv-10gb-6 10G RWO Recycle Available local-storage 7m
    pv-5gb-1 5G RWO Recycle Available local-storage 7m
    pv-5gb-2 5G RWO Recycle Available local-storage 7m
    pv-5gb-3 5G RWO Recycle Available local-storage 7m
    pv-5gb-4 5G RWO Recycle Available local-storage 7m
    pv-80gb-1 80G RWO Recycle Available local-storage 7m
    pv-80gb-2 80G RWO Recycle Available local-storage 7m
    pv-80gb-3 80G RWO Recycle Available local-storage 7m
    ```
    {: pre}

## Step 7: Set up security policies
{: #install-120-apply-security-policy}

A set of scripts is provided with the {{site.data.keyword.conversationshort}} archive package. Use the scripts to set up the appropriate security policies. The provided scripts include:

- **createSecurityNamespacePrereqs.sh**: Creates a role binding in the namespace specified and prevents pods that don't meet the `ibm-restricted-psp` pod security policy from being started. The policy named `ibm-restricted-psp` is the most restrictive policy. It requires pods to run with a non-root user ID and prevents pods from accessing the host. The role binding rules are defined in the `ibm-watson-assistant-prod-roldebinding.tpl` file, which is also provided in the archive.
- **labelNamespace.sh**: Adds the cluster namespace label to your namespace. The label is needed to permit communication between your application's namespace and the {{site.data.keyword.icp4dfull_notm}} namespace using a network policy.
 
For more information about the `ibm-restricted-psp` security policy, see the Helm Chart README.md file. A link to the file is available from the table of contents.
{: tip}

You must be a cluster administrator to run the scripts.
{: important}

1.  Change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/pre-install** directory.

1.  Run the **createSecurityNamespacePrereqs.sh** script by using a command with the following sytnax:

    ```bash
    ./namespaceAdministration/createSecurityNamespacePrereqs.sh {namespace-name}
    ```
    {: pre}

    For example:

    ```bash
    ./namespaceAdministration/createSecurityNamespacePrereqs.sh assistant
    ```
    {: pre}

1.  Run the **labelNamespace.sh** script.

    This command assumes that you used **zen** as the {{site.data.keyword.icp4dfull_notm}} namespace when you installed {{site.data.keyword.icp4dfull_notm}}. If you used a different namespace name, then specify it instead.
    {: note} 

    ```bash
    ./clusterAdministration/labelNamespace.sh zen
    ```
    {: pre}

## Step 8: Create the image policy
{: #install-120-image-policy}

For each image in a repository, an image policy scope of either cluster or namespace is applied. When you deploy an application, IBM Container Image Security Enforcement checks whether the Kubernetes namespace that you are deploying to has any policy regulations that must be applied.

1.  Create a YAML file named `policy.yaml`, and add the following content to the file:

    ```
    apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
    kind: ClusterImagePolicy
    metadata:
     name: watson-assistant-{name}-policy
    spec:
     repositories:
        - name: "{cluster4d-master-node}:8500/*"
          policy:
            va:
              enabled: false
    ```
    {: codeblock}

    Replace the following variables with the appropriate values for your cluster:

    - `{cluster4d-master-node}`: Specify the hostname for the master node of the {{site.data.keyword.icp4dfull_notm}} cluster
    - `{name}`: Specify a name that helps you identify this deployment. You can use the version number of the product, such as 120, for example.

1.  Apply the policy by running the following command:

    ```bash
    kubectl apply -f policy.yaml
    ```
    {: pre}

1.  To validate that the policy was applied properly, run the following command:

    ```bash
    kubectl get ClusterImagePolicy
    ```
    {: pre}

    A list of policies is returned. Confirm that the policy you created (for example, watson-assistant-120-policy) is included in the list. 
    
    If you want to list the content of the policy, use this command instead:

    ```bash
    kubectl get ClusterImagePolicy -o yaml
    ```
    {: pre}

## Step 9: Customize the configuration
{: #install-120-config}

The configuration settings for the deployment are defined in a file named `values.yaml`. Create a copy of the file and name it `values-override.yaml`. In this override file, you can change values for some of the default configuration settings to customize your add-on deployment.

1.  Return to the directory where you extracted the archive files earlier, and then make a copy of the `values.yaml` file.

    ```bash
    cp values.yaml values-override.yaml
    ```
    {: pre}

1.  Open the `values-override.yaml` file in a text editor.

1.  Replace any values that you want to change.

   At a minimum, you must provide your own values for the following configurable settings:

    - `global.deploymentType`: Specify whether you want to set up a **Development** or **Production** instance. These values are uppercase and the setting is case sensitive.
    - `global.icp.masterHostname`: Specify the hostname of the master node of your cloud instance. Do not include the protocol prefix (`https://`) or port number (`:8443`).  For example: `my.company.name.icp.net`.
    - `global.icp.masterIP`: If you did not define a domain name for the master node of your cloud instance, then you must also specify the IP address of the master node.
    - `global.languages.{language-name}`: Change the value for an individual language to **true** to enable it. English and Czech are enabled by default. Additional resources are required to support additional languages.
    - `license`: Read the license files that are provided in the `LICENSES` directory within the archive package. If you agree to the terms, set this configuration setting to **accept**. 

       You cannot install the product if you do not accept the license.
       {: note}

    **Attention**: Currently, the service does not support the ability to provide your own instances of resources, such as Postgres or MongoDB. The values YAML file has `{resource-name}.create` settings that suggest you can do so. However, do not change these settings from their default value of `true`.

1.  Save and close the `values-override.yaml` file.

For information about other values in the YAML file, see [Configuration details](#install-120-config-details).

## Step 10: Install from the Helm chart
{: #install-120-load-helm-chart}

1.  Log in to the {{site.data.keyword.icp4dfull_notm}} command line interface again. 
    
    ```bash
    cloudctl login -u <clusteradmin> -p <password>
    ```
    {: pre}

    Be sure to choose the new namespace in which you are installing the product.
    {: important}

1.  Verify that your Helm command line interface context is valid by running this command.

    ```bash
    helm version --tls
    ```
    {: pre}

1.  Install the chart from the Helm command line interface. 

    Enter the following command from the bm-waton-assistant-prod directory:

    ```bash
    helm install --tls --values {override-file-name} --namespace {namespace-name) --name {my-release} ibm-watson-assistant-prod-1.2.0.tgz
    ```
    {: pre}
​​​
    - Replace `{my-release}` with a name for your release. The release name must start with an alphabetic character, end with an alphanumeric character, and consist of lower case alphanumeric characters or a hyphen (-). For example *my-120-wa*.
    - Replace `{override-file-name}` with the path to the file that contains the values that you want to override from the values.yaml file provided with the chart package. For example: `my-override.yaml`
    - Replace `{namespace-name}` with the name of the Kubernetes namespace that hosts the Docker pods.
    - The `ibm-watson-assistant-prod-1.2.0.tgz` parameter represents the name of the downloaded file that contains the Helm chart.

    For example:

    ```bash
    helm install --tls --values values-override.yaml --namespace assistant --name my120wa ../ibm-watson-assistant-prod-1.2.0.tgz
    ```
    {: pre}

If you see the following recurring message, you can safely ignore it: `warning: cannot overwrite table with non table for affinity (map[])`

## Step 11: Verify that the installation was successful
{: #install-120-verify}

To check the status of the installation process:

1.  Check the status of the deployment by using the following command:

      ```bash
      watch kubectl get job,pod,svc,secret,cm,pvc --namespace {namespace-name}
      ```
      {: pre}

      Be sure to give the product time to be deployed. The pod that hosts the Recommends microservice takes 30-45 minutes to reach the `Running` state.
      {: important}

      ```bash
      helm status --tls {release-name}
      ```
      {: pre}

1.    Take one of the following actions:

      - If a pod fails, try to determine the cause and fix it.
      
        1. Run the following command to see logs for the pod:

           ```bash
           kubectl logs {podname} -n {namespace} -f --timestamps
           ```
           {: pre}

          If you cannot resolve the cause of the problem and need to start the installation over, complete the steps in [Uninstalling the service](#install-120-uninstall), so you can start over with a clean set of nodes.

    - If the deployment process was successful, test {{site.data.keyword.conversationshort}} by running a test Helm chart.

        1.  From the Helm command line interface, run the following command:

            ```bash
            helm test --tls {release name} --timeout 900
            ```
            {: pre}

        1.  If one of the tests fails, review the logs to learn more. To see the log, use a command with the syntax `kubectl logs {podname} -n {namespace-name} -f --timestamps`. For example:

            ```bash
            kubectl logs my-release-test -n assistant -f --timestamps
            ```
            {: pre}

        1.  To run the test script again, first delete the test pods by using a command with the syntax `kubectl delete pod {podname} --namespace {namespace-name}`. For example:

            ```bash
            kubectl delete pod my-release-test --namespace assistant
            ```
            {: pre}

            You must delete all of the `{podname}` pods, not just the one that failed, or the other tests will fail also.

### Uninstalling the service
{: #install-120-uninstall}

If you need to start the deployment over, be sure to remove all trace of the current installation before you try to install again.

If you need to preserve any data, do so now before you begin this procedure.

1.  {: #install-120-delete-instance-sh}If you got as far as creating an instance of the add-on, then complete the following steps to delete the add-on instance. Otherwise, skip this step.

    1. From the {{site.data.keyword.icp4dfull_notm}} web client, go to the **My Instances** page. 

    1. Find the {{site.data.keyword.conversationshort}} instance, click the More menu, and then choose **Delete**.

    1. To clean up artifacts that are associated with the instance, create a script named `deleteInstance.sh` and add the following snippet to it:

    ```
    if [ "$#" -lt 1 ]; then
      echo "Usage: $0 ICP4D_NAMESPACE (Where ICP4D is installed)"
      exit 1
    fi

    namespace=$1

    kubectl -n $namespace exec zen-metastoredb-0 \
    -- sh /cockroach/cockroach.sh sql  \
    --insecure -e "DELETE FROM zen.service_instances WHERE deleted_at IS NOT NULL RETURNING id;" \
    --host='zen-metastoredb-public'

    if [ $? -eq 0 ]; then
      exit 0
    else
      exit 1
    fi
    ```
    {: codeblock}

1.  Run the script that you created.

    ```bash
    ./deleteInstance.sh {cluster-namespace-name}
    ```
    {: pre}

    For example:

    ```bash
    ./deleteInstance.sh zen
    ```
    {: pre}

1.  To delete the deployment of the add-on, first check the status of the deployment.

    ```bash
    helm list --tls
    ```
    {: pre}

1.  Before you delete anything, get a list of the names of the persistent volumes that you created for this deployment.

    ```bash
    kubectl get pv
    ```
    {: pre}

    Make a note of the name of each persistent volume. You will need this information in a later step.

1.  Delete the deployment by using the following command:

    ```bash
    helm delete --tls --no-hooks --purge {release name}
    ```
    {: pre}

1.  Delete any associated artifacts that are left over by using the following command. 

    Reminder: This command removes the data stores you created also, which must be removed and recreated if you need to start the installation over.
    {: note}

    ```bash
    kubectl delete job,deploy,rs,pod,statefulset,configmap,secret,ingress,service,serviceaccount,role,rolebinding,pvc,poddisruptionbudget -l release={release-name}
    ```
    {: pre}

    The `-l` for label is the selector to filter by, where you can specify the release name.

1.  Remove the configmap by using the following command.

    ```bash
    kubectl delete cm stolon-cluster-{release name}
    ```
    {: pre}

1.  To delete each physical persistent volume, use the following command. 

    You should have a list of the names of each persistent volume that you noted in an earlier step.

    ```bash
    kubectl delete pv (pv-name}
    ```
    {: pre}

Now, you have cleared everything necessary to restart your installation. Your namespace exists and your add-on archive file is loaded into it, so you can pick up the installation instructions starting from [Step 6](#install-120-create-pvs).

## Step 12: Provision an instance of the add-on
{: #install-120-install-add-on}

1.  If you provisioned an instance previously, you must run a script to delete the instance from the {{site.data.keyword.icp4dfull_notm}} database before you can provision a new instance. If you did not delete the prior instance, do so now. Otherwise, skip this step.

    You can provision one instance of {{site.data.keyword.conversationshort}} only per cluster. 
    
    To delete a previous instance, run the **deleteInstances.sh** script. For more details, see [Step 1 of the Uninstalling procedure](#install-120-uninstall).

1.  From the {{site.data.keyword.icp4dfull_notm}} web client, go to the Add-ons page.

1.  Find the {{site.data.keyword.conversationshort}} add-on and click it.

1.  Click **Provision instance**.

1.  Click **Create an instance**.

1.  Name the instance. 

    This is the instance you will share with the users in your organization. They will see this instance name from the product's main page. Choose a name that represents the instance's purpose.

1.  Click **Create**. 

## Step 13: Launch the product
{: #install-120-launch-tool}

1.  From the instance you created, click **Launch Tool**.
1.  Log in using the same credentials you use to log into {{site.data.keyword.icp4dfull_notm}}.

### Investigating issues
{: #install-120-ts-get-logs}

The first step to take if you hit an installation issue, such as a cluster node is not starting as expected, is to get logs from the cluster which can provide more detail.

To get log files, complete the following steps:

1.  Log into the cluster with administrator credentials.

1.  Run the following command to get a list of the jobs that are currently running in the cluster and whether the job was successful:

    ```bash
    kubectl get jobs
    ```
    {: pre}

1.  For any jobs that show a success status of 0, get the log file for the job by entering the following command:

    ```bash
    kubectl log {job-name} -f
    ```
    {: pre}

### To check the configuration
{: #install-120-check-config}

If you want to check what configuration settings were applied to a deployment when it was set up, you can run the following command:

```bash
helm get <release-name> --tls
```
{: pre}

The user-provided configuration values are listed at the start of the information that is returned.

## Next steps
{: #install-120-next-steps}

Use the {{site.data.keyword.conversationshort}} product user interface to build training data and a dialog that can be used by your assistant.

- To learn more about the service first, read the [overview](https://cloud.ibm.com/docs/services/assistant-data?topic=assistant-data-index).
- To see how it works for yourself, follow the steps in the [getting started tutorial](https://cloud.ibm.com/docs/services/assistant-data?topic=assistant-data-getting-started).
- For help managing the cluster, see [Managing the cluster](https://cloud.ibm.com/docs/services/assistant-data?topic=assistant-data-manage).
