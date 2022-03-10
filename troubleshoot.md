---

copyright:
  years: 2015, 2022
lastupdated: "2022-03-10"

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

## 4.0.x
{: #troubleshoot-40x}

### Security context constraint permission errors
{: #troubleshoot-40x-scc-permission-error}

The following fix applies to {{site.data.keyword.conversationshort}} 4.0.0 through 4.0.5. If a cluster has a security context constraint (SCC) that takes precedence over **restricted** SCCs and has different permissions than **restricted** SCCs, then {{site.data.keyword.conversationshort}} 4.0.0 through 4.0.5 installations might fail with permission errors. For example, the `update-schema-store-db-job` job reports errors similar to the following example:
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

Now, new {{site.data.keyword.conversationshort}} pods default back to the expected **restricted** SCC. When you run the `oc describe pod wa-etcd-0 |grep scc` command, you get an output similar to the following example:
```
openshift.io/scc: restricted
```

### Unable to collect logs with a webhook
{: #troubleshoot-40x-collect-logs-webhook}

The following fix applies to all versions of {{site.data.keyword.conversationshort}} 4.0.x. If you're unable to collect logs with a webhook, it might be because you are using a webhook that connects to a server that is using a self-signed certificate. If so, complete the following steps to import the certificate into the keystore so that you can collect {{site.data.keyword.conversationshort}} logs with a webhook:

1. Log in to cluster and `oc project cpd-instance`, which is the namespace where the {{site.data.keyword.conversationshort}} instance is located.

1. Run the following command. In the following command, replace `INSTANCE_NAME` with the name of your {{site.data.keyword.conversationshort}} instance and replace `CUSTOM_CERTIFICATE` with your Base64 encoded custom certificate key:
    ```
    INSTANCE="INSTANCE_NAME"     # Replace instance-name with the name of the Watson Assistant instance
    CERT="CUSTOM_CERTIFICATE"     # Replace custom-certificate with the custom certificate key

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

### Install Redis with {{site.data.keyword.conversationshort}} if foundational services version is higher than 3.14.1
{: #troubleshoot-405-install-redis-foundational-services}

If you are installing the Redis operator with {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull}} 4.0.5 with an IBM Cloud Pak foundational services version higher than 3.14.1, the Redis operator might get stuck in `Pending` status. If you have an air-gapped cluster, complete the steps in the [Air-gapped cluster](#troubleshoot-405-install-redis-foundational-services-air-gapped) section to resolve this issue. If you are using the IBM Entitled Registry, complete the steps in the [IBM Entitled Registry](#troubleshoot-405-install-redis-foundational-services-entitled-registry) section to resolve this issue.

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

If your {{site.data.keyword.conversationshort}} 4.0.4 installation is air-gapped, your integrations image fails to properly start.

If the installation uses the IBM Entitled Registry to pull images, complete the steps in the **IBM Entitled Registry** section. If the installation uses a private Docker registry to pull images, complete the steps in the **Private Docker registry** section.

#### IBM Entitled Registry

If your installation uses the IBM Entitled Registry to pull images, complete the following steps to add an override entry to the {{site.data.keyword.conversationshort}} CR:

1. Get the name of your {{site.data.keyword.conversationshort}} by running the following command:
    ```
    oc get wa
    ```

1. Edit and save the CR.

    a) Run the following command to edit the CR. In the command, replace `INSTANCE_NAME` with the name of the {{site.data.keyword.conversationshort}} instance:
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

1. Wait for the {{site.data.keyword.conversationshort}} operator to pick up the change and start a new integrations pod. This might take up to 10 minutes.

1. After the new integrations pod starts, the old pod terminates. When the new pod starts, the server starts locally and the log looks similar to the following example:
    ```
    oc logs -f ${INTEGRATIONS_POD}
    [2022-01-07T01:33:13.609] [OPTIMIZED] db.redis.RedisManager - Redis trying to connect. counter# 1
    [2022-01-07T01:33:13.628] [OPTIMIZED] db.redis.RedisManager - Redis connected
    [2022-01-07T01:33:13.629] [OPTIMIZED] db.redis.RedisManager - Redis is ready to serve!
    [2022-01-07T01:33:14.614] [OPTIMIZED] Server - Server started at: https://localhost:9449
    ```

#### Private Docker registry

If your installation uses a private Docker registry to pull images, complete the following steps to download and push the new integrations image to your private Docker registry and add an override entry to the {{site.data.keyword.conversationshort}} CR:

1.  Edit the {{site.data.keyword.conversationshort}} CSV file to add the new integrations image.

    a) Run the following command to open the {{site.data.keyword.conversationshort}} CSV file:
    ```
    vi $OFFLINEDIR/ibm-watson-assistant-4.0.4-images.csv
    ```

    b) Add the following line to {{site.data.keyword.conversationshort}} CSV file immediately after the existing integrations image:
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

1. Get the name of your {{site.data.keyword.conversationshort}} by running the following command:
    ```
    oc get wa
    ```

1. Edit and save the CR.

    a) Run the following command to edit the CR. In the command, replace `INSTANCE_NAME` with the name of the {{site.data.keyword.conversationshort}} instance:
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

1. Wait for the {{site.data.keyword.conversationshort}} operator to pick up the change and start a new integrations pod. This might take up to 10 minutes.

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

### Install {{site.data.keyword.conversationshort}} 4.0.0 with EDB version 1.8
{: #troubleshoot-400-install-edb18}

Complete this task only if you need a fresh installation of {{site.data.keyword.conversationshort}} 4.0.0. Do not complete this task on existing clusters with data. Completing this task on an existing cluster with data results in data loss.
{: important}

If you upgraded to EDB version 1.8 and need a new installation of {{site.data.keyword.conversationshort}} 4.0.0, complete the following steps. In the following steps, `wa` is used as the name of the custom resource. Replace this value with the name of your custom resource:

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

    If the jobs don't complete successfully, then they timed out and need to be recreated. Delete the jobs by running the `oc delete jobs wa-create-slot-store-db-job wa-4.0.0-update-schema-store-db-job` command. The jobs are recreated after 10 minutes by the {{site.data.keyword.conversationshort}} operator.

For information about upgrading from {{site.data.keyword.conversationshort}} 4.0.0 to {{site.data.keyword.conversationshort}} 4.0.2, see [Upgrading {{site.data.keyword.conversationshort}} to a newer 4.0 refresh](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.0?topic=assistant-upgrading-watson-version-40){: external}.

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

1.  Override the `cacerts` file in the search skill pods. In this step, you configure the {{site.data.keyword.conversationshort}} operator to override the `cacerts` file in the search skill pods with the updated `cacerts` file. In the following example file, the {{site.data.keyword.conversationshort}} instance is called `watson-assistant---wa`. Replace this value with the name of your instance:
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

Horizontal Pod Autoscaling (HPA) is enabled automatically for {{site.data.keyword.conversationshort}}. As a result, the number of replicas changes dynamically in the range of 1 to 10 replicas. You can disable HPA if you want to limit the maximum number of master pods or if you're concerned about master pods being created and deleted too frequently.

1.  First, disable HPA for the `master` microservice by running the following command. In these steps, substitute your instance name for the `INSTANCE_NAME` variable:
    ```
    oc patch wa ${INSTANCE_NAME} --type='json' --patch='[{"op": "add", "path": "/appConfigOverrides/clu", "value":{"master":{"autoscaling":{"enabled":false}}}}]'
    ```

1.  Wait until the information propagates into the {{site.data.keyword.conversationshort}} operator:
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

### {{site.data.keyword.conversationshort}} 1.5.0 patch 1
[{{site.data.keyword.conversationshort}} 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164) is available for installations of version 1.5.0.

### Resizing the Redis statefulset memory and cpu values after applying patch 1 for {{site.data.keyword.conversationshort}} 1.5.0

{{site.data.keyword.conversationshort}} uses Redis to store web session-related data. Here are steps to resize Redis statefulset memory and cpu values after applying [{{site.data.keyword.conversationshort}} 1.5.0 patch 1](https://www.ibm.com/support/pages/node/6240164).

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

Whenever the size of {{site.data.keyword.conversationshort}} is changed from medium to small, a manual step is required to delete the `poddisruptionbudgets` that are created for medium instances.

Run the following command, replacing `<instance-name>` with the name of your {{site.data.keyword.conversationshort}} CR instance and replacing `<namespace-name>` with the name of the namespace where the instance resides.

```
oc delete pdb  -l icpdsupport/addOnId=assistant,component!=etcd,ibmevents.ibm.com/kind!=Kafka,app.kubernetes.io/instance=<instance-name> -n <namespace-name>

```

## 1.4.2
{: #troubleshoot-142}

### Cannot provision an instance, and service images are missing from the catalog
{: #troubleshoot-142-missing-label}

If you run the installation with no errors, but cannot provision an instance, check whether the product icon is visible in the service tile. From the {{site.data.keyword.icp4dfull_notm}} web client, go to the *Services* page (![Services icon](images/cp4d-services-icon.png)).

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
