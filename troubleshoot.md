---

copyright:
  years: 2015, 2021
lastupdated: "2021-10-27"

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

# Troubleshooting
{: #troubleshoot}

Get help with solving issues that you encounter while using the product.
{: shortdesc}

## 4.0.0
{: #troubleshoot-400}

### Install Watson Assistant 4.0.0 with EDB version 1.8
{: #troubleshoot-400-install-edb18}

Complete this task only if you need a fresh installation of Watson Assistant 4.0.0. Do not complete this task on existing clusters with data. Completing this task on an existing cluster with data results in data loss.
{: important}

If you upgraded to EDB version 1.8 and need a new installation of Watson Assistant 4.0.0, complete the following steps. In the following steps, `wa` is used as the name of the custom resource. Replace this value with the name of your custom resource:

1. First, apply the following patch:
    ```
    cat <<EOF | oc apply -f -
    apiVersion: assistant.watson.ibm.com/v1
    kind: TemporaryPatch
    metadata:
      name: wa-postgres-180-hotfix
    spec:
      apiVersion: assistant.watson.ibm.com/v1
      kind: WatsonAssistantStore
      name: wa     # Specify the name of your custom resource
      patchType: patchStrategicMerge
      patch:
        postgres:
          postgres:
            spec:
              bootstrap:
                initdb:
                  options:
                   - "--encoding=UTF8"
                   - "--locale=en_US.UTF-8"
                   - "--auth-host=scram-sha-256"
    EOF
    ```

1. Run the following command to check that the temporary patch is applied to the `WatsonAssistantStore` custom resource. It might take up to 10 minutes after the store custom resource is updated:
    ```
    oc get WatsonAssistantStore wa -o jsonpath='{.metadata.annotations.oppy\.ibm\.com/temporary-patches}' ; echo
    ```

    If the patch is applied, the command returns output similar to the following example:
    ```
    {"wa-postgres-180-hotfix": {"timestamp": "2021-09-23T15:48:12.071497", "api_version": "assistant.watson.ibm.com/v1"}}`
    ```

1. Run the following command to delete the Postgres instance:
    ```
    oc delete clusters.postgresql.k8s.enterprisedb.io wa-postgres
    ```

1. Wait 10 minutes. Then, run the following command to check the Postgres instance:
    ```
    oc get pods | grep wa-postgres
    ```

    When the Postgres instance is running properly, the command returns output similar to the following example:
    ```
    wa-postgres-1                                                     1/1     Running     0          37m
    wa-postgres-2                                                     1/1     Running     0          36m
    wa-postgres-3                                                     1/1     Running     0          36m
    ```

1. Run the following command to check that the jobs that initialize the Postgres database completed successfully:
    ```
    oc get jobs wa-create-slot-store-db-job wa-4.0.0-update-schema-store-db-job
    ```

    When the jobs complete successfully, the command returns output similar to the following example:
    ```
    NAME                                  COMPLETIONS   DURATION   AGE
    wa-create-slot-store-db-job           1/1           3s         31m
    wa-4.0.0-update-schema-store-db-job   1/1           13s        31m
    ```

    If the jobs don't complete successfully, then they timed out and need to be recreated. Delete the jobs by running the `oc delete jobs wa-create-slot-store-db-job wa-4.0.0-update-schema-store-db-job` command. The jobs are recreated after 10 minutes by the Watson Assistant operator.

For information about upgrading from Watson Assistant 4.0.0 to Watson Assistant 4.0.2, see [Upgrading Watson Assistant to a newer 4.0 refresh](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.0?topic=assistant-upgrading-watson-version-40){: external}.

## 1.5.0
{: #troubleshoot-150}

### Disable Horizontal Pod Autoscaling and set a maximum number of master pods
{: #troubleshoot-150-disable-hpa}

Horizontal Pod Autoscaling (HPA) is enabled automatically for Watson Assistant. As a result, the number of replicas changes dynamically in the range of 1 to 10 replicas. You can disable HPA if you want to limit the maximum number of master pods or if you're concerned about master pods being created and deleted too frequently.

1.  First, disable HPA for the `master` microservice by running the following command. In these steps, substitute your instance name for the `INSTANCE_NAME` variable:
    ```
    oc patch wa ${INSTANCE_NAME} --type='json' --patch='[{"op": "add", "path": "/appConfigOverrides/clu_master", "value":{"autoscaling":{"enabled":false}}}]'
    ```

1.  Wait until the information propagates into the Watson Assistant operator:
    ```
    sleep 600
    ```

1.  Run the following command to remove HPA for the `master` microservice
    ```
    oc delete hpa ${INSTANCE_NAME}-master
    ```

1.  Wait for about 30 seconds:
    ```
    sleep 30
    ```

1.  Finally, scale down the `master` microservice to the number of replicas that you want. In the following example, the `master` microservice is scaled down to two replicas:
    ```
    oc scale deploy ${INSTANCE_NAME}-master --replicas=2
    ```

### Watson Assistant 1.5.0 patch 1
[Watson Assistant 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164) is available for installations of version 1.5.0.

### Resizing the Redis statefulset memory and cpu values after applying patch 1 for Watson Assistant 1.5.0

Watson Assistant uses Redis to store web session-related data. Here are steps to resize Redis statefulset memory and cpu values after applying [Watson Assistant 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164).

1.  Use `oc get wa` to see your instance name:
    ```
    oc get wa
    NAME                       VERSION   READY   READYREASON   UPDATING   UPDATINGREASON   DEPLOYED   VERIFIED   AGE
    watson-assistant---wa-qa   1.5.0     True    Stable        False      Stable           18/18      18/18      11h
    ```

1.  Export your instance name as a variable that you can use in each step, for example:
    ```
    export INSTANCENAME=watson-assistant---wa-qa
    ```

1.  Change the `updateStrategy` in both Redis statefulsets to type `RollingUpdate`:
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
    oc patch statefulset c-$INSTANCENAME-redis-s -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
    ```

1.  Update the Redis statefulsets with the resized cpu and memory values:

    **Member CPU**
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/cpu", "value":"50m"}]'
    ```
    {: codeblock}

    **Member memory**
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/memory", "value":"256Mi"}]'
    ```
    {: codeblock}

    **Sentinel CPU**
    ```
    oc patch statefulset c-$INSTANCENAME-redis-s --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/cpu", "value":"50m"}]'
    ```
    {: codeblock}

    **Sentinel memory**
    ```
    oc patch statefulset c-$INSTANCENAME-redis-s --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/memory", "value":"256Mi"}]'
    ```
    {: codeblock}

1.  Confirm the Redis member and sentinel pods have the new memory and cpu values, for example:

      `oc describe pod c-$INSTANCENAME-redis-m-0 |grep cpu`

      `oc describe pod c-$INSTANCENAME-redis-m-0 |grep memory`

      `oc describe pod c-$INSTANCENAME-redis-s-0 |grep cpu`

      `oc describe pod c-$INSTANCENAME-redis-s-0 |grep memory`

      The results should look like these examples:
      ```
      oc describe sts c-$INSTANCENAME-redis-m |grep cpu
                            {"m":{"db":{"limits":{"cpu":"4","memory":"256Mi"},"requests":{"cpu":"25m","memory":"256Mi"}},"mgmt":{"limits":{"cpu":"2","memory":"100Mi"}...
            cpu:     4
            cpu:     50m
            cpu:     2
            cpu:     50m
            cpu:     2
            cpu:      50m
            cpu:     2
            cpu:     50m
      ```
      ```
      oc describe sts c-$INSTANCENAME-redis-m |grep memory
                            {"m":{"db":{"limits":{"cpu":"4","memory":"256Mi"},"requests":{"cpu":"25m","memory":"256Mi"}},"mgmt":{"limits":{"cpu":"2","memory":"100Mi"}...
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:   256Mi
            memory:  256Mi
            memory:  256Mi
      ```
      ```
      oc describe pod c-$INSTANCENAME-redis-s-0 |grep cpu
                      {"m":{"db":{"limits":{"cpu":"4","memory":"256Mi"},"requests":{"cpu":"25m","memory":"256Mi"}},"mgmt":{"limits":{"cpu":"2","memory":"100Mi"}...
            cpu:     2
            cpu:     50m
            cpu:     2
            cpu:     50m
            cpu:     2
            cpu:      50m
            cpu:     2
            cpu:     50m
      ```
      ```
      oc describe pod c-$INSTANCENAME-redis-s-0 |grep memory
                      {"m":{"db":{"limits":{"cpu":"4","memory":"256Mi"},"requests":{"cpu":"25m","memory":"256Mi"}},"mgmt":{"limits":{"cpu":"2","memory":"100Mi"}...
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:  256Mi
            memory:   256Mi
            memory:  256Mi
            memory:  256Mi
      ```

1. Change the `updateStrategy` in both Redis statefulsets back to type `OnDelete`:
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m -p '{"spec":{"updateStrategy":{"type":"OnDelete"}}}'
    oc patch statefulset c-$INSTANCENAME-redis-s -p '{"spec":{"updateStrategy":{"type":"OnDelete"}}}'
    ```

### Delete the pdb (poddisruptionbudgets) when changing instance from medium to small
{: #troubleshoot-delete-pdb}

Whenever the size of Watson Assistant is changed from medium to small, a manual step is required to delete the `poddisruptionbudgets` that are created for medium instances.

Run the following command, replacing `<instance-name>` with the name of your Watson Assistant CR instance and replacing `<namespace-name>` with the name of the namespace where the instance resides.

```
oc delete pdb  -l icpdsupport/addOnId=assistant,component!=etcd,ibmevents.ibm.com/kind!=Kafka,app.kubernetes.io/instance=<instance-name> -n <namespace-name>

```

## 1.4.2
{: #troubleshoot-142}

### Cannot provision an instance, and service images are missing from the catalog
{: #troubleshoot-142-missing-label}

If you run the installation with no errors, but cannot provision an instance, check whether the product icon is visible in the service tile. From the {{site.data.keyword.icp4dfull_notm}} web client, go to the *Services* page.

  ![Services icon](images/cp4d-services-icon.png)

1.  Find the {{site.data.keyword.conversationshort}} service tile. Check whether the product logo (![Watson Assistant logo](images/assistant-icon.png)) is displayed on the tile.

    ![Watson Assistant service tile](images/missing-icon.png)

1.  If the logo is missing, it is likely that you missed the step in the installation process where you label the namespace.

    - **3.0.1**: See [Step 4: Add the cluster namespace label to your service namespace](/docs/assistant-data?topic=assistant-data-install-142#install-142-cpd30-apply-namespace-label).
    - **2.5**: See: [Step 4: Add the cluster namespace label to your service namespace](/docs/assistant-data?topic=assistant-data-install-142#install-142-cpd25-apply-namespace-label).

### Getting an error when using v2 `/message` API
{: #troubleshoot-142-v2-error}
<!--issue 44398-->

- Problem: When calling the [`/message` v2 API](https://cloud.ibm.com/apidocs/assistant-data-v2#message){: external} to send user input to your assistant, the following error is returned: `Unable to query assistant metadata`.
- Cause: A database race condition occurs when too many Postgres heartbeat calls are sent.
- Solution: To resolve the issue, turn off the `DB_USE_HEARTBEAT` environment setting on the store pods.

  You can use the following command to change the setting:

  ```
  oc set env deploy watson-assistant-store DB_USE_HEARTBEAT=false
  ```
  {: codeblock}

### Getting an error when importing a skill
{: #troubleshoot-142-import-error}
<!--issue 44309-->

- Problem: An error is displayed when importing a skill that was exported from a different environment, where both environments are running {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull_notm}} 1.4.2. The error says, `Error - Internal`.
- Cause: The new environment does not have enough memory dedicated to the keeper pods. For example, the new environment sets the keeper pod memory setting to 256 MB, but the development environment that the skill is being exported from sets the keeper pod memory setting to 1 Gi.
- Solution: Increase the keeper pod memory in the environment where the skill is being imported to match that of the environment from which the skill is being exported.
