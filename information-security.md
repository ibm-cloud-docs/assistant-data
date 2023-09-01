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

# Information security
{: #information-security}

IBM is committed to providing our clients and partners with innovative data privacy, security and governance solutions.
{: shortdesc}

**Notice:**
Clients are responsible for ensuring their own compliance with various laws and regulations, including the European Union General Data Protection Regulation (GDPR). Clients are solely responsible for obtaining advice of competent legal counsel as to the identification and interpretation of any relevant laws and regulations that may affect the clients’ business and any actions the clients may need to take to comply with such laws and regulations.

The products, services, and other capabilities described herein are not suitable for all client situations and may have restricted availability. IBM does not provide legal, accounting or auditing advice or represent or warrant that its services or products will ensure that clients are in compliance with any law or regulation.

If you need to request GDPR support for {{site.data.keyword.cloud}} {{site.data.keyword.watson}} resources that are created

- In the European Union, see [Requesting support for IBM Cloud Watson resources created in the European Union](https://cloud.ibm.com/docs/watson/getting-started-gdpr-sar#request-EU){: external}.
- Outside the European Union, see [Requesting support for resources outside the European Union](https://cloud.ibm.com/docs/watson/getting-started-gdpr-sar#request-non-EU){: external}.

## European Union General Data Protection Regulation (GDPR)
{: #information-security-gdpr}

IBM is committed to providing our clients and partners with innovative data privacy, security and governance solutions to assist them on their journey to GDPR compliance.

Learn more about IBM's own GDPR readiness journey and our GDPR capabilities and offerings to support your compliance journey [here](http://www.ibm.com/gdpr){: external}.

## More information
{: #information-security-gdpr-icp4d}

For more information about data privacy in {{site.data.keyword.icp4dfull_notm}}, see [IBM Cloud Pak for Data considerations for GDPR readiness](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.0?topic=compliance-considerations-gdpr-readiness){: external}.

## Labeling and deleting data in Watson Assistant
{: #information-security-gdpr-wa}

**1.5.0 and later** Support for deleting data was added when log retention and analytics support were added to the product.

Do not add personal data to the training data (entities and intents, including user examples) that you create.

If you need to remove a customer's message data from a {{site.data.keyword.assistant_classic_short}} instance, you can do so based on the customer ID of the client, as long as you associate the message with a customer ID when the message is sent to {{site.data.keassistant_classic_shortnshort}}.

- The preview link integration does not support the labeling and therefore deletion of data based on customer ID. Do not use the preview link integration in a solution that must support the ability to delete data based on a customer ID.
- For the web chat integration, the service takes the `user_id` that is passed in and adds it as the `customer_id` parameter value to the `X-Watson-Metadata` header with each request.

### Before you begin
{: #information-security-delete-user-data-prereqs}

To be able to delete message data associated with a specific user, you must first associate all messages with a unique **customer ID** for each user. To specify the **customer ID** for any messages sent from a custom client application by using the `/message` API, include the `X-Watson-Metadata: customer_id` property in your header. For example:

```sh
curl -X POST -u "apikey:3Df... ...Y7Pc9"
 --header
   'Content-Type: application/json'
   'X-Watson-Metadata: customer_id=abc'
 --data
   '{"input":{"text":"hello"}}'
  '{url}/v2/assistants/{assistant_id}/sessions/{session_id}/message?version=2019-02-28'
```
{: pre}

where {url} is the appropriate URL for your instance. For more details, see [Service endpoint](/apidocs/assistant-data-v2#service-endpoint){: external}.

The `customer_id` string cannot include the semicolon (`;`) or equal sign (`=`) characters. You are responsible for ensuring that each `customer ID` property is unique across your customers.
{: note}

You can pass multiple **customer ID** values with semicolon-separated `customer_id={value}` pairs. For example: `'X-Watson-Metadata: customer_id=abc;customer_id=xyz'`

If you add a search skill to an assistant, user input that is submitted to the assistant is passed to the {{site.data.keyword.discoveryshort}} service as a search query. If the {{site.data.keyword.assistant_classic_short}} integration provides a customer ID, then the resulting `/message` API request includes the customer ID in the header, and the ID is passed through to the {{site.data.keyword.discoveryshort}} `/query` API request. To delete any query data that is associated with a specific customer, you must send a separate delete request directly to the {{site.data.keyword.discoveryshort}} service instance that is linked your the assistant. See the appropriate documentation for the version of the API that is used by your {{site.data.keyword.discoveryshort}} service instance:

- v1: [Information security](/docs/discovery?topic=discovery-information-security)
- v2: [Information security](/docs/discovery-data?topic=discovery-data-information-security)

### Querying user data
{: #information-security-query-customer-id}

Use the v1 `/logs` method `filter` parameter to search an application log for specific user data. For example, to search for data specific to a `customer_id` that matches `my_best_customer`, the query might be:

``` sh
curl -X GET -u "apikey:3Df... ...Y7Pc9" \
"{url}/v2/assistants/{assistant_id}/logs?version=2020-04-01&filter=customer_id::my_best_customer"
```
{: pre}

where {url} is the appropriate URL for your instance. For more details, see [Service endpoint](/apidocs/assistant-data-v2#service-endpoint){: external}.

See the [Filter query reference](/docs/assistant-data?topic=assistant-data-filter-reference) for additional details.

### Deleting data
{: #information-security-delete-data}

To delete any message log data associated with a specific user that your assistant might have stored, use the `DELETE /user_data` v1 API method. Specify the customer ID of the user by passing a `customer_id` parameter with the request.

Only data that was added by using the `POST /message` API endpoint with an associated customer ID can be deleted using this delete method. Data that was added by other methods cannot be deleted based on customer ID. For example, entities and intents that were added from customer conversations, cannot be deleted in this way. Personal Data is not supported for those methods.

**IMPORTANT**: Specifying a `customer_id` will delete *all* messages with that `customer_id` that were received before the delete request, across your entire {{site.data.keyword.assistant_classic_short}} instance, not just within one skill.

As an example, to delete any message data associated with a user that has the customer ID `abc` from your {{site.data.keyword.assistant_classic_short}} instance, send the following cURL command:

```sh
curl -X DELETE -u "apikey:3Df... ...Y7Pc9" \
"{url}/v2/user_data?customer_id=abc&version=2020-04-01"
```
{: pre}

where {url} is the appropriate URL for your instance. For more details, see [Service endpoint](/apidocs/assistant-data-v2#service-endpoint){: external}.

An empty JSON object `{}` is returned.

For more information, see the [API reference](/apidocs/assistant-data-v2#deleteuserdata).

**Note:** Delete requests are processed in batches and may take up to 24 hours to complete.
