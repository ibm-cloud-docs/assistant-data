---

copyright:
  years: 2015, 2020
lastupdated: "2020-04-08"

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

To back up the data, you can use a tool that Postgres provides that is called `pg_dump`. The dump tool creates a backup by sending the database contents to stdout where you can write it to a file. 

A bash script is provided in the product's PPA file. The script gathers the pod name and credentials for one of your Postgres Proxy pods, which is the pod from which the `pg_dump` command must be run, and then runs the command for you.

## Important considerations

- When you create a backup by using this procedure, the backup includes all of the assistants and skills from all of the service instances. Meaning it can include even skills and assistants to which you do not have access.
- The access permissions information of the original service instances is not stored in the backup. Meaning original access rights, which determine who can see a service instance and who cannot, are not preserved. 
- The target {{site.data.keyword.icp4dfull_notm}} cluster where you restore the data must have the same number of instances as the environment from which you back up the database.
- The tool that restores the data clears the current database before it restores the backup. Therefore, if you might need to revert to the current database, create a backup of it first.
- If you back up and restore or otherwise change the {{site.data.keyword.discoveryshort}} service that your search skill connects to, then you cannot retore the search skill, but must recreate it. When you set up a search skill, you map sections of the assistant's response to fields in a data collection that is hosted by an instance of {{site.data.keyword.discoveryshort}} on the same cluster. If the {{site.data.keyword.discoveryshort}} instance changes, your mapping to it is broken. If your {{site.data.keyword.discoveryshort}} service does not change, then the search skill can continue to connect to the data collection.

## Backing up data by using the script
{: #backup-os}

To back up data by using the provided script, complete the following steps:

1.  Log in to the OpenShift project namespace or Kubernetes namespace where you installed the product.
1.  Go to the directory where the `backupPG.sh` script is stored, which is `{compressed-file-dir}/charts/ibm-watson-assistant-prod/ibm_cloud_pak/pak_extensions/post-install/namespaceAdministration` where `{compressed-file-dir}` is the name of the directory where you extracted the downloaded PPA file.

1.  Run the script by using the following command:

    ```
    ./backupPG.sh [--release ${release-name}] > ${file-name}
    ```
    {: codeblock}

    where these are the arguments:

    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.
    - `--release ${release-name}`: Targets a specific release. Otherwise, the script backs up the first release it finds in the namespace you are logged in to.

If you prefer to back up data by using the Postgres tool directly, you can complete the procedure to back up data manually.

## Backing up data manually
{: #backup-cp4d}

Complete the steps in this procedure to back up your data by using the Postgres tool directly. 

If you have an OpenShift cluster, replace all `kubectl` commands with `oc` commands.
{: note}

To back up your data, complete these steps:

1.  Fetch a running Postgres proxy pod.

    ```
    kubectl get pods --field-selector=status.phase=Running -l component=stolon-proxy,release=${release-name} -o jsonpath="{.items[0].metadata.name}"
    ```
    {: codeblock}

    Replace ${release-name} with the release name for the deployment that you want to back up.

    Postgres pods are prefixed by `store-postgres`.

1.  Fetch the store VCAP secret name.

    ```
    kubectl get secrets -l component=store,release=${release-name} -o=custom-columns=NAME:.metadata.name | grep store-vcap
    ```
    {: codeblock}

1.  Fetch the Postgres connection values. You will pass these values to the command that you run in the next step.

    - To get the username:

      ```
      kubectl get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | grep -o '"username":"[^"]*' | cut -d'"' -f4
      ```
      {: codeblock}

      where {.data.vcap_services} is the VCAP secret name that you retrieved in the previous step.

    - To get the password:

      ```
      kubectl get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | grep -o '"password":"[^"]*' | cut -d'"' -f4
      ```
      {: codeblock}

    - To get the database:

      ```
      kubectl get secret $VCAP_SECRET_NAME -o jsonpath="{.data.vcap_services}" | base64 --decode | grep -o '"database":"[^"]*' | cut -d'"' -f4
      ```
      {: codeblock}

1.  Run the following command:

    ```
    kubectl exec $PROXY_POD -- bash -c "export PGPASSWORD='$PASSWORD' && pg_dump -Fc -h localhost -d $DATABASE -U $USERNAME" > ${file-name}
    ```
    {: codeblock}

    The following lists describes the arguments. You retrieved the values for some of these parameters in the previous step:

    - `PROXY_POD`: Any Postgres Proxy pod in your {{site.data.keyword.conversationshort}} Helm release.
    - `DATABASE`: The store database name.
    - `USERNAME`: Postgres user ID that can access the database.
    - `PASSWORD`: The password that corresponds with the Postgres user ID.
    - `${file-name}`: Specify a file where you want to write the downloaded data. Be sure to specify a backup directory in which to store the file. For example, `/bu/store.dump` to create a backup directory named `bu`. This directory will be referenced later as `$BACKUP-DIR`.

    To see more information about the `pg_dump` command, you can run this command:

    ```bash
    kubectl exec -it ${PROXY_POD} -- pg_dump --help
    ```
    {: pre}
 
## Restoring data
{: #backup-restore}

IBM created a restore tool called `pgmig`. The tool restores your database backup by adding it to a database you choose. It also upgrades the schema to the one that is associated with the version of the product where you restore the data.

Before it adds the backed-up data, the tool removes the data for all instances in the current service deployment, so any spares are removed also.
{: important}

1.  Install the target {{site.data.keyword.icp4dfull_notm}} cluster to which you want to restore the data. From the web client for the target cluster, create one service instance of {{site.data.keyword.conversationshort}} for each service instance that was backed up on the old cluster.

    The target {{site.data.keyword.icp4dfull_notm}} cluster must have the same number of instances as there were in the environment where you backed up the database.

1.  Back up the current database before you replace it with the backed-up database.

    The tool clears the current database before it restores the backup. So, if you might need to revert to the current database, be sure to create a backup of it first.

1.  Go to the backup directory that you specified in the file-name parameter in the previous procedure.

1.  Download the `pgmig` tool from the [GitHub Watson Developer Cloud Community](https://github.com/watson-developer-cloud/community/blob/master/watson-assistant/data) repository.

    ```
    wget https://github.com/watson-developer-cloud/community/blob/master/watson-assistant/data/1.4.0/pgmig
    ```
    {: codeblock}

    ```
    chmod 755 pgmig
    ```
    {: codeblock}

1.  Create two configuration files, and store them in the same backup directory.

    - **resourceController.yaml**: The Resource Controller file keeps a list of all provisioned {{site.data.keyword.conversationshort}} instances. See [Creating the resourceController.yaml file](#backup-resource-controller-yaml).

    - **postgres.yaml**: The Postgres file lists details for the target Postgres pods. See [Creating the postgres.yaml file](#backup-postgres-yaml).

1.  Copy the files that you downloaded and created in the previous steps into an existing directory of your choice on a Postgres Proxy pod. The files that you need to copy are `pgmig`, `postgres.yaml`, `resourceController.yaml`, and `store.dump`. 

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

1.  Initiate the execution of a remote command in the Proxy Pod.

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
    oc get secret ${release-name}-store-vcap -o jsonpath='{.data.vcap_services}' | base64 -d

    ```
    {: codeblock}

    The following information is returned:

    ```
    {
      "user-provided":[
      {
        "name":"pgservice",
        "label":"user-provided",
        "credentials":
          {
            "host":"${release-name}-store-postgres-proxy-svc",
            "port":5432,
            "database":"conversation_icp_${release-name}",
            "username":"store_icp_${release-name}",
            "password":"XX="
          }
        }
      ]
    }
    ```
    {: codeblock}

1.  Copy the values for user-provided credentials `host`, `port`, `database`, and `username`. 

    For example, in this sample the values are:
    
    ```yaml
    host: ${release-name}-store-postgres-proxy-svc
    port: 5432
    database: conversation_icp_${release-name}
    username: store_icp_${release-name}
    ```

1.  To get the value of `su_username`, you need to get details for the postgres keeper pods:

    To get the keeper pod names, use the following command:

    ```bash
    oc get pods -o=custom-columns=NAME:.metadata.name -l component=stolon-keeper,release=${release-name}
    ```
    {: codeblock}

    Get information about the pod.

    ```bash
    oc describe pod ${KEEPER_POD}
    ```
    {: codeblock}

    For example: 

    ```bash
    oc describe pod ibm-assistant-ts-z781-keeper-0
    ```
    {: codeblock}

    Look for this section:

    ```
    Environment:
    ...
    STKEEPER_PG_SU_USERNAME: admin
    ```
    {: screen}

    For example, you can use a command like this to find it:

    ```bash
    oc describe pod ibm-assistant-ts-z781-keeper-0 | grep STKEEPER_PG_SU_USERNAME
    ```
    {: codeblock}

    The value of `STKEEPER_PG_SU_USERNAME` is the `su_username`. Copy the username and add it to the YAML file.

1.  To get the `su_password`, you must get the postgres secret. 

    ```bash
    oc get secret ${release-name}-postgres-secret -o jsonpath={.data.pg_su_password} | base64 -d
    ```
    {: codeblock}

    Copy the password and add it to the YAML file.

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
