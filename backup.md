---

copyright:
  years: 2015, 2021
lastupdated: "2021-01-11"

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

Choose one of the following ways to manage the backup of data:

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

If you're using version 1.4.2, see the instructions [here](#backup-cronjob-142).

A CronJob named `$INSTANCE-store-cronjob` is created and enabled for you automatically when you deploy the service. A CronJob is a type of Kubernetes controller. A CronJob creates Jobs on a repeating schedule. For more information, see [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/){: external} in the Kubernetes documentation. 

The jobs that are created by the store cron job are called `$INSTANCE-backup-job-$TIMESTAMP`. Each job deletes old logs and runs a backup of the store Postgres database. Postgres provides a tool that is called `pg_dump`. The dump tool creates a backup by sending the database contents to `stdout` where you can write it to a file. The backups are created with the `pg_dump` command and stored in a persistent volume claim (PVC) named $INSTANCE-store-pvc . You are responsible for moving the backup to a more secure location after its initial creation.

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

1.  Get the name of the persistent volume that is used for the Postgres backup.

    ```bash
    oc get pv |grep $INSTANCE-store-backup
    ```
    {: codeblock}

    This command returns the name of the persistent volume claim where the store backup is located, such as `pvc-d2b7aa93-3602-4617-acea-e05baba94de3`. The name is referred to later in this procedure as the `$pv_name`.

1.  Find nodes where Portworx is running.

    ```bash
    oc get pods -n kube-system -o wide -l name=portworx-api
    ```
    {: codeblock}

1.  Log in as the core user to one of the nodes where Portworx is running.

    ```bash
    ssh core@<node hostname>
    sudo su -
    ```
    {: codeblock}

1.  Make sure the persistent volume is in a detached state and that no store backups are scheduled to occur during the time you plan to transfer the backup files.

    Remember, backups occur daily at 11 PM (in the time zone configured for the nodes) unless you change the schedule by editing the value of the `postgres.backup.schedule` configuration parameter. You can run the `oc get cronjobs` command to check the current schedule for the `$INSTANCE-store-cronjob` job.

    ```bash
    pxctl volume inspect $pv_name |head -40
    ```
    {: codeblock}

    where `$pvc_node` is the name of the node that you discovered in Step 1 of this procedure.

1.  Attach the persistent volume to the host.

   ```bash
   pxctl host attach $pv_name
   ```
   {: codeblock}

1.  Create a folder where you want to mount the node.

    ```bash
    mkdir /var/lib/osd/mounts/voldir
    ```
    {: codeblock}

1.  Mount the node.

    ```bash
    pxctl host mount $pv_name --path /var/lib/osd/mounts/voldir
    ```
    {: codeblock}

1.  Change Directory to `/var/lib/osd/mounts/voldir`. Transfer backup files to a secure location. Afterwards, exit the directory. Unmount the volume.

    ```bash
    pxctl host unmount --path /var/lib/osd/mounts/voldir $pv_name
    ```
    {: codeblock}

1.  Detach the volume from the host.

    ```bash
    pxctl host detach $pv_name
    ```
    {: codeblock}

1.  Make sure the volume is in the detached state. Otherwise, subsequent backups will fail.

    ```bash
    pxctl volume inspect $pv_name |head -40
    ```
    {: codeblock}

## Backing up data by using the script
{: #backup-os}

If you're using version 1.4.2, see the instructions [here](#backup-os-142).

The `backupPG.sh` script gathers the pod name and credentials for one of your Postgres Keeper pods, which is the pod from which the `pg_dump` command must be run, and then runs the command for you.

To back up data by using the provided script, complete the following steps:

1.  Download the `backupPG.sh` script. 

    Go to [GitHub](https://github.com/watson-developer-cloud/community/blob/master/watson-assistant/data/){: external}, and find the directory for your version to find the file.
1.  Log in to the OpenShift project namespace where you installed the product.
1.  Find out how many provisioned service instances there are in your existing cluster. You need to know this information so you can be sure to set up the target cluster with the same number of instances.

    To find out, open the {{site.data.keyword.icp4dfull_notm}} web client. From the main navigation menu, select Services, then **Instances**, and then open the **Provisioned instances** tab.

    If more than one person created instances, then ask the other people who created instances to log in and check the number they created. You can then sum them to get the total number of instances for your deployment. Not even an administrative user can see instances that were created by others from the web client user interface.

1.  Run the script by using the following command:

    ```
    ./backupPG.sh [--instance ${instance-name}] > ${file-name}
    ```
    {: codeblock}

    where these are the arguments:

    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.
    - `--instance ${instance-name}`: Targets a specific instance of the Assistant deployment. Otherwise, the script backs up the first instance it finds in the namespace you are logged in to. You can find the instance name by running the command `./cpd-cli status -n $NAMESPACE`.

If you prefer to back up data by using the Postgres tool directly, you can complete the procedure to back up data manually.

## Backing up data manually
{: #backup-cp4d}

Complete the steps in this procedure to back up your data by using the Postgres tool directly.

To back up your data, complete these steps:

1.  Fetch a running Postgres keeper pod.

    ```
    oc get pods --field-selector=status.phase=Running -l component=stolon-keeper,instance=${INSTANCE} -o jsonpath="{.items[0].metadata.name}"
    ```
    {: codeblock}

    Replace ${INSTANCE} with the instance of the Assistant deployment that you want to back up.

1.  Fetch the store VCAP secret name.

    ```
    oc get secrets -l component=store${INSTANCE} -o=custom-columns=NAME:.metadata.name | grep store-vcap

    ```
    {: codeblock}

1.  Fetch the Postgres connection values. You will pass these values to the command that you run in the next step. (You must have `jq` installed.)

    - To get the database:

      ```
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.database'
      ```
      {: codeblock}

    - To get the hostname:

      ```
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.host')
      ```

1.  Run the following command:

    ```
    oc exec $KEEPER_POD -- bash -c "export PGPASSWORD='$PASSWORD' && pg_dump -Fc -h $HOSTNAME -d $DATABASE -U $USERNAME" > ${file-name}
    ```
    {: codeblock}

    The following lists describes the arguments. You retrieved the values for some of these parameters in the previous step:

    - `KEEPER_POD`: Any Postgres Keeper pod in your {{site.data.keyword.conversationshort}} instance.
    - `DATABASE`: The store database name.
    - `HOSTNAME`: The hostname.
    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.
    -  The `su_username` and `su_password` from the Store VCAP secret are retrieved and used for `$PASSWORD` and `$USERNAME`.

    To see more information about the `pg_dump` command, you can run this command:

    ```bash
    oc exec -it ${KEEPER_POD} --pg_dump --help
    ```
    {: pre}

## Restoring data
{: #backup-restore}

IBM created a restore tool called `pgmig`. The tool restores your database backup by adding it to a database you choose. It also upgrades the schema to the one that is associated with the version of the product where you restore the data.

Before it adds the backed-up data, the tool removes the data for all instances in the current service deployment, so any spares are removed also.
{: important}

1.  Install the target {{site.data.keyword.icp4dfull_notm}} cluster to which you want to restore the data. From the web client for the target cluster, create one service instance of {{site.data.keyword.conversationshort}} for each service instance that was backed up on the old cluster.

    The target {{site.data.keyword.icp4dfull_notm}} cluster must have the same number of instances as there were in the environment where you backed up the database.
    {: important}

1.  Back up the current database before you replace it with the backed-up database.

    The tool clears the current database before it restores the backup. So, if you might need to revert to the current database, be sure to create a backup of it first.

1.  Go to the backup directory that you specified in the file-name parameter in the previous procedure.

1.  Download the `pgmig` tool from the correct directory for your release from the [GitHub Watson Developer Cloud Community](https://github.com/watson-developer-cloud/community/tree/master/watson-assistant/data) repository.

    For example:

    ```
    wget https://github.com/watson-developer-cloud/community/raw/master/watson-assistant/data/1.5.0/pgmig
    ```
    {: codeblock}

    ```
    chmod 755 pgmig
    ```
    {: codeblock}

1.  Create two configuration files, and store them in the same backup directory.

    - **resourceController.yaml**: The Resource Controller file keeps a list of all provisioned {{site.data.keyword.conversationshort}} instances. See [Creating the resourceController.yaml file](#backup-resource-controller-yaml).

    - **postgres.yaml**: The Postgres file lists details for the target Postgres pods. See [Creating the postgres.yaml file](#backup-postgres-yaml).

1.  Copy the files that you downloaded and created in the previous steps into an existing directory of your choice on a Postgres Keeper pod.

    Use the following command to find the keeper pods:

    ```
    oc get pods --field-selector=status.phase=Running -l component=stolon-keeper
    ```
    {: pre}

    The files that you need to copy are `pgmig`, `postgres.yaml`, `resourceController.yaml`, and `store.dump`. 

    You can use the following commands to do so. 
    
    If you are restoring data to a stand-alone {{site.data.keyword.icp4dfull_notm}} cluster, then replace all references to `oc` with `kubectl` in these sample commands.
    {: note}

    ```bash
    oc exec -it $KEEPER_POD -- mkdir /tmp/bu
    ```
    {: codeblock}

    ```bash
    oc rsync $BACKUP_DIR $KEEPER_POD:/tmp/bu/.
    ```
    {: codeblock}
    <!-- add command for stand-alone since there is no kubectl rsync command -->

1.  Stop the Store by scaling the store pods down to 0 replicas.

    ```bash
    oc get deployments -l component=store
    ```
    {: codeblock}
    
    Make a note of how many replicas there are.

    ```bash
    oc scale deployment $STORE_DEPLOYMENT_NAME --replicas=0
    ```
    {: codeblock}

1.  Initiate the execution of a remote command in the Keeper Pod.

    ```bash
    oc exec -it $KEEPER_POD /bin/bash
    ```
    {: codeblock}

1.  Run the `pgmig` tool.

    ```
    cd /tmp/bu
    ./pgmig --resourceController resourceController.yaml --target postgres.yaml --source store.dump
    ```
    {: codeblock}

    For more command options, see [Postgres migration tool details](#backup-pgmig-details).

    As the script runs, you are prompted for information that includes the instance on the target cluster to which to add the backed-up data. The data on the instance you specify will be removed and replaced. If there are multiple instances in the backup, you are prompted multiple times to specify the target instance information.

1.  Scale the Store database pods back up.

    ```bash
    oc scale deployment $STORE_DEPLOYMENT --replicas=$ORIGINAL_NUMBER_OF_REPLICAS
    ```
    {: codeblock}

You might need to wait a few minutes before the skills you restored are visible from the web UI. 

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

1.  To get the host information, you need details for the pod that hosts the Assistant UI component: 

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

    Copy the host that is specified in the `RESOURCE_CONTROLLER_URL`.

1.  To get the port information, again check the RESOURCE_CONTROLLER_URL entry. The port is specified after `<host>:` in the URL. In this sample URL, the port is `5000`.

1.  Paste the values that you discovered into the YAML file and save it.

### Creating the postgres.yaml file
{: #backup-postgres-yaml}

The **postgres.yaml** file contains details about the Postgres pods from the old environment where you backed up the data. Add the following information to the file:

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

    ```
    oc get secret ${instance}-store-vcap -o jsonpath='{.data.vcap_services}' | base64 -d

    ```
    {: codeblock}

    Information for the Redis and Postgres databases is returned. Look for the segment of JSON code for the Postgres database, named `pgservice`. It looks like this:

    ```
    {
      "user-provided":[
        {
          "name": "pgservice", 
          "label": "user-provided", 
          "credentials": 
          {
            "host": "${instance}-postgres-proxy-service.${namespace}.svc.cluster.local", 
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
    host: wa_inst-postgres-proxy-service.my150wa.svc.cluster.local
    port: 5432
    database: conversation_pprd_wa_inst
    username: dbadmin
    su_username: dbadmin
    su_password: mypassword
    ```
    {: codeblock}

1.  Save the **postgres.yaml** file.

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

## Backup instructions for previous releases

### Backing up data by using the CronJob (1.4.2 only)
{: #backup-cronjob-142}

A CronJob named `$RELEASE-backup-cronjob` is created and enabled for you automatically when you deploy the service. A CronJob is a type of Kubernetes controller. A CronJob creates Jobs on a repeating schedule. For more information, see [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/){: external} in the Kubernetes documentation. 

The jobs that are created by the backup cron job are called `$RELEASE-backup-job-$TIMESTAMP`. Each job deletes old logs and runs a backup of the store Postgres database. The backups are created with the `pg_dump` command and stored. You decide where to store the backups:

- **Portworx**: Jobs store backups in Portworx storage.
- **Local persistent volume claim**: Jobs store backups in a local storage persistent volume claim. If the `postgres.backup.persistence.enabled` configuration setting is set to `true`, a local storage persistent volume claim is created for you as part of the deployment process.
- **Job logs**: If the `postgres.backup.persistence.enabled` configuration setting is set to `false`, the jobs store the backups in the job logs only. 

  If you disable persistence and you're using the `storage.sh` script to create local storage for the deployment, be sure to set `pgBackupLocalStorage` to `false` so that a persistent volume is not created for storing backups. For more information, see [Creating persistent volumes for a development deployment](/docs/assistant-data?topic=assistant-data-install-142#install-142-create-pvs-dev).
  {: tip}

Regardless of the temporary storage method you choose, you are reponsible for moving the backup to a more secure location after its initial creation.
{: important}

The following table lists the configuration values that control the backup cron job. You can change the default values for these settings by adding them to the `wa-override.yaml` file and changing their default values when you deploy the service. Or you can edit these settings by editing the cron job after the service is deployed by using the `oc edit cronjob $RELEASE-backup-cronjob` command.

| Variable | Description | Default value |
|----------|-------------|---------------|
| postgres.backup.suspend | If true, the cron job does not create any backup jobs. | `false` |
| postgres.backup.schedule | Specifies the time of day at which to run the backup jobs. Specify the schedule by using a cron expression. For example `{minute} {hour} {day} {month} {day-of-week}` where `{day-of-week}` is specified as `0`=Sunday, `1`=Monday, and so on. The default schedule is to run every day at 11 PM. | `0 23 * * *` |
| postgres.backup.history.jobs.success | The number of successful jobs to keep. If `postgres.backup.persistence.enabled` is set to `false`, specifies the number of valid backup dumps to store in the job logs. | `30` |
| postgres.backup.history.jobs.failed | The number of failed jobs to keep in the job logs. | `10` |
| postgres.backup.persistence.enabled | If set to `true`, backups are written to persistent storage. | `true` |
{: caption="Cron job variables" caption-side="top"}

The following values configure how backups are stored in the persistent volume. They are only used if `postgres.backup.persistence.enabled` is `true`.

| Variable | Description | Default value |
|----------|-------------|---------------|
| postgres.backup.history.files.weeklyBackupDay | A day of the week is designated as the weekly backup day. 0=Sunday, 1=Monday and so on. | `0` |
| postgres.backup.history.files.weekly | The number of weekly backups to keep. | `4` |
| postgres.backup.history.files.daily | The number of daily backups to keep. | `6`  |
| postgres.backup.dataPVC.name | The name of the persistent volume claim in which to store the backups. | `store-backup` |
| postgres.backup.dataPVC.storageClassName | The storage class to use in the persistent volume claim. By default, a persistent volume claim with the same class that you use for the main deployment, which is typically `portworx-assistant` is used. | `global.storageClassName`  |
| postgres.backup.dataPVC.size | The size of the persistent volume claim. | `1Gi` |
{: caption="Cron job persistent volume variables" caption-side="top"}

### Backing up data by using the script (1.4.2 and earlier only)
{: #backup-os-142}

The `backupPG.sh` script gathers the pod name and credentials for one of your Postgres Proxy pods, which is the pod from which the `pg_dump` command must be run, and then runs the command for you.

To back up data by using the provided script, complete the following steps:

1.  Log in to the OpenShift project namespace or Kubernetes namespace where you installed the product.
1.  The `backupPG.sh` script is included in the Helm chart for the product. The chart can be found in fileserver https://github.com/IBM/cloud-pak/raw/master/repo/cpd3/modules/ibm-watson-assistant/x86_64/1.4.2/. Extract the backup.sh file from the Helm chart named `ibm-watson-assistant-prod-1.4.2.tgz`. The script is in the `ibm_cloud_pak/pak_extensions/post-install/namespaceAdministration` directory.

1.  Find out how many provisioned service instances there are in your existing cluster. To find out, open the {{site.data.keyword.icp4dfull_notm}} web client. From the main navigation menu, select **My instances**, and then open the **Provisioned instances** tab.

    You need to know this information so you can be sure to set up the target cluster with the same number of instances.

1.  Run the script by using the following command:

    ```
    ./backupPG.sh [--release ${release-name}] > ${file-name}
    ```
    {: codeblock}

    where these are the arguments:

    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.
    - `--release ${release-name}`: Targets a specific release. Otherwise, the script backs up the first release it finds in the namespace you are logged in to.

If you prefer to back up data by using the Postgres tool directly, you can complete the procedure to back up data manually.

### Accessing backed-up local storage files (1.4.2 only)
{: #backup-access-local-storage}

To access the backup files from local storage:

1.  SSH into the node on which you created the persistent volume.
1.  Run the following command to get the details of the persistent volume that you created by using the `storage.sh` script. 

    The `$pv_name` has the syntax `wa-$TIMESTAMP-backup-1gi-1`.

    ```bash
    oc get -o yaml pv $pv_name
    ```
    {: codeblock}

    where `$pv_name` is the name of the persistent volume.

    You will see the hostname and directory path where the backups are being written. 

1.  Log in to the node as the core user and change to the directory where the backups are stored.

    ```bash
    ssh core@<node hostname>
    cd $storage_path
    ```
    {: codeblock}

1.  Securely copy the files to wherever you want to store them for a longer period of time.

### Backing up data manually (1.4.2 and earlier)
{: #backup-cp4d-142}

Complete the steps in this procedure to back up your data by using the Postgres tool directly.

To back up your data, complete these steps:

1.  Fetch a running Postgres keeper pod.

    ```
    oc get pods --field-selector=status.phase=Running -l component=stolon-keeper,instance=${INSTANCE} -o jsonpath="{.items[0].metadata.name}"
    ```
    {: codeblock}

    Replace ${INSTANCE} with the instance of the Assistant deployment that you want to back up.

1.  Fetch the store VCAP secret name.

    ```
    oc get secrets -l component=store${INSTANCE} -o=custom-columns=NAME:.metadata.name | grep store-vcap

    ```
    {: codeblock}

1.  Fetch the Postgres connection values. You will pass these values to the command that you run in the next step. (You must have `jq` installed.)

    - To get the username:

      ```
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.username'
      ```
      {: codeblock}

      where {.data.vcap_services} is the VCAP secret name that you retrieved in the previous step.

    - To get the password:

      ```
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.password'
      ```
      {: codeblock}

    - To get the database:

      ```
      oc get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | jq --raw-output '.["user-provided"][]|.credentials|.database'
      ```
      {: codeblock}

1.  Run the following command:

    ```
    oc exec $KEEPER_POD -- bash -c "export PGPASSWORD='$PASSWORD' && pg_dump -Fc -h localhost -d $DATABASE -U $USERNAME" > ${file-name}
    ```
    {: codeblock}

    The following lists describes the arguments. You retrieved the values for some of these parameters in the previous step:

    - `KEEPER_POD`: Any Postgres Keeper pod in your {{site.data.keyword.conversationshort}} instance.
    - `DATABASE`: The store database name.
    - `USERNAME`: Postgres user ID that can access the database.
    - `PASSWORD`: The password that corresponds with the Postgres user ID.
    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.

    To see more information about the `pg_dump` command, you can run this command:

    ```bash
    oc exec -it ${KEEPER_POD} -- pg_dump --help
    ```
    {: pre}
