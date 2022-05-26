---

copyright:
  years: 2015, 2022
lastupdated: "2022-05-26"

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

# Managing the cluster
{: #manage}

Manage the cluster nodes that host your {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull}} deployment.
{: shortdesc}

## Common tasks
{: #manage-common-tasks}

You can use Kubernetes commands to perform tasks that you cannot perform from the {{site.data.keyword.icp4dfull}} web client.

### To identify which nodes the product is deployed to
{: #manage-id-nodes}

```bash
kubectl get pods -o wide
```
{: codeblock}

### To find out how many replicas are in use
{: #manage-get-replica-number}

```bash
kubectl get deploy  -n {namespace-name}
```
{: pre}

or

```bash
kubectl get statefulset -n {namespace-name}
```
{: pre}

The response shows you name and number of replicas.

## Scaling
{: #manage-scale}

 Horizontal Pod Autoscaling (HPA) is enabled automatically for {{site.data.keyword.conversationshort}}. As a result, the number of replicas changes dynamically in the range of 1 to 10 replicas.

To use horizontal pod autoscalers in a deployment with OpenShift, you must install the OpenShift Container Platform metrics server. For more information, see [Requirements for Using Horizontal Pod Autoscalers](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html#req-for-using-hpas){: external}.

The following table describes the deployment details.

For 1.4.2, the `${release-name}` is hardcoded to `watson-assistant`. If a pod name becomes too long, the `${release-name}` is shortened to 10 characters.
{: note}

| Component name | Deployment name | Pod name | Default number of replicas |
|----------------|-----------------|----------|----------------------------|
| gateway | {release-name}-addon-assistant-gateway | {release-name}-addon-assistant-gateway-{pod-id} | 2-4 with HPA |
| dialog | {release-name}-dialog | {release-name}-dialog-{pod-id} | 2-10 with HPA |
| ed-mm | {release-name}-ed-mm | {release-name}-ed-mm-{pod-id} | 2-10 with HPA |
| master | {release-name}-master | {release-name}-master-{pod-id} | 2-10 with HPA |
| nlu | {release-name}-nlu | {release-name}-nlu-{pod-id} | 2-10 with HPA |
| recommends | {release-name}-recommends | {release-name}-recommends-{pod-id} | 2 |
| SIREG | {release-name}-sireg-de-tok-{version} | {release-name}-sireg-de-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-ja-tok-{version} | {release-name}-sireg-ja-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-ko-tok-{version} | {release-name}-sireg-ko-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-zhcn-tok-{version} | {release-name}-sireg-zhcn-tok-{pod-id} | 2 |
| Dialog skill | {release-name}-skill-conversation | {release-name}-skill-conversation-{pod-id} | 2-10 with HPA |
| Search skill | {release-name}-skill-search | {release-name}-skill-search-{pod-id} | 2-10 with HPA |
| Store | {release-name}-store | {release-name}-store-{pod-id} | 2-10 with HPA |
| PostgreSQL | {release-name}-store-postgres-proxy | {release-name}-store-postgres-proxy-{pod-id} | 2 |
| PostgreSQL | {release-name}-store-postgres-sentinel | {release-name}-store-postgres-sentinel-{pod-id} | 3 |
| TAS | {release-name}-tas | {release-name}-tas-{pod-id} | 2-10 with HPA |
| UI | {release-name}-ui | {release-name}-ui-{pod-id} | 2-10 with HPA |
{: caption="Deployment details" caption-side="top"}

The following table describes the stateful set details.

| Component name | Deployment name | Pod name | Default number of replicas |
|----------------|-----------------|----------|----------------------------|
| Minio | {release-name}-clu-minio | {release-name}-clu-minio-x | 4 |
| etcd | {release-name}-etcd | {release-name}-etcd-x | 5 |
| MongoDB server | {release-name}-mongodb-server | {release-name}-mongodb-server-x | 3 |
| Redis | {release-name}-redis-sentinel | {release-name}-redis-sentinel-x | 3 |
| Redis | {release-name}-redis-server | {release-name}-redis-server-x | 2 |
| PostgreSQL | {release-name}-store-postgres-keeper | {release-name}-store-postgres-keeper-x | 3 |
{: caption="Stateful set details" caption-side="top"}

### To scale the number of replicas:
{: #manage-scale-replicas}

Some of the microservices do not benefit from being scaled up; more replicas does not always mean more throughput.

- `etcd` cannot be scaled to more than 5 replicas by using the `kubectl scale statefulset` command.
- `master` triggers workspace trainings. Adding more master microservice replicas adds resiliency.
- `TAS` and `ed-mm` manage how many models are loaded and where they are loaded. More replicas might mean that a model can be loaded more times. However, unless high load is present, scaling too high does not help.
- `Minio` has a hardcoded number of replicas and cannot be scaled manually.
- `MongoDB` cannot be scaled manually.
- `Redis` can be scaled, but adding more redis-servers only improves resiliency to outages because only one of the servers is marked as coordinator and responds to requests.
- `Store` can be scaled up to a maximum of 10 replicas.
- `PostgreSQL` can be scaled, but you might reach a limit to the number of database connections that can be created.

1.  Disable Horizontal Pod Autoscaling (HPA). If you want to scale the deployment manually, you must disable HPA first or autoscaling will override any values you try to set manually.

    To disable HPA, you must delete the HPA resource.

    ```bash
    kubectl delete hpa NAME-OF-HPA
    ```
    {: pre}

1.  To increase the number to scale up or decrease the number to scale down, use one of these commands:

    ```bash
    kubectl scale deploy {pod-name} --replicas={number}
    ```
    {: pre}

    or

    ```bash
    kubectl scale statefulsets {set-name} --replicas={number}
    ```
    {: pre}

## Stopping and restarting the cluster
{: #manage-stop-restart}

You can manually scale the cluster down and back up or use a script to stop and restart the service.

### Use a script to stop and restart Watson Assistant
{: #manage-stop-restart-script}

Starting with 1.4.2, you can use the **wactl.sh** script to stop or restart the service.

To use the script to stop or restart the service, complete the following steps:

1.  From a machine that has Kubernetes access to your cluster, log in to your cluster and change to the correct namespace (project).
1.  Access the Helm chart from the file server on [Github](https://github.com/IBM/cloud-pak/tree/master/repo/cpd3/modules/ibm-watson-assistant/x86_64/1.4.2/){: external}.
1.  Unzip the Helm chart TGZ file so you can access the scripts that are provided in the service installation package.
1.  On the coordinator node, change to the **/path/to/ibm-watson-assistant-prod/ibm_cloud_pakpak_extensions/post-install/namespaceAdministration/** subdirectory.
1.  Run the `wactl.sh` script.

```
wactl.sh
  --action [stop | start | restart | clean]
  --release RELEASE
  [--cli kubectl | oc]
  [--include-ds]
```
{: codeblock}

The script accepts the following parameters:

- `action`: Required. Indicates the action you want to perform. Options include:

  - `stop`: Stops the service by scaling down the replicas in an order that limits possible dependency issues. This option also stores an annotation within the deployment or statefulset. If you subsequently try to use the script to start the service and the annotation doesn't exist in one of the deployments or statefulsets, the script fails.
  - `start`: You can only use this command to start the service if the `wactl` script was used to stop the service.
  - `restart`: Use this method to scale down the replicas, but to keep at least one replica for each pod running at all times to prevent a disruption in service.
  - `clean`: Removes the annotation that was created by the `stop` option from all objects including datastores.

- `release`: For 1.4.2, the release name is hardcoded as `watson-assistant`.
- `cli`: Specify the command line interface you are using.

  Specify `oc` for OpenShift and `kubectl` for Kubernetes.
- `include-ds`: Optional. Indicates that you want to perform the action on the data sources. When you specify this parameter with the `restart` action, all of the data sources are restarted except Redis.

### To manually scale the cluster all the way down and back
{: #manage-restart-replicas}

To scale down the cluster all the way, you must scale down the deployed services in the following order:

- **ui** deployment
- **store** deployment
- **gateway** deployment
- All other deployments
- All statefulsets

1.  If you have local storage, complete the following steps:

    - Connect over SSH to each worker node, and then run the following command from a given worker to synchronize data to the same directory on each other worker.

      The destination folders should be empty. If they're not, empty them.

      ```bash
      rsync -av /mnt/local-storage/storage/watson/assistant/{yourPV} {other worker}:/mnt/local-storage/storage/watson/assistant
      ```
      {: codeblock}

    - Double-check the **postgres-keeper** persistent volume permissions on each worker node.

      You might need to scale postgres-keeper deployments first to see which persistent volumes they claim. Connect over SSH to a worker node, and then run the following command:

      ```bash
      kubectl get pvc
      ```
      {: pre}

      For each `postgres-keeper` persistent volume claim, change the permissions of `/postgres` to `u=rwx (0700)` on its given worker node.

      ```bash
      chmod 0700 /mnt/local-storage/storage/watson/assistant/{yourPV}/postgres`
      ```
      {: pre}

1.  If you need to stop and restart the underlying OpenShift cluster for any reason, you can do so now.

    For more information, see [How To: Stop and start a production OpenShift Cluster](https://servicesblog.redhat.com/2019/05/29/how-to-stop-and-start-a-production-openshift-cluster/){: external}.

1.  Scale the service back up by scaling up the deployed services in the following order:

    - All statefulsets (except postgres-store-keeper)
    - **postgres-store-sentinel** deployment
    - **postgres-store-proxy** deployment
    - **postgres-store-keeper** statefulset
    - All deployments (except gateway, ui, and store)
    - **gateway** deployment
    - **store** deployment
    - **ui** deployment

## To view logs
{: #manage-view-logs}

Consider using [Analytics Dashboards](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/svc/cognos/dashboard/dashboard-svc.html){: external} to identify patterns in your data.

## Managing user access
{: #manage-add-users}

After you provision an instance, you can share the URL for the product user interface with other users. However, those users can only log in to the product user interface if you give them access.

If you plan to use SAML for single sign-on (SSO), complete [Configuring single sign-on](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.0?topic=client-configuring-sso){: external} before you add users. If you add users before you configure SSO, you will need to re-add the users with their SAML ID to enable them to use SSO.

1.  From the web client menu, click **Administer > Manage users**.

1.  Click **New user**, and specify the user's full name, user name, and email address. Set the user's permissions, and then click **Save**.

1.  From the web client menu, select **My Instances**.

1.  Find your {{site.data.keyword.conversationshort}} instance, hover over the last column to find and click the menu ![More menu](images/cp4d-sideways-kebab.png), and then choose **Manage Access**.

1.  Click **Add user**.

1.  Click the user name field to see a list of the people you can add.

    The users you added in the previous steps are listed. Select a name, choose **User** or **Admin** as their access role, and then click **Add**.

    If you aren't connecting to an existing user registry and enabling single sign-on, then temporary passwords are created for the users you add and are sent to them by way of the email addresses you specified.

1. {: #manage-add-users-to-disco}Repeat the access management steps on the {{site.data.keyword.discoveryshort}} instance.

   Before people can create search skills in {{site.data.keyword.conversationshort}}, they need to have access to a {{site.data.keyword.discoveryshort}} instance. Add to the {{site.data.keyword.discoveryshort}} instance those people who need to be able to add new data collections to or query from existing collections by using a search skill.
