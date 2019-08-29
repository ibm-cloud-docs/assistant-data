---

copyright:
  years: 2015, 2019
lastupdated: "2019-08-28"

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
{: #manage-130}

Manage the cluster nodes that host your {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull}} deployment.
{: shortdesc}

## Common tasks
{: #manage-130-common-tasks}

You can use Kubernetes commands to perform tasks that you cannot perform from the {{site.data.keyword.icp4dfull}} web client.

### To identify which nodes the product is deployed to
{: #manage-130-id-nodes}

```bash
kubectl get pods -o wide
```
{: codeblock}

### To find out how many replicas are in use
{: #manage-130-get-replica-number}

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
{: #manage-130-scale}

 Horizontal Pod Autoscaling (HPA) is enabled automatically for {{site.data.keyword.conversationshort}}. As a result, the number of replicas changes dynamically in the range of 1/2 to 10 replicas.

To use horizontal pod autoscalers in a deployment with OpenShift, you must install the OpenShift Container Platform metrics server. For details, see [Requirements for Using Horizontal Pod Autoscalers](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html#req-for-using-hpas){: external}.

The following table describes the deployment details.

| Component name | Deployment name | Pod name | Default number of replicas |
|----------------|-----------------|----------|----------------------------|
| add-on | {release-name}-addon-assistant-addon | {release-name}-addon-assistant-addon-{pod-id} | 2-4 with HPA |
| dialog | {release-name}-dialog | {release-name}-dialog-{pod-id} | 2-10 with HPA |
| ed-mm | {release-name}-ed-mm | {release-name}-ed-mm-{pod-id} | 2-10 with HPA |
| master | {release-name}-master | {release-name}-master-{pod-id} | 2-10 with HPA |
| nlu | {release-name}-nlu | {release-name}-nlu-{pod-id} | 2-10 with HPA |
| recommends | {release-name}-recommends | {release-name}-recommends-{pod-id} | 2 |
| SIREG | {release-name}-sireg-de-tok-{version} | {release-name}-sireg-de-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-ja-tok-{version} | {release-name}-sireg-ja-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-ko-tok-{version} | {release-name}-sireg-ko-tok-{pod-id} | 2 |
| SIREG | {release-name}-sireg-zhcn-tok-{version} | {release-name}-sireg-ko-tok-{pod-id} | 2 |
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
| etcd | {release-name}-etcd3 | {release-name}-etcd3-x | 3 |
| MongoDB server | {release-name}-mongodb-server | {release-name}-mongodb-server-x | 3 |
| Redis | {release-name}-redis-sentinel | {release-name}-redis-sentinel-x | 3 |
| Redis | {release-name}-redis-server | {release-name}-redis-server-x | 2 |
| PostgreSQL | {release-name}-store-postgres-keeper | {release-name}-store-postgres-keeper-x | 3 |
{: caption="Stateful set details" caption-side="top"}

### To scale the number of replicas:
{: #manage-130-scale-replicas}

Some of the microservices do not benefit from being scaled up; more replicas does not always mean more throughput. 
  
- `etcd` cannot be scaled to more than 3 replicas by using the `kubectl scale statefulset` command.
- `master` triggers workspace trainings. Adding more master microservice replicas adds resiliency. 
- `TAS` and `ed-mm` manage how many models are loaded and where they are loaded. More replicas might mean that a model can be loaded more times. However, unless high load is present, scaling too high does not help.
- `Minio` has a hard-coded number of replicas and cannot be scaled manually.
- `MongoDB` cannot be scaled manually. 
- `Redis` can be scaled, but adding more redis-servers only improves resiliency to outages because only one of the servers is marked as master and responds to requests.
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

### To scale the cluster all the way down and back
{: #manage-130-restart-replicas}

To scale down the cluster all the way, you must scale down the deployed services in the following order:

- **store** deployment
- **ui** deployment
- **addon/auth** deployment
- All other deployments
- All statefulsets

1.  If you have local storage, then connect over SSH to each worker node, and then run the following command from a given worker to synchronize data to the same directory on each other worker. 

    The destination folders should be empty. If they're not, empty them.

    ```bash
    rsync -av /mnt/local-storage/storage/watson/assistant/{yourPV} {other worker}:/mnt/local-storage/storage/watson/assistant
    ```
    {: pre}

1.  Double-check the **postgres-keeper** persistent volume permissions on each worker node. 

    You might need to scale postgres-keeper deployments first to see which persistent volumes they claim.

    ```bash
    kubectl get pvc
    ``` 
    {: pre}

    For each `postgres-keeper` persistent volume claim, change the permissions of `/postgres` to `u=rwx (0700)` on its given worker node.

    ```bash
    chmod 0700 /mnt/local-storage/storage/watson/assistant/{yourPV}/postgres`
    ```
    {: pre}

1.  Scale the service back up by scaling up the deployed services in the following order:

    - All statefulsets (except postgres-store-keeper)
    - **postgres-store-sentinel** deployment
    - **postgres-store-proxy** deployment
    - **postgres-store-keeper** statefulset
    - All deployments (except addon/auth, ui, and store)
    - **addon/auth** deployment
    - **ui** deployment
    - **store** deployment

## To view logs
{: #manage-130-view-logs}

{{site.data.keyword.icp4dfull_notm}} automatically logs information from each service. For more information, see [Viewing logs](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/admin/logs.html){: external}

Also see [Integrating with Grafana or Kibana dashboards](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/admin/admindash-integrate.html#admindash-integrate){: external}.

## Managing user access
{: #manage-130-add-users}

After you provision an instance, you can share the URL for the product user interface with other users. However, those users can only log in to the product user interface if you give them access.

If you plan to use SAML for single sign-on (SSO), complete [Configuring single sign-on](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/admin/saml-sso.html#saml-sso) before you add users. If you add users before you configure SSO, you will need to re-add the users with their SAML ID to enable them to use SSO.

1.  From the web client menu, click **Administer > Manage user**.

1.  Click **Add user**, and specify the user's full name, user name, and email address. Set the user's permissions, and then click **Add**.

1.  From the web client menu, select **My Instances**.

1.  Find your {{site.data.keyword.conversationshort}} instance, click the more menu, and then choose **Manage Access**.

1.  Click **Add user**.

1.  Click the user name field to see a list of the people you can add.

    The users you added in the previous steps are listed. Select a name, choose **User** or **Admin** as their access role, and then click **Add**. 

    If you aren't connecting to an existing user registry and enabling single sign-on, then temporary passwords are created for the users you add and are sent to them by way of the email addresses you specified.

1. {: #manage-add-users-to-disco}Repeat the access management steps on the {{site.data.keyword.discoveryshort}} instance. 

   Before people can create search skills in {{site.data.keyword.conversationshort}}, they need to have access to a {{site.data.keyword.discoveryshort}} instance. Add to the {{site.data.keyword.discoveryshort}} instance those people who need to be able to add new data collections to or query from existing collections by way of a search skill.

## Prepare your local machine to perform management tasks (V1.2 only)
{: #manage-130-prep-local-machine}

Prepare your local machine to perform cluster management tasks.

1.  Install the tools you will need on your local machine before you can complete management tasks from the command line.
 
    The following software and tools are available as part of {{site.data.keyword.icp4dfull_notm}} V2.1.0.0, which runs on top of {{site.data.keyword.icpfull_notm}} V3.1.2:

    - **Helm V2.9.1**: You need this software to run the `helm` commands that are used in this installation.

    - **{{site.data.keyword.icpfull_notm}} V3.1.2 Command Line Interface**: You need this CLI to run `cloudctl` commands.

    - **Kubernetes V1.12.4**: You need this software to run `kubectl` commands. 
    
      Technically, Kubernetes V1.10 is required, but a compatible version can also be used. V1.12 is provided with the cluster tools, so is easier to download and use.

1.  Go to this url to get the tools:

    ```bash
    https://{cluster4d-master-node}:8443/console/tools/cli
    ```
    {: pre}

1.  Download each tool to your local machine by using the curl command that is appropriate for your operating system. 

1.  After downloading the file, use these commands to get access to the file and move it to the right directory:

    ```bash
    chmod 755 {file downloaded via curl}
    sudo mv {file downloaded via curl} /usr/local/bin/{cloudctl | helm | kubernetes}
    ```
    {: pre}

1.  After you download the Helm software, run this command to start it:
  
    ```bash
    helm init --client-only
    ```
    {: pre}

1.  The tools are now installed. To check whether they are set up properly, run the following commands:

    - Test CLI

      ```bash
      cloudctl login -a https://{cluster4d-master-node}:8443 -u {admin user id} -p {admin password}
      ```
      {: pre}
    
      If you are using a load balancer, the hostname to specify here is the hostname of the load balancer instead of the master node.
      {: note}

    - Test Kubernetes

      ```bash
      kubectl get namespaces
      ```
      {: pre}

    - Test Helm

      ```bash
      helm version --tls
      ```
      {: pre}

     If you cannot run the kubectl command, complete this procedure to enable access to the Kubernetes CLI. See [Enabling access to the Kubernetes command-line interface](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/install/kubectl-access.html){: external}.
