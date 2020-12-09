---

copyright:
  years: 2015, 2020
lastupdated: "2020-12-09"

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

For installation instructions, see the {{site.data.keyword.icp4dfull_notm}} knowledge center:

- [Installing on {{site.data.keyword.icp4dfull_notm}} 3.5](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/svc-assistant/assistant-install.html){: external} 

## Testing the installation
{: #install-150-test}

After installing the service, run a verification test to make sure that things are working properly.

1.  Create a yaml file with a name such as `test-override.yaml` that looks as follows.

    ```yaml
    apiVersion: com.ibm.watson.watson-assistant/v1
    kind: WatsonAssistantDvt
    metadata:
      name: watson-assistant-${instance name}
      annotations:
        oppy.ibm.com/disable-rollback: "true"
    spec:
    #Additional labels to pass down to all objects created by the operator
    labels:
    "app.kubernetes.io/instance": "watson-assistant-${instance name}"
    
    version: develop
    #The name of the WA instance to target with this DVT run
    assistantInstanceName: watson-assistant-${instance name}
    #The test tags to execute ( change @accuracy to @accuracyOS311 if you are running on OpenShift 3.11)
    testTags: "@accuracy,@dialogErrors,@dialogs,@dialogV1,@dialogV1errors,@embedsearch,@entities,@folders,@fuzzy,@generic,@healthcheck,@intents,@newse,@openentities,@patterns,@prebuilt,@search,@slots,@spellcheck,@spellcheckfr,@v2assistants,@v2authorskill,@v2authorwksp,@v2healthcheck,@v2skillref,@v2snapshots,@workspaces"
    
    #Information specific to this cluster
    cluster:
      type: "private"
      # :image_pull_secrets: pull secret names
      imagePullSecrets: []
    
      #docker_registry_prefix: (private only) Docker registry, including namespace, to get images from
      dockerRegistryPrefix: "${registry info}"
    
    appConfigOverrides:
      container_images_defaults:
        registry: stg.icr.io

    ```
    {: codeblock}

1.  Replace the following values in the YAML file:

    - `${instance name}`: Instance name, such as `wa001`.
    - `${registry info}`: Docker registry, which is typically different depending on the version of OpenShift you're using.

      - **4.5**: `"image-registry.openshift-image-registry.svc:5000/${namespace}"`
      - **3.11**: `"docker-registry.default.svc:5000/${namespace}"`

      where you replace `${namespace}` with the namespace, such as `zen`. 

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

## Creating persistent volumes
{: #install-150-pvc}

A PersistentVolume (PV) is a unit of storage in the cluster. In the same way that a node is a cluster resource, a persistent volume is also a resource in the cluster.

For more information, see [Persistent Volumes in the Kubernetes documentation ](https://kubernetes.io/docs/concepts/storage/persistent-volumes/){: external}.

### Storage requirements
{: #install-150-storage-reqs}

The following table lists the storage resources that are required to support the persistent volumes that are used by the service.

Table 1. Storage requirements

| Component | Number of replicas | Space per pod |
|-----------|--------------------|---------------|
| Postgres  | 3 | 5 GB |
| etcd      | 5 | 1 GB |
| Kafka     | 3 | 2 GB |
| Minio     | 4 | 5 GB |
| Backup    | 1 | 1 GB |
{: caption="Storage details" caption-side="top"}

The systems that host {{site.data.keyword.conversationshort}} must meet these additional requirements:

- {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} can run on Intel architecture nodes only.
- CPUs must have 2.4 GHz or higher clock speed
- CPUs must support Linux SSE 4.2
- CPUs must support the AVX2 instruction set extension. See the [Advanced Vector Extensions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2){: external} Wikipedia page for a list of CPUs that include this support (most CPUs since 2013). The service cannot function properly without AVX2 support.

## Creating the volumes
{: #install-150-create-pvs}

Follow the correct procedure for your deployment.

- [Creating persistent volumes for a production environment](#install-150-create-pvs-prod)
- [Creating persistent volumes for a development environment](#install-150-create-pvs-dev)

### Creating persistent volumes for a production environment
{: #install-150-create-pvs-prod}

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

**Red Hat OpenShift 4.5 only**: Alternatively, you can use VMware vSphere volumes, Microsoft Azure Disk volumes, or Amazon Web Services Elastic Block Store (EBS) as a storage solution. For more information, see [Understanding persistent storage](https://docs.openshift.com/container-platform/4.5/storage/understanding-persistent-storage.html){: external}.

1.  You can use the Portworx deployment that is bundled with {{site.data.keyword.icp4dfull_notm}}.

    - **3.5.0**: See [Installing the entitled Portworx instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install/portworx-install.html){: external}.
    - **3.0.1**: See [Installing the entitled Portworx instance](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/install/portworx-install.html){: external}.

The storage is ready to be used by your deployment.

### Using local storage for backups only
{: #install-150-local-for-backup}

A script is provided that you can use to create a local storage persistent volume for storing data backups only.

1.  If you want to use a local storage persistent volume to store backups only, run the `storage.sh` script to create it.

    You must be a cluster administrator to create a local storage volume, and the script used to create it must be run from the coordinator node of the cluster. The coordinator node must have `ssh` access to all of the nodes in your cluster.
    {: important}

    - Download the `storage.sh` script from GitHub.
    - Run the following command:

      ```bash
      ./clusterAdministration/storage.sh --pgBackupLocalStorage only
      ```
      {: codeblock}

      where `pgBackupLocalStorage` indicates that you want to use a local storage persistent volume only to store backups and will use Portworx for the rest of your storage needs.

### Creating persistent volumes for a development deployment
{: #install-150-create-pvs-dev}

You can use the Portworx storage for a development environment deployment.

If you prefer to use local storage, you can use the **storage.sh** script to create local storage persistent volumes.

Do not use local storage persistent volumes in a production deployment for anything other than data backup storage.
{: important}

When you install the service, persistent volume claims are created for the components automatically. However, when the preferred storage class for the service is **local-storage**, you must explicitly create the persistent volumes in the cluster before you install the service.

Use the **storage.sh** script to create persistent volumes that are bounded to cluster nodes where data stores such as MongoDB and Postgres will run. Volumes that you create with the script are assigned a label that uses the release name. This label is used later to bound each volume to the correct datastore pods.

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
    - `release`: Specify the release name. The release name must start with an alphabetic character, end with an alphanumeric character, and consist of all lower case alphanumeric characters or a hyphen (-). For example *my-150-wa*. This release name is added to the labels that are used in the `wa-persistence.yaml` file. Be sure to use the same release name when you run the service install command later. If you don't specify a name, a timestamp value is used. The timestamp has the format: `YYYY-MM-DD--HH-mm`.
    - `tmp`: Name of the temporary directory in which to store the persistent volumes. The default value is `/tmp`.

    For example:

    ```bash
    ./clusterAdministration/storage.sh --release my-150-wa --label "node-role.kubernetes.io/worker=true" \
     --nodes 192.18.36.181,192.18.36.182,192.18.36.183,192.18.36.184,192.18.36.185
    ```
    {: codeblock}

    A `wa-persistence.yaml` file is created by the script.

1.  {: #install-150-replace-local-storage-class}Configure {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} to use local storage by editing the `overrides.yaml` file.

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
| wa-my-150-wa-etcd-10gi-1 | 10Gi | dedication=wa-my-150-wa-etcd,release=my-150-wa |
| wa-my-150-wa-etcd-10gi-2 | 10Gi | dedication=wa-my-150-wa-etcd,release=my-150-wa |
| wa-my-150-wa-etcd-10gi-3 | 10Gi | dedication=wa-my-150-wa-etcd,release=my-150-wa |
| wa-my-150-wa-etcd-10gi-4 | 10Gi | dedication=wa-my-150-wa-etcd,release=my-150-wa |
| wa-my-150-wa-etcd-10gi-5 | 10Gi | dedication=wa-my-150-wa-etcd,release=my-150-wa |
| wa-my-150-wa-minio-5gi-1 | 5Gi | dedication=wa-my-150-wa-minio,release=my-150-wa | 
| wa-my-150-wa-minio-5gi-2 | 5Gi | dedication=wa-my-150-wa-minio,release=my-150-wa |
| wa-my-150-wa-minio-5gi-3 | 5Gi | dedication=wa-my-150-wa-minio,release=my-150-wa |
| wa-my-150-wa-minio-5gi-4 | 5Gi | dedication=wa-my-150-wa-minio,release=my-150-wa |
| wa-my-150-wa-mongodb-75gi-1 | 75Gi | dedication=wa-my-150-wa-mongodb,release=my-150-wa |
| wa-my-150-wa-mongodb-75gi-2 | 75Gi | dedication=wa-my-150-wa-mongodb,release=my-150-wa |
| wa-my-150-wa-mongodb-75gi-3 | 75Gi | dedication=wa-my-150-wa-mongodb,release=my-150-wa |
| wa-my-150-wa-postgres-10gi-1 | 10Gi | dedication=wa-my-150-wa-postgres,release=my-150-wa | 
| wa-my-150-wa-postgres-10gi-2 | 10Gi | dedication=wa-my-150-wa-postgres,release=my-150-wa |
| wa-my-150-wa-postgres-10gi-3 | 10Gi | dedication=wa-my-150-wa-postgres,release=my-150-wa |
| wa-my-150-wa-backup-1gi-1 | 1Gi | dedication=wa-my-150-wa-backup,release=my-150-wa |
{: caption="Unique persistent volume information" caption-side="top"}

The following information is also provided, but is the same for all volumes:

Table 5b. More sample persistent volume information

| ACCESS MODES | RECLAIM POLICY | STATUS | STORAGECLASS |
|--------------|----------------|--------|--------------|
| RWO | Retain | Available | local-storage |
{: caption="Static persistent volume information" caption-side="top"}

Be careful if you need to update or stop a node with bounded local-storage persistent volumes to perform maintenance tasks.

### Language considerations
{: #install-150-lang-reqs}

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

At installation time, you must specify a configuration setting for each language that you want to support. For more information, see [Creating an override file for Watson Assistant](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.5.0/cpd/install//svc-assistant/assistant-svc-override.html){: external}

## Launching the product
{: #install-150-launch-tool}

1.  Open an incognito window in your web browser to prevent cached information from being used by the product.
1.  From the main {{site.data.keyword.icp4dfull_notm}} web client navigation menu, select *Services>Instances*, find your {{site.data.keyword.conversationshort}} instance, and double-click it to open it.
1.  Click **Launch tool**.

## Next steps
{: #install-150-next-steps}

Use the {{site.data.keyword.conversationshort}} product user interface to build training data and a dialog that can be used by your assistant.

- To learn more about the service first, read the [overview](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-index).
- To see how it works for yourself, follow the steps in the [getting started tutorial](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-getting-started).
- For help managing the cluster, see [Managing the cluster](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-manage).

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
