---

copyright:
  years: 2015, 2020
lastupdated: "2020-08-12"

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

# Installing Watson Assistant 1.4.2
{: #install-142}

Use {{site.data.keyword.conversationfull}} for {{site.data.keyword.icp4dfull}} to build conversational interfaces into any app, device, or channel. 
{: shortdesc}

Use this installation method if you do not have a {{site.data.keyword.icp4dfull}} cluster. These instructions include pointers to instructions for installing {{site.data.keyword.icp4dfull}} 2.5 as a Red Hat OpenShift 3.11 deployment or installing {{site.data.keyword.icp4dfull}} 3.0.1 as a Red Hat OpenShift 3.11 or 4.3 deployment. You then add {{site.data.keyword.conversationshort}} to it as a service.

## Application details
{: #install-142-wa-details}

### Microservices
{: #install-142-microservices}

Microservices are individual components that together comprise a service. {{site.data.keyword.conversationshort}} consists of the following microservices:

- **NLU (Natural Language Understanding)**: Interface for store to communicate with the back-end to initiate ML training.
- **CLU Embeddings**: Manages word embeddings for CLU.
- **Dialog**: Dialog runtime, or user-chat capability.
- **ed-mm**: Manages contextual entity capabilities.
- **Master**: Controls the lifecycle of underlying intent and entity models.
- **Recommends**: Supports recommendations from Watson, such as dictionary-based entity synonyms and intent conflicts. 
- **SIREG** - Manages tokenization and system entity capabilities.
- **skill-search**: Manages search skills.
- **SLAD**: Manages service training capabilities.
- **Spellchecker**: Corrects spelling mistakes in user input.
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
{: #install-142-lang-reqs}

The components that are necessary to process some natural languages require more resources. The following languages require that additional CPU and memory be available for the installation.

Table 1. Language resource requirements

| Language | Pods for production | Pods for development | Additional memory requirements per pod | 
|----------|---------------------|----------------------|------------|
| Chinese (Simplified or Traditional or both) | 2 | 1 | 8 GB |
| German | 2 | 1 | 2 GB |
| Japanese | 2 | 1 | 2 GB |
| Korean | 2 | 1 | 4 GB |
{: caption="Language resource requirements" caption-side="top"}

Each of these languages requires an additional VPC for a production deployment and an additional 1/2 VPC for a development deployment.

For the full list of supported languages, see [Supported languages](/docs/assistant-data?topic=assistant-data-language-support).

At installation time, you must specify a configuration setting for each language that you want to support. For information about how to enable languages in the values.yaml file, see one of the following sections:

- **{{site.data.keyword.icp4dfull_notm}} 3.0.1**: [Customize the configuration](#install-142-cpd30-config)
- **{{site.data.keyword.icp4dfull_notm}} 2.5**: [Customize the configuration](#install-142-cpd25-config)

You must enable additional languages when you install the service; you cannot enable them afterward.
{: important}

## System requirements
{: #install-142-reqs-over}

For details of the minimum requirements that must be met to support {{site.data.keyword.icp4dfull}} itself, see the appropriate documentation.

- **3.0.1**: [System requirements](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/plan/rhos-reqs.html){: external}. 
- **2.5**: [System requirements](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/plan/rhos-reqs.html){: external}. 

### Service requirements
{: #install-142-reqs-addon}

Before you install the service, ensure that you have sufficient resources to run the service. The following resources are required in addition to the minimum platform requirements. 

In development:

- Minimum worker nodes: 3
- Minimum CPU available: 7
- Minimum memory available: 100Gi
- Storage required for physical volumes : 326Gi
- Minimum disk per node available: 500 GB

In production:

- Minimum worker nodes: 5
- Minimum CPU available: 10
- Minimum memory available: 150Gi
- Storage required for physical volumes : 326Gi
- Minimum disk per node available: 500 GB

### Optimal deployment configuration for development
{: #install-142-tested-sys-reqs-dev}

Table 2. Hardware verified to support a development deployment of the service with {{site.data.keyword.icp4dfull_notm}}

| Number of nodes | CPU per node | Memory per node (GB) | Disk per node (GB) |
|-----------------|--------------|-----------------|---------------|
| 3 | 8 | 64 | 500 |
{: caption="Non-production hardware requirements" caption-side="top"}

Keep in mind that in a cluster environment, where CPU and memory are assigned to containers dynamically, CPU and memory resources can become stranded on nodes, leaving insufficient resources to schedule subsequent workloads. In particular, the process of training a machine learning model requires at least one node to have 4 CPUs that can be dedicated to training. This capacity is only needed when training occurs, which happens after changes are made to the training data for an assistant.
{: important}

#### Storage requirements
{: #install-142-storage-reqs}

The following table lists the storage resources that are required to support the persistent volumes that are used by the service.

Table 3. Storage requirements

| Component | Number of replicas | Space per pod |
|-----------|--------------------|---------------|
| Postgres  | 3 | 10 GB |
| etcd      | 5 | 10 GB |
| Minio     | 4 |  5 GB |
| MongoDB   | 3 | 75 GB |
| Backup    | 1 |  1 GB |
{: caption="Storage details" caption-side="top"}

The systems that host {{site.data.keyword.conversationshort}} must meet these additional requirements:

- {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} can run on Intel architecture nodes only.
- CPUs must have 2.4 GHz or higher clock speed
- CPUs must support Linux SSE 4.2
- CPUs must support the AVX2 instruction set extension. See the [Advanced Vector Extensions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2){: external} Wikipedia page for a list of CPUs that include this support (most CPUs since 2013). The service cannot function properly without AVX2 support.

## Creating persistent volumes
{: #install-142-create-pvs}

A PersistentVolume (PV) is a unit of storage in the cluster. In the same way that a node is a cluster resource, a persistent volume is also a resource in the cluster.

For more information, see [Persistent Volumes in the Kubernetes documentation ](https://kubernetes.io/docs/concepts/storage/persistent-volumes/){: external}.

Follow the correct procedure for your deployment.

- [Creating persistent volumes for a production environment](#install-142-create-pvs-prod)
- [Creating persistent volumes for a development environment](#install-142-create-pvs-dev)

### Creating persistent volumes for a production environment
{: #install-142-create-pvs-prod}

To deploy resilient storage in a production environment, the service uses Portworx. Portworx dynamically creates persistent volumes. 

Portworx grabs unmounted disks to use. For example, if you have 7 worker nodes in your cluster, 4 of which have unmounted disks of 400G each, Portworx might provision each worker node with 3 to 4 pods-worth of persistent volumes.

A Portworx license is provided with {{site.data.keyword.icp4dfull_notm}}. The license covers the following features:
  
- Maximum number of nodes: 8
- Maximum number of volumes per cluster: 500
- Maximum volume capacity: 5 TB

The following features are not supported:

- BYOK data encryption
- Snapshot to object store
- Cluster-level migration
- Disaster recovery
- Autopilot capacity management
 
To upgrade the license, go to the [Portworx support site](https://docs.portworx.com/knowledgebase/support.html).

**Red Hat OpenShift 4.3 only**: Alternatively, you can use VMware vSphere volumes, Microsoft Azure Disk volumes, or Amazon Web Services Elastic Block Store (EBS) as a storage solution. For more information, see [Understanding persistent storage](https://docs.openshift.com/container-platform/4.3/storage/understanding-persistent-storage.html){: external}.

1.  You can use the Portworx deployment that is bundled with {{site.data.keyword.icp4dfull_notm}}.

    - **3.0.1**: See [Installing the entitled Portworx instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/portworx-install.html).
    - **2.5**: See [Installing the entitled Portworx instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/install/portworx-install.html).

1.  If you are using Portworx as the storage solution, create the Portworx storage class for {{site.data.keyword.conversationshort}}.

    - 3.0.1: [Creating Portworx storage classes](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/portworx-storage-classes.html){: external}.
    - 2.5: You can run a script that is provided with the service installation files to define the Portworx storage class.

      - Access the Helm chart from the file server at https://github.com/IBM/cloud-pak/tree/master/repo/cpd/modules/ibm-watson-assistant/x86_64/1.4.2/.
      - Unzip the helm chart so you can access the scripts that are provided in the service installation package.
      - On the coordinator node, change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/pre-install** subdirectory of the archive file that you extracted the product files from earlier.
      - Run the **storage.sh** script and specify a single parameter.

        ```bash
        ./clusterAdministration/storage.sh --enablePortworx
        ```
        {: codeblock}

        where `enablePortworx` indicates that you want the script to create the Portworx storage class.

    The following storage class type is defined for the persistent volumes:

    ```yaml
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: portworx-assistant
    provisioner: kubernetes.io/portworx-volume
    parameters:
       repl: "3"
       priority_io: "high"
       io_profile: "db"
       block_size: "64k"
    ```
    {: codeblock}

The storage is ready to be used by your deployment.

#### Using local storage for backups only
{: #install-142-local-for-backup}

A script is provided that you can use to create a local storage persistent volume for storing data backups only.

1.  If you want to use a local storage persistent volume to store backups only, run the `storage.sh` script to create it.

    You must be a cluster administrator to create a local storage volume, and the script used to create it must be run from the coordinator node of the cluster. The coordinator node must have `ssh` access to all of the nodes in your cluster.
    {: important}

    - Access the Helm chart from the file server at https://github.com/IBM/cloud-pak/tree/master/repo/cpd/modules/ibm-watson-assistant/x86_64/1.4.2/.
    - Unzip the helm chart so you can access the scripts that are provided in the service installation package.
    - On the coordinator node, change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/pre-install** subdirectory of the archive file that you extracted the product files from earlier.
    - Run the following command:

      ```bash
      ./clusterAdministration/storage.sh --pgBackupLocalStorage only
      ```
      {: codeblock}

      where `pgBackupLocalStorage` indicates that you want to use a local storage persistent volume only to store backups and will use Portworx for the rest of your storage needs.

### Creating persistent volumes for a development deployment
{: #install-142-create-pvs-dev}

You can use the Portworx storage for a development environment deployment.

If you prefer to use local storage, you can use the **storage.sh** script to create local storage persistent volumes.

Do not use only local storage persistent volumes for a production deployment.
{: important}

When you install the service, persistent volume claims are created for the components automatically. However, when the preferred storage class for the service is **local-storage**, you must explicitly create the persistent volumes in the cluster before you install the service.

Use the **storage.sh** script to create persistent volumes that are bounded to cluster nodes where data stores such as MongoDB and Postgres will run. Volumes that you create with the script are assigned a label that uses the release name. This label is used later to bound each volume to the correct datastore pods.

The `storage.sh` script is included in the Helm chart for the product. The chart can be found in fileserver https://github.com/IBM/cloud-pak/tree/master/repo/cpd/modules/ibm-watson-assistant/x86_64/1.4.2/. Extract the storage.sh file from the Helm chart named `ibm-watson-assistant-prod-1.4.2.tgz`. The script is in the `ibm_cloud_pak/pak_extensions/pre-install/clusterAdministration` directory.

The **storage.sh** script generates a file named `wa-persistence.yaml`. This YAML file adds configuration values that prevent dynamic provisioning from being applied to the volumes. You will reference the content of this YAML file in an override file that you create and pass to the install command later.

The script binds the persistent volumes by using the schema described in the following table.

Table 4. Local storage schema

| Persistent volume type | node1 | node2 | node3 | node4 | node5 |
| -----------------------|-------|-------|-------|-------|-------|
| mongodb  | wa-${RELEASE_NAME}-mongodb-75gi-1  | wa-${RELEASE_NAME}-mongodb-75gi-2  | wa-${RELEASE_NAME}-mongodb-75gi-3  | N/A | N/A |
| etcd     | wa-${RELEASE_NAME}-etcd-10gi-1     | wa-${RELEASE_NAME}-etcd-10gi-2     | wa-${RELEASE_NAME}-etcd-10gi-3     | wa-${RELEASE_NAME}-etcd-10gi-4 | wa-${RELEASE_NAME}-etcd-10gi-5 |
| postgres | wa-${RELEASE_NAME}-postgres-10gi-1 | wa-${RELEASE_NAME}-postgres-10gi-2 | wa-${RELEASE_NAME}-postgres-10gi-3 | N/A | N/A |
| minio    | wa-${RELEASE_NAME}-minio-5gi-1     | wa-${RELEASE_NAME}-mini0-5gi-2     | wa-${RELEASE_NAME}-minio-5gi-3     | wa-${RELEASE_NAME}-minio-5gi-4 | N/A |
| backup   | wa-${RELEASE_NAME}-backup-1gi-1    | N/A | N/A | N/A | N/A |
{: caption="Local persistent volume schema" caption-side="top"}

To create local storage persistent volumes, complete the following steps:

You must be a cluster administrator to create local storage volumes, and the script used to create them must be run from the coordinator node of the cluster. The coordinator node must have `ssh` access to all of the nodes in your development cluster.
{: important}

1.  You will need to choose available worker nodes that can host the persistent volumes. Find out which worker nodes are available to host the volumes by running the following command:

    ```bash
    oc get nodes
    ```
    {: pre}

    From the list of nodes that is returned, choose 5 nodes where you want the persistent volumes to reside. Make a note of the node IP addresses because you will pass them as arguments to the `--nodes` parameter later.

1.  Unzip the helm chart so you can access the scripts that are provided in the service installation package.

1.  On the coordinator node, change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/pre-install** subdirectory.

1.  If you want to see all of the options that are available to you when you create persistent volumes, run the following command:

    ```bash
    ./clusterAdministration/storage.sh --help
    ```
    {: pre}

1.  Run the **storage.sh** script to create the persistent volumes.

    ```bash
    ./clusterAdministration/storage.sh \
    [--pgBackupLocalStorage true | false | only] [--path PATH] \
    [--release RELEASE_NAME]  [--label LABEL] [--nodes node1,node2,node3,node4,node5] \
    [--nodesForDatastore DATASTORE node1,node2,...,node_n] \
    [--tmp DIR] [--force] [--cli oc | kubectl] [--help]
    ```
    {: codeblock}

    - `cli`: The command line interface to use. The options are `oc` and `kubectl`. The default value is `oc`.
    - `force`: Specify this option to force the creation of the data directories. You will be prompted to confirm that you want to delete an existing directory.
    - `label`: Optionally specify the labels to use when selecting the nodes. If not provided, you might be prompted to provide the labels during the installation process. Consider using something like `--label "node-role.kubernetes.io/worker=true"` or `--label "beta.kubernetes.io/arch=amd64"`.
    - `nodes`: Use this parameter to specify which nodes you want the persistent volues to be bound to. Pass as arguments the five node IP addresses, separated by commas, for five of the worker nodes that were returned when you ran the `oc get nodes` command earlier. If you don't provide a value for this parameter, then five nodes are selected randomly from nodes that have the specified label.
    - `nodesForDatastore`: Specify this parameter to override the nodes to be used for a particular datastore. Specify this parameter once for each datastore type you want to customize. This parameter takes two values:
    
       - Datastore for which the nodes are specified. Supported datastores: `mongodb, etcd, postgres, minion, backup`.
       - Comma-separated list of nodes to which the persisten volume for the datastore should be bound. The list must contain at least 3 nodes. Minio requires 4 nodes and etcd requires 5 nodes. For example:
       
       ```
       --nodesForDatastore mongodb node1,node2,node3 --nodesForDatastore etcd node4,node5,node6 --nodesForDatastore postgres node7,node8,node9 --nodesForDatastore minio node10,node11,node12,node13"
       ```
       {:pre}

      You don't need to use different nodes for different datastores. However, do not reuse nodes for a single datastore.

    - `pgBackupLocalStorage`: Specifies whether to create a local-storage persistent volume for Postgres backups. The default value is `true`, a persistent volume for Postgres backups is created. If set to `only`, then a local storage persistent volume is created and used only for storing the Postgres backup. For more information, see [Backing up and restoring data](/docs/assistant-data?topic=assistant-data-backup).
    - `path`: Specify a directory path for the physical location of the volume on the nodes. If you don't, the path `/mnt/local-storage/storage/watson/assistant` is used. The script appends two directories to the path: a ${release-name} and a unique directory name for each volume, such as `mongodb-80gi-1`, to the end of the path you specify.
    - `release`: Specify the release name. The release name must start with an alphabetic character, end with an alphanumeric character, and consist of all lower case alphanumeric characters or a hyphen (-). For example *my-142-wa*. This release name is added to the labels that are used in the `wa-persistence.yaml` file. Be sure to use the same release name when you run the service install command later. If you don't specify a name, a timestamp value is used. The timestamp has the format: `YYYY-MM-DD--HH-mm`.
    - `tmp`: Name of the temporary directory in which to store the persistent volumes. The default value is `/tmp`.

    For example:

    ```bash
    ./clusterAdministration/storage.sh --release my-142-wa --label "node-role.kubernetes.io/worker=true" \
     --nodes 192.18.36.181,192.18.36.182,192.18.36.183,192.18.36.184,192.18.36.185
    ```
    {: codeblock}

    A `wa-persistence.yaml` file is created by the script.

1.  {: #install-142-replace-local-storage-class}Configure {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} to use local storage by editing the `overrides.yaml` file.

    - Override the persistent volume storage class setting. By default, it is set to use `portworx-assistant`. You can specify `local-storage` instead.
  
      This class sets the provisioner to kubernetes.io/portworx-volume. Specifically, you must override the following values:

      - `cos.minio.persistence.storageClass: portworx-assistant`
      - `etcd.config.dataPVC.storageClassName: portworx-assistant`
      - `postgres.config.persistence.storageClassName: portworx-assistant`
      - `mongodb.config.persistentVolume.storageClass: portworx-assistant`
      - `recommendsMongodbLoadEmbeddings.dataPVC.storageClassName: portworx-assistant`

    - Copy the content from the `wa-persistence.yaml` file and paste it into the `overrides.yaml` file.

1.  Use the following command to verify that the persistent volumes were created successfully.

    ```bash
    oc get persistentvolumes -l release=${release-name} --show-labels
    ```
    {: pre}

The resulting list contains information like the information that is displayed in the following table.

Table 5a. Sample persistent volume information

| NAME | CAPACITY | LABELS |
|------|----------|--------|
| wa-my-142-wa-etcd-10gi-1 | 10Gi | dedication=wa-my-142-wa-etcd,release=my-142-wa |
| wa-my-142-wa-etcd-10gi-2 | 10Gi | dedication=wa-my-142-wa-etcd,release=my-142-wa |
| wa-my-142-wa-etcd-10gi-3 | 10Gi | dedication=wa-my-142-wa-etcd,release=my-142-wa |
| wa-my-142-wa-etcd-10gi-4 | 10Gi | dedication=wa-my-142-wa-etcd,release=my-142-wa |
| wa-my-142-wa-etcd-10gi-5 | 10Gi | dedication=wa-my-142-wa-etcd,release=my-142-wa |
| wa-my-142-wa-minio-5gi-1 | 5Gi | dedication=wa-my-142-wa-minio,release=my-142-wa | 
| wa-my-142-wa-minio-5gi-2 | 5Gi | dedication=wa-my-142-wa-minio,release=my-142-wa |
| wa-my-142-wa-minio-5gi-3 | 5Gi | dedication=wa-my-142-wa-minio,release=my-142-wa |
| wa-my-142-wa-minio-5gi-4 | 5Gi | dedication=wa-my-142-wa-minio,release=my-142-wa |
| wa-my-142-wa-mongodb-75gi-1 | 75Gi | dedication=wa-my-142-wa-mongodb,release=my-142-wa |
| wa-my-142-wa-mongodb-75gi-2 | 75Gi | dedication=wa-my-142-wa-mongodb,release=my-142-wa |
| wa-my-142-wa-mongodb-75gi-3 | 75Gi | dedication=wa-my-142-wa-mongodb,release=my-142-wa |
| wa-my-142-wa-postgres-10gi-1 | 10Gi | dedication=wa-my-142-wa-postgres,release=my-142-wa | 
| wa-my-142-wa-postgres-10gi-2 | 10Gi | dedication=wa-my-142-wa-postgres,release=my-142-wa |
| wa-my-142-wa-postgres-10gi-3 | 10Gi | dedication=wa-my-142-wa-postgres,release=my-142-wa |
| wa-my-142-wa-backup-1gi-1 | 1Gi | dedication=wa-my-142-wa-backup,release=my-142-wa |
{: caption="Unique persistent volume information" caption-side="top"}

The following information is also provided, but is the same for all volumes:

Table 5b. More sample persistent volume information

| ACCESS MODES | RECLAIM POLICY | STATUS | STORAGECLASS |
|--------------|----------------|--------|--------------|
| RWO | Retain | Available | local-storage |
{: caption="Static persistent volume information" caption-side="top"}

Be careful if you need to update or stop a node with bounded local-storage persistent volumes to perform maintenance tasks.

## Installation overview
{: #install-142-choose-cluster}

The intallation steps you need to perform differ slightly depending on whether you are installing the service to a 2.5 or 3.0.1 cluster. Follow the appropriate set of instructions for your deployment.

- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.0.1](#install-142-cpd30)
- [Installing on {{site.data.keyword.icp4dfull_notm}} 2.5](#install-142-cpd25)

If you have a previous version of the service installed, you can retain any assistants and skills that you created. Follow the instructions to back up data from the previous version, so that you can restore it in this new version. For more information, see [Backing up and restoring data](/docs/assistant-data?topic=assistant-data-backup).

This documentation refers to the `master` node as the `coordinator` node.
{: note}

## Installing on Cloud Pak for Data 3.0.1
{: #install-142-cpd30}

### Software prerequisites
{: #install-142-cpd30-prereqs}

- {{site.data.keyword.icp4dfull_notm}} 3.0.1
- Kubernetes 1.16.2
- Helm 2.14.3

Before installing {{site.data.keyword.conversationshort}}, you must install and configure [`helm`](https://helm.sh/docs/using_helm/#installing-the-helm-client) and [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl).

Follow these steps to install {{site.data.keyword.conversationshort}} on {{site.data.keyword.icp4dfull_notm}} 3.0.1.

1.  [Purchase and download installation artifacts](#install-142-cpd30-download-wa-cpd)
1.  [Install {{site.data.keyword.icp4dfull_notm}} on OpenShift](#install-142-cpd30-install-icp4d)
1.  [Apply security policy to namespace](#install-142-cpd30-securitypolicy)
1.  [Add required namespace label](#install-142-cpd30-apply-namespace-label)
1.  [Get the Docker secret](#install-142-cpd30-get-secret)
1.  [Create persistent volumes](#install-142-cpd30-pv-step)
1.  [Configure the deployment](#install-142-cpd30-config)
1.  [Install the service](#install-142-cpd30-install-assembly)

### Step 1: Purchase and download installation artifacts
{: #install-142-cpd30-download-wa-cpd}

After you purchase the service, you download the software from GitHub.

1.  Purchase {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} 1.4.2 from [Passport Advantage](https://www.ibm.com/software/passportadvantage/index.html){: external}.

1.  For the instructions to follow to get the installation files, see [Obtaining the installation files](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/installation-files.html).

### Step 2: Install {{site.data.keyword.icp4dfull_notm}} on OpenShift
{: #install-142-cpd30-install-icp4d}

1.  Install {{site.data.keyword.icp4dfull_notm}} on Red Hat OpenShift.

    Follow the instructions for [Installing on OpenShift](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/install.html){:external}.

1.  Enable metrics in OpenShift so that it can support horizontal pod autoscaling in {{site.data.keyword.icp4dfull_notm}}.

    - 3.11: See [Pod Autoscaling](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html){: external}.
    - 4.3: See [Automatically scaling pods](https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-autoscaling.html){: external}.

1.  Review the following topics about cluster security and take steps to implement any security measures that you want to have in place before you install the service:

    - [Security considerations](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/plan/security.html){: external}

      Encryption of data at rest must be managed by the storage provider.
      {: important}

    - [Using a custom TLS certificate for HTTPS connections](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/https-config-openshift.html){: external}

### Step 3: Apply the appropriate security policy
{: #install-142-cpd30-securitypolicy}

A SecurityContextConstraints security policy must be bound to the target namespace before you run the installation. The predefined SecurityContextConstraints name: `restricted` has been verified for this service.
    
1.  Do one of the following things:

    - If your target namespace is bound to the restricted SecurityContextConstraints resource already, skip this step. 
    - Otherwise, run the following command to bind the SecurityContextConstraints to your namespace:

      ```bash
      oc adm policy add-scc-to-group restricted system:serviceaccounts:{namespace-name}
      ```
      {: pre}

      where `{namespace-name}` is the namespace where {{site.data.keyword.conversationshort}} will be installed, which is typically `zen`.

    Use the standard definition of the restricted SCC if you want to create a Custom SecurityContextConstraints definition.

SecurityContextConstraints definition:

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: restr

icted
  annotations:
    kubernetes.io/description: restricted denies access to all host features and requires
      pods to be run with a UID, and SELinux context that are allocated to the namespace.  This
      is the most restrictive SCC and it is used by default for authenticated users.
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegeEscalation: true
allowPrivilegedContainer: false
allowedCapabilities: null
defaultAddCapabilities: null
fsGroup:
  type: MustRunAs
groups:
- system:authenticated
priority: null
readOnlyRootFilesystem: false
requiredDropCapabilities:
- KILL
- MKNOD
- SETUID
- SETGID
runAsUser:
  type: MustRunAsRange
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
users: []
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```
{: codeblock}

### Step 4: Add the cluster namespace label to your service namespace
{: #install-142-cpd30-apply-namespace-label}

The cluster namespace label is needed to permit communication between the namespace of the service and the {{site.data.keyword.icp4dfull_notm}} namespace by using a network policy.

1.  Log in to OpenShift.

    ```
    oc login
    ```
    {: codeblock}

1.  Run the following command to label the namespace:

    ```
    oc label --overwrite namespace {namespace-name} ns={namespace-name}
    ```
    {: codeblock}

    If you get a message that says, for example, `namespace/zen not labeled`, it means the namespace was already labeled.
    {: note}

1.  Make sure you are pointing at the correct OpenShift project.

    ```bash
    oc project {namespace-name}
    ```
    {: pre}

      - `{namespace-name}` is the OpenShift project (and Kubernetes namespace) where {{site.data.keyword.conversationshort}} will be installed. This is typically the same namespace where {{site.data.keyword.icp4dfull_notm}} is installed, such as `zen`.

### Step 5: Get the Docker secret
{: #install-142-cpd30-get-secret}

1. Run the following command to fetch the secret:

    ```
    oc get secrets | grep default-dockercfg
    ```
    {: codeblock}

    The secret typically has the syntax: `default-dockercfg-v5blj`.

### Step 6: Create persistent volumes
{: #install-142-cpd30-pv-step}

For more information, see [Creating persistent volumes](#install-142-create-pvs).

### Step 7: Configure the deployment
{: #install-142-cpd30-config}

1.  Create a YAML file in which to specify configuration settings for your deployment. You can download the [sample overrides.yaml file](https://github.com/IBM/cloud-pak/blob/master/repo/cpd3/modules/ibm-watson-assistant/x86_64/1.4.2/overrides.yaml) from GitHub to use as a starting point.

    At a minimum, you must provide your own values for the following configurable settings:

    - `global.deploymentType`: Specify whether you want to set up a **Development** or **Production** instance. These values are proper case and the setting is case sensitive. **Production** is the default value.
    - `global.languages.{language-name}`: Change the value for an individual language to **true** to enable it. Only English is enabled by default. Additional resources are required to add support for some languages. See [Language considerations](#assistant-install-lang-reqs).

      Be sure to specify all of the languages that you want to support now. You cannot add support for more languages later.
      {: important}

    - Edit or add the Docker image pull secret.

      ```yaml
      global:
        image:
          pullSecret: "{docker-secret}"
      ```
      {: codeblock}

      Use the value that you discovered in Step 5.

If you ran the `storage.sh` script, copy the content from the `wa-persistence.yaml` file that was generated by the script into the `overrides.yaml` file.

### Step 8: Install the service
{: #install-142-cpd30-install-assembly}

1.  Create a `wa-repo.yaml` file.

    Add the following content to the file:

    ```yaml
    registry:
      - url: cp.icr.io/cp/cpd
        username: "cp"
        apikey: {entitlement-key}
        namespace: ""
        name: base-registry
      - url: cp.icr.io
        username: "cp"
        apikey: {entitlement-key}
        namespace: "cp/watson-assistant"
        name: wa-registry
    fileservers:
      - url: https://raw.github.com/IBM/cloud-pak/master/repo/cpd3
    ```
    {: codeblock}

1.  If you don't already know it or have it, get your entitlement key from [myibm.com](https://myibm.ibm.com/products-services/containerlibrary){: external}.

1.  Replace the references to `{entitlement-key}` in the YAML file with the actual key value, and then save and close the file. Store the file in the same directory as the {{site.data.keyword.icp4dfull_notm}} command-line interface.

1.  Complete one of the following steps:

    - To install the service from the local registry:

      - Change to the directory where you placed the {{site.data.keyword.icp4dfull_notm}} command-line interface and the `wa-repo.yaml` file.
      - Log in to your Red Hat OpenShift cluster as a project administrator:

        ```bash
        oc login OpenShift_URL:port
        ```
        {: codeblock}
    
      - Run the following command to install the service assembly:

        ```
        ./cpd-{Operating_System} --repo ./wa-repo.yaml \
        --assembly ibm-watson-assistant \
        --version {assembly_version} \
        --namespace {Project} \
        --transfer-image-to Registry_location \
        --target-registry-username={openshift_username} \
        --target-registry-password={openshift_password} \
        --insecure-skip-tls-verify \
        --cluster-pull-prefix Registry_from_cluster \
        --storageclass {Storage_class_name} \
        --override {Filepath_of_overrides.yaml} \
        ```
        {: codeblock}

        - Replace the `{Operating_System}` in the `cpd-{Operating_System}` command with `linux` for Linux and with `darwin` for Mac OS.
        - The`wa-repo.yaml` file is the file you created earlier.
        - For `{assembly_version}`, specify `1.4.2`.
        - For `Registry_location`, you must specify a route to the registry followed by the namespace (project name). The route must be accessible from the machine where you run the install command. If the cluster you are installing does not have a route to the registry, you can to (temporarily) enable external access to the registries. For more information, see one of the following topics:

          - Red Hat OpenShift 4.3: [Exposing the registry](https://docs.openshift.com/container-platform/4.3/registry/securing-exposing-registry.html){: external}
          - Red Hat OpenShift 3.11: [Securing and exposing the registry](https://docs.openshift.com/container-platform/3.11/install_config/registry/securing_and_exposing_registry.html){: external}

          When you add the `--transfer-image-to` parameter, you can specify the registry location as follows:

          - **OpenShift 4.3**

            ```
            oc get route/default-route -n openshift-image-registry --template='{{ .spec.host }}'
            ```
            {: codeblock}

            The command returns a route similar to `default-route-openshift-image-registry.apps.my_cluster_address`. Append the namespace to the route. For example:

            ```
            default-route-openshift-image-registry.apps.my_cluster_address/zen
            ```
            {: codeblock}

          - **OpenShift 3.11**

            ```
            oc get route/docker-registry -n default --template {{.spec.host}}
            ```
            {: codeblock}

            The command returns a route similar to `docker-registry-default.apps.my_cluster_address`. Append the namespace to the route. For example:

            ```
            docker-registry-default.apps.my_cluster_address/zen
            ```
            {: codeblock}

        - Provide the username and password for a user with access to the registry in the `target-registry-username` and `target-registry-password` parameters. This name must be the same name that you used when you ran the `oc login` command. The default username is typically `kubeadmin` for OpenShift 4.x and `ocadmin` for OpenShift 3.x. If you specify `$(oc whoami -t)` as the password, the corresponding password is populated for you.
        - If you are using the internal Red Hat OpenShift registry and you are using the default self-signed certificate, specify the `--insecure-skip-tls-verify` flag to prevent x509 errors.
        - `Registry_from_cluster`: Address of the internal OpenShift Docker registry. For OpenShift 4.x, it is typically, `image-registry.openshift-image-registry.svc:5000`. For OpenShift 3.x, it is typically, `docker-registry.default.svc:5000`.
        - Specify the `--storageclass` parameter only if you're using a storage class other than `portworx-assistant`.
     
        For example:

        ```
        ./cpd-linux --repo ./wa-repo.yaml --assembly ibm-watson-assistant \
         --version 1.4.2 --namespace zen \
         --transfer-image-to docker-registry-default.9.87.654.321.nip.io/zen --target-registry-username=kubeadmin \
         --target-registry-password=$(oc whoami -t) --insecure-skip-tls-verify \
        --cluster-pull-prefix image-registry.openshift-image-registry.svc:5000/zen \
        --override overrides.yaml
        ```
        {: codeblock}

    - To install the service on an air-gapped cluster, complete these steps:

      - Run the following command in a location with access to internet and the cpd-linux tool to download the images and assembly files:

        ```
        ./cpd-linux preloadImages --repo wa-repo.yaml \
        --assembly ibm-watson-assistant --version 1.4.2 \
        --action download --download-path ./wa-workspace
        ```
        {: codeblock}

      - Push the `wa-workspace` folder to a location with access to the OpenShift cluster to be installed and to its internal image registry. The same version of the cpd-linux tool must be used as was used in the previous step.

      - Login to the Openshift cluster

        ```
        oc login
        ```
        {: codeblock}

      - Push the Docker images to the internal docker registry.

        ```
        ./cpd-linux preloadImages --action push \
        --load-from ./wa-workspace \
        --assembly ibm-watson-assistant --version 1.4.2 \
        --transfer-image-to Registry_location \
        --target-registry-username={openshift_username} \
        --target-registry-password={openshift_password} \
        --insecure-skip-tls-verify
        ```
        {: codeblock}

        - For `Registry_location`, you must specify a route to the registry followed by the namespace. The route must be accessible from the machine where you run the install command. If the cluster you are installing does not have a route to the registry, you can to (temporarily) enable external access to the registries. For more information, see one of the following topics:

          - Red Hat OpenShift 4.3: [Exposing the registry](https://docs.openshift.com/container-platform/4.3/registry/securing-exposing-registry.html){: external}
          - Red Hat OpenShift 3.11: [Securing and exposing the registry](https://docs.openshift.com/container-platform/3.11/install_config/registry/securing_and_exposing_registry.html){: external}

          When you add the `--transfer-image-to` parameter, you can specify the registry location as follows:

          - **OpenShift 4.3**

            ```
            oc get route/default-route -n openshift-image-registry --template='{{ .spec.host }}'
            ```
            {: codeblock}

            The command returns a route similar to `default-route-openshift-image-registry.apps.my_cluster_address`. Append the namespace to the route. For example:

            ```
            default-route-openshift-image-registry.apps.my_cluster_address/zen
            ```
            {: codeblock}

          - **OpenShift 3.11**

            ```
            oc get route/docker-registry -n default --template {{.spec.host}}
            ```
            {: codeblock}

            The command returns a route similar to `docker-registry-default.apps.my_cluster_address`. Append the namespace to the route. For example:

            ```
            docker-registry-default.apps.my_cluster_address/zen
            ```
            {: codeblock}

        - Provide the username and password for a user with access to the registry in the `target-registry-username` and `target-registry-password` parameters. This name must be the same name that you used when you ran the `oc login` command. The default username is typically `kubeadmin` for OpenShift 4.x and `ocadmin` for OpenShift 3.x. If you specify `$(oc whoami -t)` as the password, the corresponding password is populated for you.
        - If you are using the internal Red Hat OpenShift registry and you are using the default self-signed certificate, specify the `--insecure-skip-tls-verify` flag to prevent x509 errors.

        For example:

        ```
        ./cpd-linux preloadImages --action push \
        --load-from ./wa-workspace --assembly ibm-watson-assistant \ --version 1.4.2 --transfer-image-to $(oc registry info)/zen \
        --target-registry-username {openshift_username} \
        --target-registry-password $(oc whoami -t) \
        --insecure-skip-tls-verify
        ```
        {: codeblock}

        For more information, see [Preparing for air-gapped intallations](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/ag-install-prep.html){: external}

      - Run the following command to install the service:

        ```
        ./cpd-{Operating_System} \
        --assembly ibm-watson-assistant \
        --version Assembly_version \
        --namespace Project \
        --storageclass Storage_class_name \
        --override Filepath_of_overrides.yaml \
        --cluster-pull-prefix Registry_from_cluster \
        --ask-push-registry-credentials \
        --load-from Image_directory_location
        ```
        {: codeblock}

        - Replace the `{Operating_System}` in the `cpd-{Operating_System}` command with `linux` for Linux and with `darwin` for Mac OS.
        - For `Assembly_version`, specify `1.4.2`.
        - Specify the `--storageclass` parameter only if you're using a storage class other than `portworx-assistant`.
        - For `Image_directory_location`, specify the `./wa-workspace` directory.
        - If you are using the internal Red Hat OpenShift registry, do not specify the `--ask-pull-registry-credentials` parameter.

        For example:

        ```
        ./cpd-linux --load-from ./wa-workspace \
        --assembly ibm-watson-assistant --version 1.4.2 --namespace zen \
        --cluster-pull-prefix image-registry.openshift-image-registry.svc:5000/zen \
        --override overrides.yaml
        ```
        {: codeblock}

    For a list of all available options, enter the command: `./cpd-{Operating_System} --help`.
    {: tip}

1.  **For deployments that don't use Portworx**: This step is required if you are using a storage solution other than Portworx, such as VMware vSphere volumes, Microsoft Azure Disk volumes, or Amazon Web Services Elastic Block Store (EBS). You must run the backup cron job as part of the installation process.

    The installation process waits until all PVCs have been bound before it completes. However, if you use a storage solution other than Portworx, the PVC for the Postgres backup won't become bound until after the Postgres backup cron job runs for the first time. To prevent the installation from having to wait for the job or timing out, start the cron job manually.

    - Check the status of the installation. Do not run the cron job until after the store pod is running. You can check the status of the store pod by using the following command:

      ```
      oc get pods -l release=watson-assistant,component=store
      ```
      {: codeblock}

    - Run the cron job:

      ```
      oc create job --from=cronjob/watson-assistant-backup-cronjob 
      watson-assistant-backup-cronjob-first-run -n {namespace-name}
      ```
      {: codeblock}

    After the job creates a pod, the PVC becomes bound and the installation can finish. After the installation is complete, you can delete the job you created.

    For more information about using cron jobs, see [Backing up and restoring data](/docs/assistant-data?topic=assistant-data-backup).

## Installing on an Cloud Pak for Data 2.5
{: #install-142-cpd25}

### Software prerequisites
{: #install-142-cpd25-prereqs}

- {{site.data.keyword.icp4dfull_notm}} 2.5 with Red Hat OpenShift 3.11
- Kubernetes 1.11.0
- Helm 2.14.3

Before installing {{site.data.keyword.conversationshort}}, you must install and configure [`helm`](https://helm.sh/docs/using_helm/#installing-the-helm-client) and [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl).

Follow these steps to install {{site.data.keyword.conversationshort}} on {{site.data.keyword.icp4dfull_notm}} 2.5.

1.  [Purchase and download installation artifacts](#install-142-cpd25-download-wa-cpd)
1.  [Install {{site.data.keyword.icp4dfull_notm}} on OpenShift](#install-142-cpd25-install-icp4d)
1.  [Apply security policy to namespace](#install-142-cpd25-securitypolicy)
1.  [Add required namespace label](#install-142-cpd25apply-namespace-label)
1.  [Get the Docker secret](#install-142-cpd25-get-secret)
1.  [Create persistent volumes](#install-142-cpd25-pv-step)
1.  [Configure the deployment](#install-142-cpd25-config)
1.  [Install the service](#install-142-cpd25-install-assembly)

### Step 1: Purchase and download installation artifacts
{: #install-142-cpd25-download-wa-cpd}

After you purchase the service, you download the software from GitHub.

1.  Purchase {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} 1.4.2 from [Passport Advantage](https://www.ibm.com/software/passportadvantage/index.html){: external}.

1. For the instructions to follow to get the installation files, see [Obtaining the installation files](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/install/installation-files.html).

### Step 2: Install {{site.data.keyword.icp4dfull_notm}} on OpenShift
{: #install-142-cpd25-install-icp4d}

1.  Install {{site.data.keyword.icp4dfull_notm}} on Red Hat OpenShift.

    Follow the instructions for [Installing on OpenShift](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/install/rhos-install.html){:external}.

1.  Enable metrics in OpenShift so that it can support horizontal pod autoscaling in {{site.data.keyword.icp4dfull_notm}}.

    - 3.11: See [Pod Autoscaling](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html){: external}.

1.  Review the following topics about cluster security and take steps to implement any security measures that you want to have in place before you install the service:

    - [Security considerations](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/plan/security.html){: external}

      Encryption of data at rest must be managed by the storage provider.
      {: important}

    - [Use a custom TLS certificate on OpenShift](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/install/https-config-openshift.html){: external}

### Step 3: Apply the appropriate security policy
{: #install-142-cpd25-securitypolicy}

A SecurityContextConstraints security policy must be bound to the target namespace before you run the installation. The predefined SecurityContextConstraints name: `restricted` has been verified for this service.
    
1.  Do one of the following things:

    - If your target namespace is bound to the restricted SecurityContextConstraints resource already, skip this step. 
    - Otherwise, run the following command to bind the SecurityContextConstraints to your namespace:

      ```bash
      oc adm policy add-scc-to-group restricted system:serviceaccounts:{namespace-name}
      ```
      {: pre}

      where `{namespace-name}` is the namespace where {{site.data.keyword.conversationshort}} will be installed, which is typically `zen`.

    Use the standard definition of the restricted SCC if you want to create a Custom SecurityContextConstraints definition.

SecurityContextConstraints definition:

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: restr

icted
  annotations:
    kubernetes.io/description: restricted denies access to all host features and requires
      pods to be run with a UID, and SELinux context that are allocated to the namespace.  This
      is the most restrictive SCC and it is used by default for authenticated users.
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegeEscalation: true
allowPrivilegedContainer: false
allowedCapabilities: null
defaultAddCapabilities: null
fsGroup:
  type: MustRunAs
groups:
- system:authenticated
priority: null
readOnlyRootFilesystem: false
requiredDropCapabilities:
- KILL
- MKNOD
- SETUID
- SETGID
runAsUser:
  type: MustRunAsRange
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
users: []
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```
{: codeblock}

### Step 4: Add the cluster namespace label to your service namespace
{: #install-142-cpd25-apply-namespace-label}

The cluster namespace label is needed to permit communication between the namespace of the service and the {{site.data.keyword.icp4dfull_notm}} namespace by using a network policy.

1.  Log in to OpenShift.

    ```bash
    oc login
    ```
    {: codeblock}

1.  Run the following command to label the namespace:

    ```bash
    oc label --overwrite namespace {namespace-name} ns={namespace-name}
    ```
    {: codeblock}

    If you get a message that says, for example, `namespace/zen not labeled`, it means the namespace was already labeled.
    {: note}

1.  Make sure you are pointing at the correct OpenShift project.

    ```bash
    oc project {namespace-name}
    ```
    {: pre}

    - `{namespace-name}` is the OpenShift project (and Kubernetes namespace) where {{site.data.keyword.conversationshort}} will be installed. This is typically the same namespace where {{site.data.keyword.icp4dfull_notm}} is installed, such as `zen`.

### Step 5: Get the Docker secret
{: #install-142-cpd25-get-secret}

1. Run the following command to fetch the secret:

    ```
    oc get secrets | grep default-dockercfg
    ```
    {: codeblock}

    The secret typically has the syntax: `default-dockercfg-v5blj`.

### Step 6: Create persistent volumes
{: #install-142-cpd25-pv-step}

For more information, see [Creating persistent volumes](#install-142-create-pvs).

### Step 7: Configure the deployment
{: #install-142-cpd25-config}

1.  Create a YAML file in which to specify configuration settings for your deployment. You can download the [sample overrides.yaml file](https://github.com/IBM/cloud-pak/blob/master/repo/cpd/modules/ibm-watson-assistant/x86_64/1.4.2/overrides.yaml) from GitHub to use as a starting point.

    At a minimum, you must provide your own values for the following configurable settings:

    - `global.deploymentType`: Specify whether you want to set up a **Development** or **Production** instance. These values are proper case and the setting is case sensitive. **Production** is the default value.
    - `global.languages.{language-name}`: Change the value for an individual language to **true** to enable it. Only English is enabled by default. Additional resources are required to add support for some languages. See [Language considerations](#assistant-install-lang-reqs).

      Be sure to specify all of the languages that you want to support now. You cannot add support for more languages later.
      {: important}
    
    - Edit or add the Docker image pull secret.

      ```yaml
      global:
        image:
          pullSecret: "{docker-secret}"
      ```
      {: codeblock}

      Use the value that you discovered in Step 5.

If you ran the `storage.sh` script, copy the content from the `wa-persistence.yaml` file that was generated by the script into the `overrides.yaml` file.

### Step 8: Install the service
{: #install-142-cpd25-install-assembly}

1.  Create a `wa-repo.yaml` file.

    Add the following content to the file:

    ```yaml
    registry:
      - url: cp.icr.io/cp/cpd
        username: "cp"
        apikey: {entitlement-key}
        namespace: ""
        name: base-registry
      - url: cp.icr.io
        username: "cp"
        apikey: {entitlement-key}
        namespace: "cp/watson-assistant"
        name: wa-registry
    fileservers:
      - url: https://raw.github.com/IBM/cloud-pak/master/repo/cpd
    ```
    {: codeblock}

1.  If you don't already know it or have it, get your entitlement key from [myibm.com](https://myibm.ibm.com/products-services/containerlibrary){: external}.

1.  Replace the references to `{entitlement-key}` in the YAML file with the actual key value, and then save and close the file. Store the file in the same directory as the {{site.data.keyword.icp4dfull_notm}} command-line interface.

1.  Complete one of the following steps:

    - To install the service from the local registry:

      - Change to the directory where you placed the {{site.data.keyword.icp4dfull_notm}} command-line interface and the `wa-repo.yaml` file.
      - Log in to your Red Hat OpenShift cluster as a project administrator:

        ```bash
        oc login OpenShift_URL:port
        ```
        {: codeblock}
    
      - Run the following command to install the service assembly:

        ```
        ./cpd-{Operating_System} --repo ./wa-repo.yaml \
        --assembly ibm-watson-assistant \
        --version {assembly_version} \
        --namespace {Project} \
        --transfer-image-to Registry_location \
        --target-registry-username={openshift_username} \
        --target-registry-password=$(oc whoami -t) \
        --insecure-skip-tls-verify \
        --cluster-pull-prefix Registry_from_cluster \
        --storageclass {Storage_class_name} \
        --override {Filepath_of_overrides.yaml} \
        ```
        {: codeblock}

        - Replace the `{Operating_System}` in the `cpd-{Operating_System}` command with `linux` for Linux and with `darwin` for Mac OS.
        - The`wa-repo.yaml` file is the file you created earlier.
        - For `{assembly_version}`, specify `1.4.2`.
        - For `Registry_location`, you must specify a route to the registry followed by the namespace. The route must be accessible from the machine where you run the install command. If the cluster you are installing does not have a route to the registry, you can to (temporarily) enable external access to the registries. For more information, see one of the following topics:

          - Red Hat OpenShift 4.3: [Exposing the registry](https://docs.openshift.com/container-platform/4.3/registry/securing-exposing-registry.html){: external}
          - Red Hat OpenShift 3.11: [Securing and exposing the registry](https://docs.openshift.com/container-platform/3.11/install_config/registry/securing_and_exposing_registry.html){: external}

          For example, for OpenShift 4.3:

          ```
          export REGISTRY_ROUTE=`oc get route default-route -n openshift-image-registry | grep registry | awk {'print $2'}
          ```
          {: codeblock}

          For example, for OpenShift 3.11:

          ```
          export REGISTRY_ROUTE=`oc get route docker-registry -n default | grep registry | awk {'print $2'}`
          ```
          {: codeblock}

          When you add the `--transfer-image-to` parameter, you can specify `${REGISTRY_ROUTE}/{namespace}`.
        - Provide the username and password for a user with access to the registry in the `target-registry-username` and `target-registry-password` parameters. The default username is `ocadmin` for OpenShift 3.x. If you specify `$(oc whoami -t)` as the password, the corresponding password is populated for you.
        - If you are using the internal Red Hat OpenShift registry and you are using the default self-signed certificate, specify the `--insecure-skip-tls-verify` flag to prevent x509 errors.
        - `Registry_from_cluster`: Address of the internal OpenShift Docker registry. For OpenShift 3.x, it is typically, `docker-registry.default.svc:5000`.
        - Specify the `--storageclass` parameter only if you're using a storage class other than `portworx-assistant`.
     
        For example:

        ```
        ./cpd-linux --repo ./wa-repo.yaml --assembly ibm-watson-assistant \
        --version 1.4.2 --namespace zen \
        --transfer-image-to docker-registry-default.9.87.654.321.nip.io/zen --target-registry-username=ocadmin \
        --target-registry-password=xxx --insecure-skip-tls-verify \
        --cluster-pull-prefix docker-registry.default.svc:500/zen \
        --override overrides.yaml
        ```
        {: codeblock}

    - To install the service on an air-gapped cluster, complete these steps:

      - Run the following command in a location with access to internet and the cpd-linux tool to download the images and assembly files:

        ```
        ./cpd-linux preloadImages --repo wa-repo.yaml \
        --assembly ibm-watson-assistant --version 1.4.2 \
        --action download --download-path ./wa-workspace
        ```
        {: codeblock}

      - Push the `wa-workspace` folder to a location with access to the OpenShift cluster to be installed and to its internal image registry. The same version of the cpd-linux tool must be used as was used in the previous step.

      - Login to the Openshift cluster

        ```
        oc login
        ```
        {: codeblock}

      - Push the Docker images to the internal docker registry.

        ```
        ./cpd-linux preloadImages --action push \
        --load-from ./wa-workspace \
        --assembly ibm-watson-assistant --version 1.4.2 \
        --transfer-image-to Registry_location \
        --target-registry-username={openshift_username} \
        --target-registry-password={openshift_password} \
        --insecure-skip-tls-verify
        ```
        {: codeblock}

        - For `Registry_location`, you must specify a route to the registry followed by the namespace. The route must be accessible from the machine where you run the install command. If the cluster you are installing does not have a route to the registry, you can to (temporarily) enable external access to the registries. For more information, see one of the following topics:

          - Red Hat OpenShift 4.3: [Exposing the registry](https://docs.openshift.com/container-platform/4.3/registry/securing-exposing-registry.html){: external}
          - Red Hat OpenShift 3.11: [Securing and exposing the registry](https://docs.openshift.com/container-platform/3.11/install_config/registry/securing_and_exposing_registry.html){: external}

          For example, for OpenShift 4.3:

          ```
          export REGISTRY_ROUTE=`oc get route default-route -n openshift-image-registry | grep registry | awk {'print $2'}
          ```
          {: codeblock}

          For example, for OpenShift 3.11:

          ```
          export REGISTRY_ROUTE=`oc get route docker-registry -n default | grep registry | awk {'print $2'}`
          ```
          {: codeblock}

          When you add the `--transfer-image-to` parameter, you can specify `${REGISTRY_ROUTE}/{namespace}`.
        - Provide the username and password for a user with access to the registry in the `target-registry-username` and `target-registry-password` parameters. This name must be the same name that you used when you ran the `oc login` command. The default username is typically `ocadmin` for OpenShift 3.x. If you specify `$(oc whoami -t)` as the password, the corresponding password is populated for you.
        - If you are using the internal Red Hat OpenShift registry and you are using the default self-signed certificate, specify the `--insecure-skip-tls-verify` flag to prevent x509 errors.

        For example:

        ```
        ./cpd-linux preloadImages --action push \
        --load-from ./wa-workspace --assembly ibm-watson-assistant \ --version 1.4.2 --transfer-image-to $(oc registry info)/zen \
        --target-registry-username {openshift_username} \
        --target-registry-password $(oc whoami -t) \
        --insecure-skip-tls-verify
        ```
        {: codeblock}

        For more information, see [Preparing for air-gapped intallations](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/install/ag-install-prep.html){: external}

      - Run the following command to install the service:

        ```
        ./cpd-{Operating_System} \
        --assembly ibm-watson-assistant \
        --version Assembly_version \
        --namespace Project \
        --storageclass Storage_class_name \
        --override Filepath_of_overrides.yaml \
        --cluster-pull-prefix Registry_from_cluster \
        --ask-push-registry-credentials \
        --load-from Image_directory_location
        ```
        {: codeblock}

        - Replace the `{Operating_System}` in the `cpd-{Operating_System}` command with `linux` for Linux and with `darwin` for Mac OS.
        - For `Assembly_version`, specify `1.4.2`.
        - Specify the `--storageclass` parameter only if you're using a storage class other than `portworx-assistant`.
        - For `Image_directory_location`, specify the `./wa-workspace` directory.
        - If you are using the internal Red Hat OpenShift registry, do not specify the `--ask-pull-registry-credentials` parameter.

        For example:

        ```
        ./cpd-linux --load-from ./wa-workspace \
        --assembly ibm-watson-assistant --version 1.4.2 --namespace zen \
        --cluster-pull-prefix image-registry.openshift-image-registry.svc:5000/zen \
        --override overrides.yaml
        ```
        {: codeblock}

For a list of all available options, enter the command: `./cpd-{Operating_System} --help`.
{: tip}

## Verifying that the installation was successful
{: #install-142-verify}

To check the status of the installation process:

1.  Check the status of the assembly and modules

      ```bash
      ./cpd-linux status --namespace {namespace} --assembly ibm-watson-assistant
      ```
      {: pre}

      - `{namespace}`: Namespace where {{site.data.keyword.icp4dfull_notm}} was installed, which is typically `zen`.

1.  Set up your Helm environment

    ```bash
    export TILLER_NAMESPACE={namespace}
    oc get secret helm-secret -n $TILLER_NAMESPACE -o yaml|grep -A3 '^data:'|tail -3 | awk -F: '{system("echo "$2" |base64 --decode > "$1)}'
    export HELM_TLS_CA_CERT=$PWD/ca.cert.pem
    export HELM_TLS_CERT=$PWD/helm.cert.pem
    export HELM_TLS_KEY=$PWD/helm.key.pem
    helm version --tls
    ```
    {: pre}

    The output should look like this:

    ```bash
    Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
    Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
    ```
    {: codeblock}

1.  Check the status of the resources.

    ```bash
    helm status watson-assistant --tls
    ```
    {: pre}

1.  Run Helm tests.

    ```bash
    helm test watson-assistant --tls --timeout=18000 [--cleanup]
    ```
    {: pre}

    - `--timeout={time}`: waits for the time in seconds for the tests to run
    - `--cleanup`: deletes test pods upon completion

## Checking for available patches
{: #install-142-patches}

From a cluster that can connect to the internet, run the following command to check for available patches:

    ```bash
    ./cpd-Operating_System --repo ./wa-repo.yaml status \
    --namespace Project \ 
    --assembly ibm-watson-assistant \
    --patches
    ```
    {: pre}

    For example:

    ```bash
    ./cpd-linux --repo wa-repo.yaml status --namespace zen \
    --assembly ibm-watson-assistant --patches
    ```
    {: pre}

To check for patches on an air-gapped cluster, see the list of [available patches for Watson Assistant](https://www.ibm.com/support/pages/node/6240164).

### Uninstalling the service
{: #install-142-uninstall}

If you need to start the deployment over, be sure to remove all trace of the current installation before you try to install again.

If you need to preserve any data, do so now before you begin this procedure.
{: important}

1.  {: #install-142-delete-instance-sh}If you got as far as creating one or more instances of the service, then complete the following steps to delete the instances. Otherwise, skip this step.

    1. From the main menu of the {{site.data.keyword.icp4dfull_notm}} web client, go to the **My Instances** page, and then click the **Provisioned instances** tab.

    1. For each {{site.data.keyword.conversationshort}} instance that you provisioned, click the More menu, and then choose **Delete**.

    ![Shows the only menu location where you can delete the instance.](images/delete-instance.png)

    Currently, you cannot delete an instance by clicking the *Delete Instance* button from the details page for the deployment or from the details page for an individual instance.
    {: important}

1.  If you created local storage persistent volumes: Before you delete anything else, get a list of the names of the persistent volumes that you created for this deployment.

    ```bash
    oc get pv
    ```
    {: pre}

    Make a note of the name of each persistent volume. You will need this information in a later step.

1.  Delete the deployment by using the following command:

    ```bash
    ./cpd-linux uninstall --assembly ibm-watson-assistant --namespace {namespace}
    ```
    {: pre}

1.  Delete any associated artifacts that are left over, such as service data stores that are intentionally preserved, by using the following command. 

    Reminder: This command also removes the data stores you created, which must be removed and recreated if you need to start the installation over.
    {: note}

    ```bash
    oc delete job,deploy,replicaset,pod,statefulset,configmap,secret,ingress,service,serviceaccount,role,rolebinding,persistentvolumeclaim,poddisruptionbudget,horizontalpodautoscaler,networkpolicies,cronjob -l release={release-name}
    ```
    {: pre}

    The `-l` for label is the selector to filter by, where you can specify the release name.

1.  Remove the configmap by using the following command.

    ```bash
    oc delete cm stolon-cluster-{release-name}
    ```
    {: pre}

1.  If you created local storage persistent volumes: You must delete each physical persistent volume. Use the following command to do so.

    You should have a list of the names of each persistent volume that you noted in an earlier step.

    ```bash
    oc delete pv (pv-name}
    ```
    {: pre}

Now, you have cleared everything necessary to restart your installation. Your namespace exists and your service archive file is loaded into it, so you can pick up the installation instructions starting from the step in which you create persistent volumes.

## Provisioning an instance of the service
{: #install-142-install-service}

You can provision up to 30 instances of {{site.data.keyword.conversationshort}} per deployment of Watson Assistant.

1.  From the {{site.data.keyword.icp4dfull_notm}} web client, go to the *Services* page. 

    ![Services icon](images/cp4d-services-icon.png)

1.  Find the {{site.data.keyword.conversationshort}} service tile, and then click it.

    The tile shows that the service is *Available* only if the service was added to the cluster by following the installation steps that are described earlier.

1.  Click the *More* icon ![kebab icon](images/cp4d-vertical-kebab.png), and then click **Provision instance**.

1.  Name the instance. 

    This is the instance you will share with the users in your organization. They will see this instance name from the product's main page. Choose a unique name that represents the instance's purpose.

1.  Click **Create**.

If you plan to restore service instances that you backed up from a deployment of an earlier version of the service, recreate the same number of service instances as you plan to restore. For more information, see [Backing up and restoring data](/docs/assistant-data?topic=assistant-data-backup).

## Launching the product
{: #install-142-launch-tool}

1.  Open an incognito window in your web browser to prevent cached information from being used by the product.
1.  From the main {{site.data.keyword.icp4dfull_notm}} web client navigation menu, select **My instances**.
1.  On the **Provisioned instances** tab, find your {{site.data.keyword.conversationshort}} instance, and double-click it to open it.
1.  Click **Launch tool**.

## Next steps
{: #install-142-next-steps}

Use the {{site.data.keyword.conversationshort}} product user interface to build training data and a dialog that can be used by your assistant.

- To learn more about the service first, read the [overview](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-index).
- To see how it works for yourself, follow the steps in the [getting started tutorial](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-getting-started).
- For help managing the cluster, see [Managing the cluster](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-manage).

## Troubleshooting issues
{: #install-142-ts-get-logs}

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

### Cannot provision an instance, and service images are missing from the catalog
{: #install-142-missing-label}

If you run the installation with no errors, but cannot provision an instance, check whether the product icon is visible in the service tile. From the {{site.data.keyword.icp4dfull_notm}} web client, go to the *Services* page. 

    ![Services icon](images/cp4d-services-icon.png)

1.  Find the {{site.data.keyword.conversationshort}} service tile. Check whether the product logo (![Watson Assistant logo](images/assistant-icon.png)) is displayed on the tile. 

    ![Watson Assistant service tile](images/missing-icon.png)

1.  If the logo is missing, it is likely that you missed the step in the installation process where you label the namespace.

    - **3.0.1**: See [Step 4: Add the cluster namespace label to your service namespace](#install-142-cpd30-apply-namespace-label).
    - **2.5**: See: [Step 4: Add the cluster namespace label to your service namespace](#install-142-cpd25-apply-namespace-label).

### To check the configuration
{: #install-142-check-config}

If you want to check what configuration settings were applied to a deployment when it was set up, you can run the following command:

```bash
helm get <release-name> --tls [--tiller-namespace {namespace-name}]
```
{: pre}

The user-provided configuration values are listed at the start of the information that is returned.

