---

copyright:
  years: 2015, 2023
lastupdated: "2023-05-10"

subcollection: assistant-data

---

{{site.data.keyword.attribute-definition-list}}

Documentation about **{{site.data.keyword.assistant_classic_full}} for {{site.data.keyword.icp4dfull}}** has moved. For the most up-to-date version, see [Troubleshooting known issues for for {{site.data.keyword.icp4dfull_notm}}](/docs/watson-assistant?topic=watson-assistant-troubleshoot-data){: external}.
{: attention}

# Troubleshooting known issues
{: #troubleshoot}

Get help with solving issues that you might encounter while using {{site.data.keyword.assistant_classic_short}}.
{: shortdesc}

## 4.5.x
{: #troubleshoot-45x}

### Pod "RESTARTS" count stays at 0 after a 4.5.x upgrade even though a few assistant pods are restarting
{: #troubleshoot-453-restarts-zero}


- Problem: After upgrading {{site.data.keyword.assistant_classic_short}}, the pod "RESTARTS" count stays at 0 even though certain assistant pods are restarting.
- Cause: During the upgrade, custom resources owned by {{site.data.keyword.assistant_classic_short}} for the "certificates.certmanager.k8s.io" CRD are deleted using a script that runs in the background. Sometimes the CR deletion script completes before the assistant operator gets upgraded. In that case, the old assistant operator might recreate custom resources for the "certificates.certmanager.k8s.io" CRD. Leftover CRs might cause the certificate manager to continuously regenerate some certificate secrets, causing some assistant pods to restart recursively.
- Solution: Run the following script to delete leftover custom resources for the "certificates.certmanager.k8s.io" CRD after setting INSTANCE (normally `wa`) and PROJECT_CPD_INSTANCE variables:
```
for i in `oc get certificates.certmanager.k8s.io -l icpdsupport/addOnId=assistant --namespace ${PROJECT_CPD_INSTANCE} | grep "${INSTANCE}-"| awk '{print $1}'`; do oc delete certificates.certmanager.k8s.io $i --namespace ${PROJECT_CPD_INSTANCE}; done
```

## 4.5.0
{: #troubleshoot-450}

### Data Governor not healthy after installation
{: #troubleshoot-450-data-governor-not-healthy}

After {{site.data.keyword.assistant_classic_short}} is installed, the `dataexhausttenant` custom resource named `wa-data-governor-ibm-data-governor-data-exhaust-internal` gets stuck in the `Topics` phase. When this happens, errors in the Data Governor pods report that the service does not exist.

1. Get the status of the `wa-data-governor` custom resource:
    ```bash
    oc get DataExhaust
    ```
    {: codeblock}

1. Wait for the `wa-data-governor` custom resource to be in the `Completed` phase:
    ```
    NAME               STATUS      VERSION   COMPLETED
    wa-data-governor   Completed   master    1s
    ```
    {: codeblock}

1. Pause the reconciliation of the `wa-data-governor` custom resource:
    ```bash
    oc patch dataexhaust wa-data-governor -p '{"metadata":{"annotations":{"pause-reconciliation":"true"}}}' --type merge
    ```
    {: codeblock}

1. Apply the fix to the `dataexhausttenant` custom resource:
    ```bash
    oc patch dataexhausttenant wa-data-governor-ibm-data-governor-data-exhaust-internal  -p '{"spec":{"topics":{"data":{"replicas": 1}}}}' --type merge
    ```
    {: codeblock}

1. Wait for the Data Governor pods to stop failing. You can restart the admin pods to speed up this process.

1. Continue the reconciliation of the `wa-data-governor` custom resource:
    ```bash
    oc patch dataexhaust wa-data-governor --type=json -p='[{"op": "remove", "path": "/metadata/annotations/pause-reconciliation"}]'
    ```
    {: codeblock}

### RabbitMQ gets stuck in a loop after several installation attempts
{: #troubleshoot-450-rabbitmq-stuck}

After an initial installation or upgrade failure and repeated attempts to retry, the common services RabbitMQ operator pod can get into a `CrashLoopBackOff` state. For example, the log might include the following types of messages:
```
"error":"failed to upgrade release: post-upgrade hooks failed: warning:
Hook post-upgrade ibm-rabbitmq/templates/rabbitmq-backup-labeling-job.yaml
failed: jobs.batch "{%name}-ibm-rabbitmq-backup-label" already exists"
```
{: codeblock}

Resources for the `ibm-rabbitmq-operator.v1.0.11` component must be removed before a new installation or upgrade is started. If too many attempts occur in succession, remaining resources can cause new installations to fail.

1. Delete the RabbitMQ backup label job from the previous installation or upgrade attempt. Look for the name of the job in the logs. The name ends in `-ibm-rabbitmq-backup-label`:
    ```bash
    oc delete job {%name}-ibm-rabbitmq-backup-label -n ${PROJECT_CPD_INSTANCE}
    ```
    {: codeblock}

1. Check that the pod returns a `Ready` state:
    ```bash
    oc get pods -n ibm-common-services | grep ibm-rabbitmq
    ```
    {: codeblock}

### Preparing to install a size large {{site.data.keyword.assistant_classic_short}} deployment
{: #troubleshoot-450-prepare-large-install}

If you specify `large` for the `watson_assistant_size` option when you install {{site.data.keyword.assistant_classic_short}}, the installation fails to complete successfully.

Before you install a size large deployment of {{site.data.keyword.assistant_classic_short}}, apply the following fix. The following fix uses `wa` as the name of the Watson Assistant instance and `cpd` as the namespace where Watson Assistant is installed. These values are set in environment variables. Before you run the command, update the `INSTANCE` variable with the name of your instance, and update the `NAMESPACE` variable with the namespace where your instance in installed:
```bash
INSTANCE=wa ; \
NAMESPACE=cpd ; \
#DRY_RUN="--dry-run=client --output=yaml"     # To apply the changes, change to an empty string
DRY_RUN=""
cat <<EOF | tee wa-network-policy-base.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  annotations:
    oppy.ibm.com/internal-name: infra.networkpolicy
  labels:
    app: ${INSTANCE}-network-policy
    app.kubernetes.io/instance: ${INSTANCE}
    app.kubernetes.io/managed-by: Ansible
    app.kubernetes.io/name: watson-assistant
    component: network-policy
    icpdsupport/addOnId: assistant
    icpdsupport/app: ${INSTANCE}-network-policy
    icpdsupport/ignore-on-nd-backup: "true"
    icpdsupport/serviceInstanceId: inst-1
    service: conversation
    slot: ${INSTANCE}
    tenant: PRIVATE
    velero.io/exclude-from-backup: "true"
  name: ${INSTANCE}-network-policy
  namespace: ${NAMESPACE}
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          service: conversation
          slot: ${INSTANCE}
    - podSelector:
        matchLabels:
          slot: global
    - podSelector:
        matchLabels:
          component: watson-gateway
    - podSelector:
        matchLabels:
          component: dvt
    - podSelector:
        matchLabels:
          dwf_service: ${INSTANCE}-clu
          network-policy: allow-egress
    - podSelector:
        matchLabels:
          app: 0020-zen-base
    - namespaceSelector:
        matchLabels:
          ns: ${NAMESPACE}
      podSelector:
        matchLabels:
          app: 0020-zen-base
    - podSelector:
        matchLabels:
          component: ibm-nginx
    - namespaceSelector:
        matchLabels:
          ns: ${NAMESPACE}
      podSelector:
        matchLabels:
          component: ibm-nginx
    - namespaceSelector:
        matchLabels:
          assistant.watson.ibm.com/role: operator
      podSelector:
        matchLabels:
          release: assistant-operator
    - namespaceSelector:
        matchLabels:
          assistant.watson.ibm.com/role: operator
      podSelector:
        matchLabels:
          app: watson-assistant-operator
    - namespaceSelector:
        matchLabels:
          assistant.watson.ibm.com/role: operator
      podSelector:
        matchLabels:
          app.kubernetes.io/instance: watson-assistant-operator
    - namespaceSelector:
        matchLabels:
          assistant.watson.ibm.com/role: operator
      podSelector:
        matchLabels:
          app.kubernetes.io/instance: ibm-etcd-operator-release
    - namespaceSelector:
        matchLabels:
          assistant.watson.ibm.com/role: operator
      podSelector:
        matchLabels:
          app.kubernetes.io/instance: ibm-etcd-operator
  podSelector:
    matchLabels:
      service: conversation
      slot: ${INSTANCE}
  policyTypes:
  - Ingress
EOF
for MICROSERVICE in analytics clu-embedding clu-serving clu-training create-slot-job data-governor dialog dragonfly-clu-mm ed es-store etcd integrations master nlu recommends sireg-ubi-ja-tok-20160902 sireg-ubi-ko-tok-20181109 spellchecker-mm store store-admin store-cronjob store-sync store_db_creator store_db_schema_updater system-entities tfmm ui ${INSTANCE}-redis webhooks-connector
do
  # Change name and add component to selector
  # Apply to the cluster
  cat wa-network-policy-base.yaml | \
    oc patch --dry-run=client --output=yaml -f - --type=merge --patch "{
        \"metadata\": {\"name\": \"${INSTANCE}-network-policy-$( echo $MICROSERVICE | tr _ -)\"},
        \"spec\": {\"podSelector\":{\"matchLabels\":{\"component\": \"${MICROSERVICE}\"}}}
      }" |
    oc apply -f - ${DRY_RUN}
done
```
{: codeblock}

### Fixing a size large {{site.data.keyword.assistant_classic_short}} installation
{: #troubleshoot-450-fix-large-install}

Apply this fix if you installed a size `large` {{site.data.keyword.assistant_classic_short}} deployment, and your installation fails to complete successfully. In some cases, {{site.data.keyword.assistant_classic_short}} pods aren't able to communicate with other pods, and the Transmission Control Protocol (TCP) connections can't be established.

To confirm whether you are affected by this issue, run the following command:
```bash
oc logs --selector app=sdn --namespace openshift-sdn --container sdn | grep "Ignoring NetworkPolicy"
```
{: codeblock}

If you are affected, you see output similar to the following example:
```
W0624 12:58:21.407901    2480 networkpolicy.go:484] Ignoring NetworkPolicy cpd/wa-network-policy because it generates
```
{: codeblock}

If you encounter this error, apply the following fix to resolve the issue. The following fix uses `wa` as the name of the Watson Assistant instance. This value is set in an environment variable. Before you run the command, update the `INSTANCE` variable with the name of your instance:
```bash
INSTANCE=wa ; \
#DRY_RUN="--dry-run=client --output=yaml"     # To apply the changes, change to an empty string
DRY_RUN=""
for MICROSERVICE in analytics clu-embedding clu-serving clu-training create-slot-job data-governor dialog dragonfly-clu-mm ed es-store etcd integrations master nlu recommends sireg-ubi-ja-tok-20160902 sireg-ubi-ko-tok-20181109 spellchecker-mm store store-admin store-cronjob store-sync store_db_creator store_db_schema_updater system-entities tfmm ui ${INSTANCE}-redis webhooks-connector
do
  # Get original networking policy
  # Clean up metadata fields to get the resource applied by Watson Assistant
  # Change name and add component to selector
  # Apply to the cluster
  oc get networkpolicy $INSTANCE-network-policy --output yaml | \
  oc patch --dry-run=client --output=yaml -f - --type=json --patch='[
      {"op":"remove", "path":"/metadata/creationTimestamp"},
      {"op":"remove", "path":"/metadata/generation"},
      {"op":"remove", "path":"/metadata/resourceVersion"},
      {"op":"remove", "path":"/metadata/uid"},
      {"op":"remove", "path":"/metadata/annotations/kubectl.kubernetes.io~1last-applied-configuration"},
      {"op":"remove", "path":"/metadata/ownerReferences"}
    ]' | \
    oc patch --dry-run=client --output=yaml -f - --type=merge --patch "{
        \"metadata\": {\"name\": \"${INSTANCE}-network-policy-$( echo $MICROSERVICE | tr _ -)\"},
        \"spec\": {\"podSelector\":{\"matchLabels\":{\"component\": \"${MICROSERVICE}\"}}}
      }" |
    oc apply -f - ${DRY_RUN}
done
```
{: codeblock}

### Unable to scale the size of Redis pods
{: #troubleshoot-450-scale-redis}

When you scale the deployment size of {{site.data.keyword.assistant_classic_short}}, the Redis pods do not scale correctly and will not match the new size of the {{site.data.keyword.assistant_classic_short}} deployment (`small`, `medium`, or `large`). This problem is a known issue in Redis operator v1.5.1.

When you scale the size of your {{site.data.keyword.assistant_classic_short}} deployment, you must delete the Redis custom resource. This allows Redis to automatically re-create the custom resource with the correct size and pods.

To delete and re-create the Redis custom resource:

1. Source the environment variables from the script:
    ```bash
    source ./cpd_vars.sh
    ```
    {: codeblock}

    If you don't have the script that defines the environment variables, see [Setting up installation environment variables](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=information-setting-up-installation-environment-variables){: external}.

1. Export the name of the Redis custom resource to an environment variable:
    ```bash
    export REDIS_CR_NAME=`oc get redissentinels.redis.databases.cloud.ibm.com -l icpdsupport/addOnId=assistant -n ${PROJECT_CPD_INSTANCE} | grep -v NAME| awk '{print $1}'`
    ```
    {: codeblock}

1. Delete the Redis custom resource:
    ```bash
    oc delete redissentinels.redis.databases.cloud.ibm.com ${REDIS_CR_NAME} -n ${PROJECT_CPD_INSTANCE}
    ```
    {: codeblock}

    It might take approximately 5 minutes for the custom resource to be re-created.

1. Verify that Redis is running:
    ```bash
    oc get redissentinels.redis.databases.cloud.ibm.com -n ${PROJECT_CPD_INSTANCE}
    ```
    {: codeblock}

1. Export the name of your {{site.data.keyword.assistant_classic_short}} instance as an environment variable:
    ```bash
    export INSTANCE=`oc get wa -n ${PROJECT_CPD_INSTANCE} |grep -v NAME| awk '{print $1}'`
    ```
    {: codeblock}

1. Delete the Redis analytics secret:
    ```bash
    oc delete secrets ${INSTANCE}-analytics-redis
    ```
    {: codeblock}

1. Delete the `${INSTANCE}-analytics` deployment pods.

## 4.0.x
{: #troubleshoot-40x}

### Data Governor error causing deployment failure
{: #troubleshoot-40x-data-governor-deployment-fail}

The following fix applies to {{site.data.keyword.assistant_classic_short}} 4.0.0 through 4.0.8. In some cases, {{site.data.keyword.assistant_classic_short}} deployment is stuck and pods are not coming up because of an issue with the interaction between the Events operator and the Data Governor custom resource (CR).  

Complete the following steps to determine whether you are impacted by this issue and, if necessary, apply the patch to resolve it:

1. To determine whether you are impacted, run the following command to see whether the CR was applied successfully:
    ```sh
    oc get dataexhaust wa-data-governor -n $OPERAND_NS -o yaml
    ```
    {: pre}

    If you do not receive any error, then you do not need to apply the patch. If you receive an error similar to the following example, complete the next step to apply the patch:
    ```text
    message: 'Failed to create object: b''{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Internal
          error occurred: replace operation does not apply: doc is missing path: /metadata/labels/icpdsupport/serviceInstanceId:
          missing value","reason":"InternalError","details":{"causes":[{"message":"replace
          operation does not apply: doc is missing path: /metadata/labels/icpdsupport/serviceInstanceId:
          missing value"}]},"code":500}\n'''
        reason: Failed
        status: "True"
        type: Failure
      ibmDataGovernorService: InProgress
    ```
    {: codeblock}

1. From your operand namespace, run the following command to apply the patch. In the command, `wa` is used as the name of the instance. Replace this value with the name of your {{site.data.keyword.assistant_classic_short}} instance:

    ```
    cat <<EOF | oc apply -f -
    apiVersion: assistant.watson.ibm.com/v1
    kind: TemporaryPatch
    metadata:
      name: wa-data-governor
    spec:
        apiVersion: assistant.watson.ibm.com/v1
        kind: WatsonAssistant
        name: wa     # Replace wa with the name of your Watson Assistant instance
        patch:
          data-governor:
            dataexhaust:
              spec:
                additionalLabels:
                  icpdsupport/serviceInstanceId: inst-1
          kafkauser:
            metadata:
              labels:
                icpdsupport/serviceInstanceId: inst-1
    patchType: patchStrategicMerge
    EOF
    ```
    {: codeblock}

    Wait about 15 minutes for the changes to take effect.

1. Validate that the patch was applied successfully:
    ```sh
    oc get dataexhaust wa-data-governor -n $OPERAND_NS -o yaml
    ```
    {: pre}

    The patch was applied successfully when the value of `serviceInstanceId` is `inst-1`:
    ```text
    spec:
      additionalLabels:
        icpdsupport/serviceInstanceId: inst-1
    ```
    {: codeblock}

### Security context constraint permission errors
{: #troubleshoot-40x-scc-permission-error}

The following fix applies to {{site.data.keyword.assistant_classic_short}} 4.0.0 through 4.0.5. If a cluster has a security context constraint (SCC) that takes precedence over **restricted** SCCs and has different permissions than **restricted** SCCs, then {{site.data.keyword.assistant_classic_short}} 4.0.0 through 4.0.5 installations might fail with permission errors. For example, the `update-schema-store-db-job` job reports errors similar to the following example:
```
oc logs wa-4.0.2-update-schema-store-db-job-bpsdr postgres-is-prepared
Waiting until postgres is running and responding
psql: error: could not read root certificate file "/tls/ca.crt": Permission denied
    - The basic command to postgres failed (retry in 5 sec)
psql: error: could not read root certificate file "/tls/ca.crt": Permission denied
    - The basic command to postgres failed (retry in 5 sec)
psql: error: could not read root certificate file "/tls/ca.crt": Permission denied
    - The basic command to postgres failed (retry in 5 sec)
..
..
```

Other pods might have similar permission errors. If you look at the SCCs of the pods, you can see they are not restricted. For example, if you run the `oc describe pod wa-etcd-0 |grep scc` command, you get an output similar to the following example:
```
openshift.io/scc: fsgroup-scc
```

To fix this issue, raise the priority of the **restricted** SCC so that it takes precedence. To do this, complete the following steps:

1. Run the following command:
    ```
    oc edit scc restricted
    ```

1. Change the `priority` from `null` to `1`.

Now, new {{site.data.keyword.assistant_classic_short}} pods default back to the expected **restricted** SCC. When you run the `oc describe pod wa-etcd-0 |grep scc` command, you get an output similar to the following example:
```
openshift.io/scc: restricted
```

### Unable to collect logs with a webhook
{: #troubleshoot-40x-collect-logs-webhook}

The following fix applies to all versions of {{site.data.keyword.assistant_classic_short}} 4.0.x. If you're unable to collect logs with a webhook, it might be because you are using a webhook that connects to a server that is using a self-signed certificate. If so, complete the following steps to import the certificate into the keystore so that you can collect {{site.data.keyword.assistant_classic_short}} logs with a webhook:

1. Log in to cluster and `oc project cpd-instance`, which is the namespace where the {{site.data.keyword.assistant_classic_short}} instance is located.

1. Run the following command. In the following command, replace `INSTANCE_NAME` with the name of your {{site.data.keyword.assistant_classic_short}} instance and replace `CUSTOM_CERTIFICATE` with your Base64 encoded custom certificate key:
    ```
    INSTANCE="INSTANCE_NAME"     # Replace INSTANCE_NAME with the name of the Watson Assistant instance
    CERT="CUSTOM_CERTIFICATE"     # Replace CUSTOM_CERTIFICATE with the custom certificate key

    cat <<EOF | oc apply -f -
    apiVersion: v1
    data:
      ca_cert: ${CERT}
    kind: Secret
    metadata:
      name: ${INSTANCE}-custom-webhooks-cert
    type: Opaque
    ---
    apiVersion: assistant.watson.ibm.com/v1
    kind: TemporaryPatch
    metadata:
      name: ${INSTANCE}-add-custom-webhooks-cert
    spec:
      apiVersion: assistant.watson.ibm.com/v1
      kind: WatsonAssistantStore
      name: ${INSTANCE}
      patchType: patchStrategicMerge
      patch:
        webhooks-connector:
          deployment:
            spec:
              template:
                spec:
                  containers:
                  - name: webhooks-connector
                    env:
                    - name: CERTIFICATES_IMPORT_LIST
                      value: /etc/secrets/kafka/ca.pem:kafka_ca,/etc/secrets/custom/ca.pem:custom_ca
                    volumeMounts:
                    - mountPath: /etc/secrets/custom
                      name: custom-cert
                      readOnly: true
                  volumes:
                  - name: custom-cert
                    secret:
                      defaultMode: 420
                      items:
                      - key: ca_cert
                        path: ca.pem
                      secretName: ${INSTANCE}-custom-webhooks-cert
    EOF
    ```

1. Wait approximately 10 minutes for the `wa-webhooks-connector` pod to restart. This pod restarts automatically.

1. After the pod restarts, check the logs by running the following command. In the command, replace `XXXX` with the suffix of the `wa-webhooks-connector` pod:
    ```
    oc logs wa-webhooks-connector-XXXX     // Replace XXXX with the suffix of the wa-webhooks-connector pod
    ```

    After you run this command, you should see two lines similar to the following example at the beginning of the log:
    ```
    Certificate was added to keystore
    Certificate was added to keystore
    ```

    When you see these two lines, then the custom certificate was properly imported into the keystore.

## 4.0.5
{: #troubleshoot-405}

### Install Redis with {{site.data.keyword.assistant_classic_short}} if foundational services version is higher than 3.14.1
{: #troubleshoot-405-install-redis-foundational-services}

If you are installing the Redis operator with {{site.data.keyword.assistant_classic_short}} for {{site.data.keyword.icp4dfull}} 4.0.5 with an IBM Cloud Pak foundational services version higher than 3.14.1, the Redis operator might get stuck in `Pending` status. If you have an air-gapped cluster, complete the steps in the [Air-gapped cluster](#troubleshoot-405-install-redis-foundational-services-air-gapped) section to resolve this issue. If you are using the IBM Entitled Registry, complete the steps in the [IBM Entitled Registry](#troubleshoot-405-install-redis-foundational-services-entitled-registry) section to resolve this issue.

#### Air-gapped cluster
{: #troubleshoot-405-install-redis-foundational-services-air-gapped}

1. Check the status of the Redis operator:
    ```
    oc get opreq common-service-redis -n ibm-common-services -o jsonpath='{.status.phase}  {"\n"}'
    ```

1. If the Redis operand request is stuck in `Pending` status, delete the operand request:
    ```
    oc delete opreq watson-assistant-redis -n ibm-common-services
    ```

1. Set up your environment to download the CASE packages.

    a) Create the directories where you want to store the CASE packages:
    ```
    mkdir -p $HOME/offline/cpd  
    mkdir -p $HOME/offline/cpfs  
    ```

    b) Set the following environment variables:
    ```
    export CASE_REPO_PATH=https://github.com/IBM/cloud-pak/raw/master/repo/case  
    export OFFLINEDIR=$HOME/offline/cpd  
    export OFFLINEDIR_CPFS=$HOME/offline/cpfs  
    ```

1. Download the Redis operator and {{site.data.keyword.icp4dfull}} platform operator CASE packages:
    ```
    cloudctl case save \
    --repo ${CASE_REPO_PATH} \
    --case ibm-cloud-databases-redis \
    --version  1.4.5  \
    --outputdir $OFFLINEDIR

    cloudctl case save \
    --repo ${CASE_REPO_PATH} \
    --case ibm-cp-datacore \
    --version 2.0.10 \
    --outputdir ${OFFLINEDIR} \
    --no-dependency
    ```

1. Create the Redis catalog source:
    ```
    cloudctl case launch \
    --case ${OFFLINEDIR}/ibm-cloud-databases-redis-1.4.5.tgz \
    --inventory redisOperator \
    --action install-catalog \
    --namespace openshift-marketplace \
    --args "--registry icr.io --inputDir ${OFFLINEDIR} --recursive"
    ```

1. Set the environment variables for your registry credentials:
    ```
    export PRIVATE_REGISTRY_USER=username  
    export PRIVATE_REGISTRY_PASSWORD=password  
    export PRIVATE_REGISTRY={registry-info}  
    ```

1. Run the following command to store the credentials:
    ```
    cloudctl case launch \
    --case ${OFFLINEDIR}/ibm-cp-datacore-2.0.10.tgz \
    --inventory cpdPlatformOperator \
    --action configure-creds-airgap \
    --args "--registry ${PRIVATE_REGISTRY} --user ${PRIVATE_REGISTRY_USER} --pass ${PRIVATE_REGISTRY_PASSWORD}"
    ```

1. Mirror the images:
    ```
    export USE_SKOPEO=true  
    cloudctl case launch \
    --case ${OFFLINEDIR}/ibm-cp-datacore-2.0.10.tgz \
    --inventory cpdPlatformOperator \
    --action mirror-images \
    --args "--registry ${PRIVATE_REGISTRY} --user ${PRIVATE_REGISTRY_USER} --pass ${PRIVATE_REGISTRY_PASSWORD} --inputDir ${OFFLINEDIR}"
    ```

1. Create the Redis subscription.

    a) Export the project that contains the {{site.data.keyword.icp4dfull}} operator:
    ```
    export OPERATOR_NS=ibm-common-services|cpd-operators     # Select the project that contains the Cloud Pak for Data operator
    ```

    b) Create the subscription:
    ```
    cat <<EOF | oc apply --namespace $OPERATOR_NS -f -
    apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
      name: ibm-cloud-databases-redis-operator
    spec:
      name: ibm-cloud-databases-redis-operator
      source: ibm-cloud-databases-redis-operator-catalog
      sourceNamespace: openshift-marketplace
    EOF
    ```

#### IBM Entitled Registry
{: #troubleshoot-405-install-redis-foundational-services-entitled-registry}

1. Check the status of the Redis operator:
    ```
    oc get opreq common-service-redis -n ibm-common-services -o jsonpath='{.status.phase}  {"\n"}'

    ```

1. If the Redis operand request is stuck in `Pending` status, delete the operand request:
    ```
    oc delete opreq watson-assistant-redis -n ibm-common-services
    ```

1. Create the Redis subscription. Create one of the following two subscriptions, depending on whether you are using.

    a) Export the project that contains the {{site.data.keyword.icp4dfull}} operator:
    ```
    export OPERATOR_NS=ibm-common-services|cpd-operators     # Select the project that contains the Cloud Pak for Data operator
    ```

    b) Create the subscription. Choose one of the following two subscriptions, depending on how you are using the IBM Entitled Registry:

    - IBM Entitled Registry from the `ibm-operator-catalog`:
    ```
    cat <<EOF | oc apply --namespace  $OPERATOR_NS -f -
    apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
      name: ibm-cloud-databases-redis-operator
    spec:
      name: ibm-cloud-databases-redis-operator
      source: ibm-operator-catalog
      sourceNamespace: openshift-marketplace
    EOF
    ```

    - IBM Entitled Registry with catalog sources that pull specific versions of the images:
    ```
    cat <<EOF | oc apply --namespace $OPERATOR_NS -f -
    apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
      name: ibm-cloud-databases-redis-operator
    spec:
      name: ibm-cloud-databases-redis-operator
      source: ibm-cloud-databases-redis-operator-catalog
      sourceNamespace: openshift-marketplace
    EOF
    ```

1. Validate that the operator was successfully created.

    a) Run the following command to confirm that the subscription was applied:
    ```
    oc get sub -n $OPERATOR_NS  ibm-cloud-databases-redis-operator -o jsonpath='{.status.installedCSV} {"\n"}'
    ```

    b) Run the following command to confirm that the operator is installed:
    ```
    oc get pod -n $OPERATOR_NS -l app.kubernetes.io/name=ibm-cloud-databases-redis-operator \
    -o jsonpath='{.items[0].status.phase} {"\n"}'
    ```

## 4.0.4
{: #troubleshoot-404}

### Integrations image problem on air-gapped installations
{: #troubleshoot-404-integrations-image-air-gapped}

If your {{site.data.keyword.assistant_classic_short}} 4.0.4 installation is air-gapped, your integrations image fails to properly start.

If the installation uses the IBM Entitled Registry to pull images, complete the steps in the **IBM Entitled Registry** section. If the installation uses a private Docker registry to pull images, complete the steps in the **Private Docker registry** section.

#### IBM Entitled Registry

If your installation uses the IBM Entitled Registry to pull images, complete the following steps to add an override entry to the {{site.data.keyword.assistant_classic_short}} CR:

1. Get the name of your {{site.data.keyword.assistant_classic_short}} by running the following command:
    ```
    oc get wa
    ```

1. Edit and save the CR.

    a) Run the following command to edit the CR. In the command, replace `INSTANCE_NAME` with the name of the {{site.data.keyword.assistant_classic_short}} instance:
    ```
    oc edit wa INSTANCE_NAME
    ```

    b) Edit the CR by adding the following lines:
    ```
    appConfigOverrides:
      container_images:
      integrations:
        image: cp.icr.io/cp/watson-assistant/servicedesk-integration
        tag: 20220106-143142-0ea3fbf7-wa_icp_4.0.5-signed@sha256:7078fdba4ab0b69dbb93f47836fd9fcb7cfb12f103662fef0d9d1058d2553910
    ```

1. Wait for the {{site.data.keyword.assistant_classic_short}} operator to pick up the change and start a new integrations pod. This might take up to 10 minutes.

1. After the new integrations pod starts, the old pod terminates. When the new pod starts, the server starts locally and the log looks similar to the following example:
    ```
    oc logs -f ${INTEGRATIONS_POD}
    [2022-01-07T01:33:13.609] [OPTIMIZED] db.redis.RedisManager - Redis trying to connect. counter# 1
    [2022-01-07T01:33:13.628] [OPTIMIZED] db.redis.RedisManager - Redis connected
    [2022-01-07T01:33:13.629] [OPTIMIZED] db.redis.RedisManager - Redis is ready to serve!
    [2022-01-07T01:33:14.614] [OPTIMIZED] Server - Server started at: https://localhost:9449
    ```

#### Private Docker registry

If your installation uses a private Docker registry to pull images, complete the following steps to download and push the new integrations image to your private Docker registry and add an override entry to the {{site.data.keyword.assistant_classic_short}} CR:

1.  Edit the {{site.data.keyword.assistant_classic_short}} CSV file to add the new integrations image.

    a) Run the following command to open the {{site.data.keyword.assistant_classic_short}} CSV file:
    ```
    vi $OFFLINEDIR/ibm-watson-assistant-4.0.4-images.csv
    ```

    b) Add the following line to {{site.data.keyword.assistant_classic_short}} CSV file immediately after the existing integrations image:
    ```
    cp.icr.io,cp/watson-assistant/servicedesk-integration,20220106-143142-0ea3fbf7-wa_icp_4.0.5-signed,sha256:7078fdba4ab0b69dbb93f47836fd9fcb7cfb12f103662fef0d9d1058d2553910,IMAGE,linux,x86_64,"",0,CASE,"",ibm_wa_4_0_0;ibm_wa_4_0_2;ibm_wa_4_0_4;vLatest
    ```

1. Mirror the image again using the commands that you used to download and push all the images, for example:
    ```
    cloudctl case launch \
      --case ${OFFLINEDIR}/ibm-cp-datacore-2.0.9.tgz \
      --inventory cpdPlatformOperator \
      --action configure-creds-airgap \
      --args "--registry cp.icr.io --user cp --pass $PRD_ENTITLED_REGISTRY_APIKEY --inputDir ${OFFLINEDIR}"
    ```

1. Get the name of your {{site.data.keyword.assistant_classic_short}} by running the following command:
    ```
    oc get wa
    ```

1. Edit and save the CR.

    a) Run the following command to edit the CR. In the command, replace `INSTANCE_NAME` with the name of the {{site.data.keyword.assistant_classic_short}} instance:
    ```
    oc edit wa INSTANCE_NAME
    ```

    b) Edit the CR by adding the following lines:
    ```
    appConfigOverrides:
      container_images:
      integrations:
        image: cp.icr.io/cp/watson-assistant/servicedesk-integration
        tag: 20220106-143142-0ea3fbf7-wa_icp_4.0.5-signed@sha256:7078fdba4ab0b69dbb93f47836fd9fcb7cfb12f103662fef0d9d1058d2553910
    ```

1. Wait for the {{site.data.keyword.assistant_classic_short}} operator to pick up the change and start a new integrations pod. This might take up to 10 minutes.

1. After the new integrations pod starts, the old pod terminates. When the new pod starts, the server starts locally and the log looks similar to the following example:
    ```
    oc logs -f ${INTEGRATIONS_POD}
    [2022-01-07T01:33:13.609] [OPTIMIZED] db.redis.RedisManager - Redis trying to connect. counter# 1
    [2022-01-07T01:33:13.628] [OPTIMIZED] db.redis.RedisManager - Redis connected
    [2022-01-07T01:33:13.629] [OPTIMIZED] db.redis.RedisManager - Redis is ready to serve!
    [2022-01-07T01:33:14.614] [OPTIMIZED] Server - Server started at: https://localhost:9449
    ```

## 4.0.0
{: #troubleshoot-400}

### Install {{site.data.keyword.assistant_classic_short}} 4.0.0 with EDB version 1.8
{: #troubleshoot-400-install-edb18}

Complete this task only if you need a fresh installation of {{site.data.keyword.assistant_classic_short}} 4.0.0. Do not complete this task on existing clusters with data. Completing this task on an existing cluster with data results in data loss.
{: important}

If you upgraded to EDB version 1.8 and need a new installation of {{site.data.keyword.assistant_classic_short}} 4.0.0, complete the following steps. In the following steps, `wa` is used as the name of the custom resource. Replace this value with the name of your custom resource:

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

    If the jobs don't complete successfully, then they timed out and need to be recreated. Delete the jobs by running the `oc delete jobs wa-create-slot-store-db-job wa-4.0.0-update-schema-store-db-job` command. The jobs are recreated after 10 minutes by the {{site.data.keyword.assistant_classic_short}} operator.

For information about upgrading from {{site.data.keyword.assistant_classic_short}} 4.0.0 to {{site.data.keyword.assistant_classic_short}} 4.0.2, see [Upgrading {{site.data.keyword.assistant_classic_short}} to a newer 4.0 refresh](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.0?topic=assistant-upgrading-watson-version-40){: external}.

## 1.5.0
{: #troubleshoot-150}

### Search skill not working because of custom certificate
{: #troubleshoot-150-search-skill}

The search skill, which is the integration with the Watson Discovery service, might not work if you [configured a custom TLS certificate](https://www.ibm.com/docs/en/cloud-paks/cp-data/3.5.0?topic=client-using-custom-tls-certificate) in {{site.data.keyword.icp4dfull}}. If the custom certificate is not signed by a well-known certificate authority (CA), the search skill does not work as expected. You might also see errors in the **Try out** section of the search skill.

#### Validate the issue
First, check the logs of the search skill pods to confirm whether this issue applies to you.

1.  Run the following command to list the search skill pods:
    ```
    oc get pods -l component=skill-search
    ```

1.  Run the following command to check the logs for the following exception:
    ```
    oc logs -l component=skill-search | grep "IBMCertPathBuilderException"
    ```

    The error looks similar to the following example:
    ```
    {"level":"ERROR","logger":"wa-skills","message":"Search skill exception","exception":[{"message":"com.ibm.jsse2.util.h: PKIX path building failed: com.ibm.security.cert.IBMCertPathBuilderException: unable to find valid certification path to requested target","name":"SSLHandshakeException"
    ```

    If you see this error, continue to follow the steps to apply the fix.

#### Apply the fix

To fix the search skill, you inject the CA that signed your TLS certificate into the Java truststore that is used by the search skill. The search skill pods are then able to validate your certificate and communicate with the Watson Discovery service.

1.  First, get your certificate. You might have this certificate, but in these steps you can retrieve the certificate directly from the cluster.

    a) Run the following command to check that the secret exists:
    ```
    oc get secret external-tls-secret
    ```

    b) Run the following command to retrieve the certificate chain from the secret:
    ```
    oc get secret external-tls-secret --output jsonpath='{.data.cert\.crt}' | base64 -d | tee ingress_cert_chain.crt
    ```

    c) Extract the CA certificate. The `ingress_cert_chain.crt` file typically contains multiple certificates. The last certificate in the file is usually your CA certificate. Copy the last certificate block that begins with `-----BEGIN CERTIFICATE-----` and ends with `-----END CERTIFICATE-----`. Save this certificate in the `ingress_ca.crt` file. When you save the `ingress_ca.crt` file, the `-----BEGIN CERTIFICATE-----` line must be the first line of the file, and the `-----END CERTIFICATE-----` line must be the last line of the file.

1.  Retrieve the truststore that is used by the search skill pods.

    a) Run the following command to list the search skill pods:
    ```
    oc get pods -l component=skill-search
    ```

    b) Run the following command to set the `SEARCH_SKILL_POD` environment variable with the search skill pod name:
    ```
    SEARCH_SKILL_POD="$(oc get pods -l component=skill-search --output custom-columns=NAME:.metadata.name --no-headers | head -n 1)"
    ```

    c) Run the following command to see the selected pod:
    ```
    echo "Selected search skill pod: ${SEARCH_SKILL_POD}"
    ```

    d) Retrieve the truststore file. The `cacerts` file is the default truststore that is used by Java. It contains the list of the certificate authorities that Java trusts by default. Run the following command to copy the binary `cacerts` file from the pod into your current directory:
    ```
    oc cp ${SEARCH_SKILL_POD}:/opt/ibm/java/jre/lib/security/cacerts cacerts
    ```

1.  Run the following command to inject the `ingress_ca.crt` file into the `cacerts` file:
    ```
    keytool -import -trustcacerts -keystore cacerts -storepass changeit -alias customer_ca -file ingress_ca.crt
    ```

    You can run the `keytool -list -keystore cacerts -storepass changeit | grep customer_ca -A 1` command to check that your CA certificate is included in the `cacerts` file.
    {: tip}

1.  Run the following command to create the configmap that contains the updated `cacerts` file:
    ```
    oc create configmap watson-assistant-skill-cacerts --from-file=cacerts
    ```

    Because the `cacerts` file is binary, the output of the `oc describe configmap watson-assistant-skill-cacerts` command shows an empty data section. To check whether the updated `cacerts` file is present in the configmap, run the `oc get configmap watson-assistant-skill-cacerts --output yaml` command.

1.  Override the `cacerts` file in the search skill pods. In this step, you configure the {{site.data.keyword.assistant_classic_short}} operator to override the `cacerts` file in the search skill pods with the updated `cacerts` file. In the following example file, the {{site.data.keyword.assistant_classic_short}} instance is called `watson-assistant---wa`. Replace this value with the name of your instance:
    ```
    cat <<EOF | oc apply -f -
    kind: TemporaryPatch
    apiVersion: com.ibm.oppy/v1
    metadata:
      name: watson-assistant---wa-skill-cert
    spec:
      apiVersion: com.ibm.watson.watson-assistant/v1
      kind: WatsonAssistantSkillSearch
      name: "watson-assistant---wa"    # Replace this with the name of your Watson Assistance instance
      patchType: patchStrategicMerge
      patch:
        "skill-search":
          deployment:
            spec:
              template:
                spec:
                  volumes:
                   - name: updated-cacerts
                     configMap:
                       name: watson-assistant-skill-cacerts
                       defaultMode: 420
                  containers:
                  - name: skill-search
                    volumeMounts:
                    - name: updated-cacerts
                      mountPath: /opt/ibm/java/jre/lib/security/cacerts
                      subPath: cacerts
    EOF
    ```  

1.  Wait until new search skill pods are created. It might take up to 10 minutes before the updates take affect.

1.  Check that the search skill feature is working as expected.

### Disable Horizontal Pod Autoscaling and set a maximum number of master pods
{: #troubleshoot-150-disable-hpa}

Horizontal Pod Autoscaling (HPA) is enabled automatically for {{site.data.keyword.assistant_classic_short}}. As a result, the number of replicas changes dynamically in the range of 1 to 10 replicas. You can disable HPA if you want to limit the maximum number of master pods or if you're concerned about master pods being created and deleted too frequently.

1.  First, disable HPA for the `master` microservice by running the following command. In these steps, substitute your instance name for the `INSTANCE_NAME` variable:
    ```
    oc patch wa ${INSTANCE_NAME} --type='json' --patch='[{"op": "add", "path": "/appConfigOverrides/clu", "value":{"master":{"autoscaling":{"enabled":false}}}}]'
    ```

1.  Wait until the information propagates into the {{site.data.keyword.assistant_classic_short}} operator:
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

### {{site.data.keyword.assistant_classic_short}} 1.5.0 patch 1
[{{site.data.keyword.assistant_classic_short}} 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164) is available for installations of version 1.5.0.

### Resizing the Redis statefulset memory and cpu values after applying patch 1 for {{site.data.keyword.assistant_classic_short}} 1.5.0

{{site.data.keyword.assistant_classic_short}} uses Redis to store web session-related data. Here are steps to resize Redis statefulset memory and cpu values after applying [{{site.data.keyword.assistant_classic_short}} 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164).

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

    Member CPU
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/cpu", "value":"50m"}]'
    ```
    {: codeblock}

    Member memory
    ```
    oc patch statefulset c-$INSTANCENAME-redis-m --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/limits/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/memory", "value":"256Mi"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/memory", "value":"256Mi"}]'
    ```
    {: codeblock}

    Sentinel CPU
    ```
    oc patch statefulset c-$INSTANCENAME-redis-s --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/1/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/2/resources/requests/cpu", "value":"50m"},{"op": "replace", "path": "/spec/template/spec/containers/3/resources/requests/cpu", "value":"50m"}]'
    ```
    {: codeblock}

    Sentinel memory
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

Whenever the size of {{site.data.keyword.assistant_classic_short}} is changed from medium to small, a manual step is required to delete the `poddisruptionbudgets` that are created for medium instances.

Run the following command, replacing `<instance-name>` with the name of your {{site.data.keyword.assistant_classic_short}} CR instance and replacing `<namespace-name>` with the name of the namespace where the instance resides.

```
oc delete pdb  -l icpdsupport/addOnId=assistant,component!=etcd,ibmevents.ibm.com/kind!=Kafka,app.kubernetes.io/instance=<instance-name> -n <namespace-name>

```
