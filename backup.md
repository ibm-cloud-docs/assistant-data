---

copyright:
  years: 2015, 2022
lastupdated: "2022-11-04"

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

# Backing up and restoring data
{: #backup}

You can back up and restore the data that is associated with your {{site.data.keyword.conversationshort}} deployment in {{site.data.keyword.icp4dfull_notm}}.
{: shortdesc}

The primary data storage for {{site.data.keyword.conversationshort}} is a Postgres database. Your data, such as workspaces, assistants, and skills are stored in Postgres. Other internal data, such as trained models, can be recreated from the data in Postgres.

Choose one of the following ways to manage the back up of data:

- **[Kubernetes CronJob](#backup-cronjob)**: Use the `$INSTANCE-store-cronjob` cron job that is provided for you.
- **[backupPG.sh script](#backup-os)**: Use the `backupPG.sh` bash script.
- **[pg_dump tool](#backup-cp4d)**: Run the `pg_dump` tool on each cluster directly. This is the most manual option, but also affords the most control over the process.

When you back up data with one of these procedures before you upgrade from one version to another, the workspace IDs of your skills are preserved, but the service instance IDs and credentials change.
{: note}

## Before you begin

- When you create a backup by using this procedure, the backup includes all of the assistants and skills from all of the service instances. Meaning it can include even skills and assistants to which you do not have access.
- The access permissions information of the original service instances is not stored in the backup. Meaning original access rights, which determine who can see a service instance and who cannot, are not preserved.
- You cannot use this procedure to back up the data that is returned by the search skill. Data that is retrieved by the search skill comes from a data collection in a {{site.data.keyword.discoveryshort}} instance. See the [{{site.data.keyword.discoveryshort}} documentation](/docs/discovery-data?topic=discovery-data-backup-restore) to find out how to back up its data.
- If you back up and restore or otherwise change the {{site.data.keyword.discoveryshort}} service that your search skill connects to, then you cannot restore the search skill, but must recreate it. When you set up a search skill, you map sections of the assistant's response to fields in a data collection that is hosted by an instance of {{site.data.keyword.discoveryshort}} on the same cluster. If the {{site.data.keyword.discoveryshort}} instance changes, your mapping to it is broken. If your {{site.data.keyword.discoveryshort}} service does not change, then the search skill can continue to connect to the data collection.
- The tool that restores the data clears the current database before it restores the backup. Therefore, if you might need to revert to the current database, create a backup of it first.
- The target {{site.data.keyword.icp4dfull_notm}} cluster where you restore the data must have the same number of provisioned {{site.data.keyword.conversationshort}} service instances as the environment from which you back up the database.

## Backing up data by using the CronJob
{: #backup-cronjob}

A CronJob named `$INSTANCE-store-cronjob` is created and enabled for you automatically when you deploy the service. A CronJob is a type of Kubernetes controller. A CronJob creates Jobs on a repeating schedule. For more information, see [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/){: external} in the Kubernetes documentation.

The jobs that are created by the store cron job are called `$INSTANCE-backup-job-$TIMESTAMP`. Each job deletes old logs and runs a backup of the store Postgres database. Postgres provides a tool that is called `pg_dump`. The dump tool creates a backup by sending the database contents to `stdout` where you can write it to a file. The backups are created with the `pg_dump` command and stored in a persistent volume claim (PVC) named $INSTANCE-store-pvc.

You are responsible for moving the backup to a more secure location after its initial creation, preferrably a location that can be accessed outside of the cluster where the backups cannot be deleted easily. Ensure this happens for all environments, especially for Production clusters.
{: note}

The following table lists the configuration values that control the backup cron job. You can change the default values for these settings by adding them to the `install-override.yaml` file and changing their default values when you deploy the service. Or you can edit these settings by editing the cron job after the service is deployed by using the `oc edit cronjob $INSTANCE-store-cronjob` command.

| Variable | Description | Default value |
|----------|-------------|---------------|
| store.backup.suspend | If True, the cron job does not create any backup jobs. | `False` |
| store.backup.schedule | Specifies the time of day at which to run the backup jobs. Specify the schedule by using a cron expression. For example `{minute} {hour} {day} {month} {day-of-week}` where `{day-of-week}` is specified as `0`=Sunday, `1`=Monday, and so on. The default schedule is to run every day at 11 PM. | `0 23 * * *` |
| store.backup.history.jobs.success | The number of successful jobs to keep. | `30` |
| store.backup.history.jobs.failed | The number of failed jobs to keep in the job logs. | `10` |
| store.backup.history.files.weekly_backup_day | A day of the week is designated as the weekly backup day. 0=Sunday, 1=Monday and so on. | `0` |
| store.backup.history.files.keep_weekly | The number of backups to keep that were taken on weekly_backup_day. | `4`  |
| store.backup.history.files.keep_daily | The number of backups to keep that were taken on all the other days of the week | `6`  |
{: caption="Cron job variables" caption-side="top"}

### Accessing backed-up files from Portworx
{: #backup-access-portworx}

To access the backup files from Portworx, complete the following steps:

1.  Get the name of the persistent volume that is used for the Postgres backup:

    ```bash
    oc get pv |grep $INSTANCE-store
    ```
    {: codeblock}

    This command returns the name of the persistent volume claim where the store backup is located, such as `pvc-d2b7aa93-3602-4617-acea-e05baba94de3`. The name is referred to later in this procedure as the `$pv_name`.

1.  Find nodes where Portworx is running:

    ```bash
    oc get pods -n kube-system -o wide -l name=portworx-api
    ```
    {: codeblock}

1.  Log in as the core user to one of the nodes where Portworx is running:

    ```bash
    ssh core@<node hostname>
    sudo su -
    ```
    {: codeblock}

1.  Make sure the persistent volume is in a detached state and that no store backups are scheduled to occur during the time you plan to transfer the backup files.

    Remember, backups occur daily at 11 PM (in the time zone configured for the nodes) unless you change the schedule by editing the value of the `postgres.backup.schedule` configuration parameter. You can run the `oc get cronjobs` command to check the current schedule for the `$RELEASE-backup-cronjob` job. In the following command, `$pvc_node` is the name of the node that you discovered in the first step of this task:

    ```bash
    pxctl volume inspect $pv_name |head -40
    ```
    {: codeblock}

1.  Attach the persistent volume to the host:

    ```bash
    pxctl host attach $pv_name
    ```
    {: codeblock}

1.  Create a folder where you want to mount the node:

    ```bash
    mkdir /var/lib/osd/mounts/voldir
    ```
    {: codeblock}

1.  Mount the node:

    ```bash
    pxctl host mount $pv_name --path /var/lib/osd/mounts/voldir
    ```
    {: codeblock}

1.  Change directory to `/var/lib/osd/mounts/voldir`. Transfer backup files to a secure location. Afterwards, exit the directory. Unmount the volume:

    ```bash
    pxctl host unmount --path /var/lib/osd/mounts/voldir $pv_name
    ```
    {: codeblock}

1.  Detach the volume from the host:

    ```bash
    pxctl host detach $pv_name
    ```
    {: codeblock}

1.  Make sure the volume is in the detached state. Otherwise, subsequent backups will fail:

    ```bash
    pxctl volume inspect $pv_name |head -40
    ```
    {: codeblock}

### Accessing backed-up files from OpenShift Container Storage
{: #backup-access-ocs}

To access the backup files from OpenShift Container Storage (OCS), complete the following steps:

1.  Create a volume snapshot of the persistent volume claim that is used for the Postgres backup:

    ```yaml
    cat <<EOF | oc apply -f -
    apiVersion: snapshot.storage.k8s.io/v1
    kind: VolumeSnapshot
    metadata:
      name: wa-backup-snapshot
    spec:
      source:
        persistentVolumeClaimName: ${INSTANCE_NAME}-store-pvc
      volumeSnapshotClassName: ocs-storagecluster-rbdplugin-snapclass
    EOF
    ```
    {: codeblock}

1.  Create a persistent volume claim from the volume snapshot:

    ```yaml
    cat <<EOF | oc apply -f -
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: wa-backup-snapshot-pvc
    spec:
      storageClassName: ocs-storagecluster-ceph-rbd
      accessModes:
      - ReadWriteOnce
      volumeMode: Filesystem
      dataSource:
        apiGroup: snapshot.storage.k8s.io
        kind: VolumeSnapshot
        name: wa-backup-snapshot
      resources:
        requests:
          storage: 1Gi     
    EOF
    ```
    {: codeblock}

1.  Create a pod to access the persistent volume claim:

    ```yaml
    cat <<EOF | oc apply -f -
    kind: Pod
    apiVersion: v1
    metadata:
      name: wa-retrieve-backup
    spec:
      volumes:
        - name: backup-snapshot-pvc
          persistentVolumeClaim:
           claimName: wa-backup-snapshot-pvc
      containers:
        - name: retrieve-backup-container
          image: cp.icr.io/cp/watson-assistant/conan-tools:20210630-0901-signed@sha256:e6bee20736bd88116f8dac96d3417afdfad477af21702217f8e6321a99190278
          command: ['sh', '-c', 'echo The pod is running && sleep 360000']
          volumeMounts:
            - mountPath: "/watson_data"
              name: backup-snapshot-pvc
    EOF
    ```
    {: codeblock}

1.  If you do not know the name of the backup file that you want to extract and are unable to check the most recent backup cron job, run the following command:

    ```bash
    oc exec -it wa-retrieve-backup -- ls /watson_data
    ```
    {: codeblock}

1.  Transfer the backup files to a secure location:

    ```bash
    kubectl cp wa-retrieve-backup:/watson_data/${FILENAME} ${SECURE_LOCAL_DIRECTORY}/${FILENAME}
    ```
    {: codeblock}

1.  Run the following commands to clean up the resources that you created for to retrieve the files:

    ```bash
    oc delete wa-retrieve-backup
    oc delete pvc wa-restore-backup
    oc delete volumesnapshot wa-backup-snapshot-pvc
    ```
    {: codeblock}

## Backing up data by using the script
{: #backup-os}

The `backupPG.sh` script gathers the pod name and credentials for one of your Postgres pods, which is the pod from which the `pg_dump` command must be run, and then runs the command for you.

To back up data by using the provided script, complete the following steps:

1.  Download the `backupPG.sh` script.

    Go to [GitHub](https://github.com/watson-developer-cloud/community/blob/master/watson-assistant/data/){: external}, and find the directory for your version to find the file.

1.  Log in to the OpenShift project namespace where you installed the product.

1.  Find out how many provisioned service instances there are in your existing cluster. You need to know this information so you can be sure to set up the target cluster with the same number of instances.

    To find out, open the {{site.data.keyword.icp4dfull_notm}} web client. From the main navigation menu, select **Services**, then **Instances**, and then open the **Provisioned instances** tab.

    If more than one person created instances, then ask the other people who created instances to log in and check the number they created. You can then sum them to get the total number of instances for your deployment. Not even an administrative user can see instances that were created by others from the web client user interface.

1.  Run the script:

    ```bash
    ./backupPG.sh --instance ${INSTANCE} > ${BACKUP_DIR}
    ```
    {: codeblock}

    Replace the following values in the command:

    - `${BACKUP_DIR}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/backup-file-name.dump` creates a backup directory named `bu`.
    - `--instance ${INSTANCE}`: Select the specific instance of {{site.data.keyword.conversationshort}} to be backed up.

If you prefer to back up data by using the Postgres tool directly, you can complete the procedure to back up data manually.

## Backing up data manually
{: #backup-cp4d}

Complete the steps in this procedure to back up your data by using the Postgres tool directly.

To back up your data, complete these steps:

1.  Fetch a running Postgres pod:

    ```bash
    oc get pods -l app=${INSTANCE}-postgres -o jsonpath="{.items[0].metadata.name}"
    ```
    {: codeblock}

    Replace ${INSTANCE} with the instance of the {{site.data.keyword.conversationshort}} deployment that you want to back up.

1.  Fetch the store VCAP secret name:

    ```bash
    oc get secrets -l component=store,app.kubernetes.io/instance=${INSTANCE} -o=custom-columns=NAME:.metadata.name | grep store-vcap
    ```
    {: codeblock}

1.  Fetch the Postgres connection values. You will pass these values to the command that you run in the next step. You must have `jq` installed.

    - To get the database:

      ```bash
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.database'
      ```
      {: codeblock}

    - To get the hostname:

      ```bash
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.host'
      ```
      {: codeblock}

    - To get the username:

      ```bash
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.username'
      ```
      {: codeblock}

    - To get the password:

      ```bash
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.password'
      ```
      {: codeblock}

1.  Run the following command:

    ```bash
    oc exec $KEEPER_POD -- bash -c "export PGPASSWORD='$PASSWORD' && pg_dump -Fc -h $HOSTNAME -d $DATABASE -U $USERNAME" > ${BACKUP_DIR}
    ```
    {: codeblock}

    The following lists describes the arguments. You retrieved the values for some of these parameters in the previous step:

    - `$KEEPER_POD`: Any Postgres pod in your {{site.data.keyword.conversationshort}} instance.
    - `${BACKUP_DIR}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/backup-file-name.dump` creates a backup directory named `bu`.
    - `$DATABASE`: The store database name that was retrieved from the Store VCAP secret in step 3.
    - `$HOSTNAME`: The hostname that was retrieved from the Store VCAP secret in step 3.
    - `$USERNAME`: The username that was retrieved from the Store VCAP secret in step 3.
    - `$PASSWORD`: The password that was retrieved from the Store VCAP secret in step 3.

    To see more information about the `pg_dump` command, you can run this command:

    ```bash
    oc exec -it ${KEEPER_POD} -- pg_dump --help
    ```
    {: codeblock}

## Restoring data
{: #backup-restore}

IBM created a restore tool called `pgmig`. The tool restores your database backup by adding it to a database you choose. It also upgrades the schema to the one that is associated with the version of the product where you restore the data. Before the tool adds the backed-up data, it removes the data for all instances in the current service deployment, so any spares are also removed.

1.  Install the target {{site.data.keyword.icp4dfull_notm}} cluster to which you want to restore the data.

    From the web client for the target cluster, create one service instance of {{site.data.keyword.conversationshort}} for each service instance that was backed up on the old cluster. The target {{site.data.keyword.icp4dfull_notm}} cluster must have the same number of instances as there were in the environment where you backed up the database.

1.  Back up the current database before you replace it with the backed-up database.

    The tool clears the current database before it restores the backup. So, if you might need to revert to the current database, be sure to create a backup of it first.

1.  Go to the backup directory that you specified in the `${BACKUP_DIR}` parameter in the previous procedure.

1.  Run the following command to download the `pgmig` tool from the [GitHub Watson Developer Cloud Community](https://github.com/watson-developer-cloud/community/tree/master/watson-assistant/data) repository.

    In the first command, update `<WA_VERSION>` to the version that you want to restore. For example, update `<WA_VERSION>` to `4.5.0` if you want to restore {{site.data.keyword.conversationshort}} 4.5.0.
    {:important: .important}

    ```bash
    wget https://github.com/watson-developer-cloud/community/raw/master/watson-assistant/data/<WA_VERSION>/pgmig
    chmod 755 pgmig
    ```
    {: codeblock}

1.  Create the following two configuration files and store them in the same backup directory:

    - `resourceController.yaml`: The Resource Controller file keeps a list of all provisioned {{site.data.keyword.conversationshort}} instances. See [Creating the resourceController.yaml file](#backup-resource-controller-yaml).

    - `postgres.yaml`: The Postgres file lists details for the target Postgres pods. See [Creating the postgres.yaml file](#backup-postgres-yaml).

1.  Get the secret:

    ```bash
    oc get secret ${INSTANCE}-postgres-ca -o jsonpath='{.data.ca\.crt}' | base64 -d | tee ${BACKUP_DIR}/ca.crt | openssl x509 -noout -text
    ```
    {: codeblock}

    - Replace `${INSTANCE}` with the name of the {{site.data.keyword.conversationshort}} instance that you want to back up.
    - Replace `${BACKUP_DIR}` with the directory where the `postgres.yaml` and `resourceController.yaml` files are located.

1.  Copy the files that you downloaded and created in the previous steps to any existing directory on a Postgres pod.

    1. Run the following command to find Postgres pods:

        ```bash
        oc get pods | grep postgres
        ```
        {: codeblock}

    1. The files that you must copy are `pgmig`, `postgres.yaml`, `resourceController.yaml`, and the file that you created for your downloaded data. Run the following commands to copy the files.

        If you are restoring data to a stand-alone {{site.data.keyword.icp4dfull_notm}} cluster, then replace all references to `oc` with `kubectl` in these sample commands.
        {: note}

        ```bash
        oc exec -it ${POSTGRES_POD} -- mkdir /controller/tmp
        oc exec -it ${POSTGRES_POD} -- mkdir /controller/tmp/bu
        oc rsync bu/ ${POSTGRES_POD}:/controller/tmp/bu/
        ```
        {: codeblock}

        - Replace `${POSTGRES_POD}` with the name of one of the Postgres pods from the previous step.

1.  Stop the store deployment by scaling the store deployment down to 0 replicas:

    ```bash
    oc scale deploy ibm-watson-assistant-operator -n ${OPERATOR_NS} --replicas=0
    oc get deployments -l component=store
    ```
    {: codeblock}

    Make a note of how many replicas there are in the store deployment:

    ```bash
    oc scale deployment ${STORE_DEPLOYMENT} --replicas=0
    ```
    {: codeblock}

1.  Initiate the execution of a remote command in the Postgres pod:

    ```bash
    oc exec -it ${POSTGRES_POD} /bin/bash
    ```
    {: codeblock}

1.  Run the `pgmig` tool:

    ```bash
    cd /controller/tmp/bu
    export PG_CA_FILE=/controller/tmp/bu/ca.crt
    ./pgmig --resourceController resourceController.yaml --target postgres.yaml --source <backup-file-name.dump>
    ```
    {: codeblock}

    - Replace `<backup-file-name.dump>` with the name of the file that you created for your downloaded data.

    For more command options, see [Postgres migration tool details](#backup-pgmig-details).

    As the script runs, you are prompted for information that includes the instance on the target cluster to which to add the backed-up data. The data on the instance you specify will be removed and replaced. If there are multiple instances in the backup, you are prompted multiple times to specify the target instance information.

1.  Scale the store deployment back up:

    ```bash
    oc scale deployment ${STORE_DEPLOYMENT} --replicas=${ORIGINAL_NUMBER_OF_REPLICAS}
    oc scale deploy ibm-watson-assistant-operator -n ${OPERATOR_NS} --replicas=1
    ```
    {: codeblock}

    You might need to wait a few minutes before the skills you restored are visible from the web interface.

Reopen only one assistant or dialog skill at a time. Each time you open a dialog skill after its training data has been changed, training is initiated automatically. Give the skill time to retrain on the restored data. Remember, the process of training a machine learning model requires at least one node to have 4 CPUs that can be dedicated to training. Therefore, open restored assistants and skills during low traffic periods and open them one at a time.

### Creating the resourceController.yaml file
{: #backup-resource-controller-yaml}

The **resourceController.yaml** file contains details about the new environment where you are adding the backed-up data. Add the following information to the file:

```yaml
accessTokens: 
  - value
  - value2
host: localhost
port: 5000
```
{: codeblock}

To add the values that are required but currently missing from the file, complete the following steps:

1.  To get the accessTokens values list, you need to get a list of bearer tokens for the service instances.

    - Log in to the {{site.data.keyword.icp4dfull_notm}} web client.
    - From the main {{site.data.keyword.icp4dfull_notm}} web client navigation menu, select **My instances**.
    - On the **Provisioned instances** tab, find your {{site.data.keyword.conversationshort}} instance, and then hover over the last column to show and click the ellipses icon ![More icon](images/cp4d-sideways-kebab.png).
    - Choose **View details**.
    - In the details of the instance, find the **Bearer token**. Copy the token and paste it into the accessTokens list.

    A bearer token for an instance can access all instances that are owned by the user. Therefore, if a single user owns all of the instances, then only one bearer token is required.

    If the service has multiple instances, each owned by a different user, then you must gather bearer tokens for each user who owns an instance. You can list multiple bearer token values in the `accessTokens` section.

1.  To get the host information, you need details for the pod that hosts the {{site.data.keyword.conversationshort}} UI component: 

    ```bash
    oc describe pod -l component=ui
    ```
    {: codeblock}

    Look for the section that says, `RESOURCE_CONTROLLER_URL: https://${release-name}-addon-assistant-gateway-svc.zen:5000/api/ibmcloud/resource-controller`

    For example, you can use a command like this to find it:

    ```bash
    oc describe pod -l component=ui | grep RESOURCE_CONTROLLER_URL
    ```
    {: codeblock}

    Copy the host that is specified in the `RESOURCE_CONTROLLER_URL`. The host value is the `RESOURCE_CONTROLLER_URL` value, excluding the protocol at the beginning and everything from the port to the end of the value. For example, for the previous example, the host is `${release-name}-addon-assistant-gateway-svc.zen`.

1.  To get the port information, again check the RESOURCE_CONTROLLER_URL entry. The port is specified after `<host>:` in the URL. In this sample URL, the port is `5000`.

1.  Paste the values that you discovered into the YAML file and save it.

### Creating the postgres.yaml file
{: #backup-postgres-yaml}

The **postgres.yaml** file contains details about the Postgres pods in your target environment (the environment where you will restore the data). Add the following information to the file:

```yaml
host: localhost
port: 5432
database: store
username: user
su_username: admin
su_password: password
```
{: codeblock}

To add the values that are required but currently missing from the file, complete the following steps:

1.  To get information about the `host`, you must get the Store VCAP secret.

    ```bash
    oc get secret ${instance}-store-vcap -o jsonpath='{.data.vcap_services}' | base64 -d
    ```
    {: codeblock}

    Information for the Redis and Postgres databases is returned. Look for the segment of JSON code for the Postgres database, named `pgservice`. It looks like this:

    ```json
    {
      "user-provided":[
        {
          "name": "pgservice",
          "label": "user-provided",
          "credentials":
          {
            "host": "${instance}-rw",
            "port": 5432,
            "database": "conversation_pprd_${instance}",
            "username": "${dbadmin}",
            "password": "${password}"
          }
        }
      ],
      ...
    }
    ```
    {: codeblock}

1.  Copy the values for user-provided credentials (`host`, `port`, `database`, `username`, and `password`).

    You can specify the same values that were returned for `username` and `password` as the `su_username` and `su_password` values.

    The updated file will look something like this:

    ```yaml
    host: wa_inst-postgres-rw
    port: 5432
    database: conversation_pprd_wa_inst
    username: dbadmin
    su_username: dbadmin
    su_password: mypassword
    ```
    {: codeblock}

1.  Save the `postgres.yaml` file.

### Postgres migration tool details
{: #backup-pgmig-details}

The following table lists the arguments that are supported by the `pgmig` tool:

| Argument | Description |
|---------|-------------|
| -h, --help | Command usage |                    
| -f, --force | Erase data if present in target Store |
| -s, --source string | Backup file name |   
| -r, --resourceController string | Resource Controller configuration file name |
| -t, --target string | Target Postgres server configuration file name |
| -m, --mapping string | Service instance-mapping configuration file name (optional) |
| --testRCConnection | Test the connection for Resource Controller, then exit |
| --testPGConnection | Test the connection for Postgres server, then exit |
| -v, --version | Get Build version |
{: caption="pgmig tool arguments" caption-side="top"}

### The mapping configuration file
{: #backup-mapping-file}

After you run the script and specify the mappings when prompted, the tool generates a file that is named `enteredMapping.yaml` in the current directory. This file reflects the mapping of the old cluster details to the new cluster based on the interactive inputs that were provided while the script was running.

For example, the YAML file contains values like this:

```yaml
instance-mappings:
  00000000-0000-0000-0000-001570184978: 00000000-0000-0000-0000-001570194490
```
{: codeblock}

where the first value (`00000000-0000-0000-0000-001570184978`) is the instance ID in the database backup and the second value (`00000000-0000-0000-0000-001570194490`) is the ID of a provisioned instance in the {{site.data.keyword.conversationshort}} service on the system.

You can pass this file to the script for subsequent runs of the script in the same environment. Or you can edit it for use in other back up and restore operations. The mapping file is optional. If it is not provided, the tool prompts you for the mapping details based on information you provide in the YAML files.
