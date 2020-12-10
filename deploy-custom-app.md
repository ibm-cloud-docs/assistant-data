---

copyright:
  years: 2015, 2020
lastupdated: "2020-10-06"

subcollection: assistant-data


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
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

# Integrating with a custom application
{: #deploy-custom-app}

Build your own client application as the interface between the assistant and your customers.
{: shortdesc}

To use the API, you need to construct the URL to use in your requests.

1.  From the {{site.data.keyword.icp4dfull_notm}} web client, go to the details page for the provisioned instance.
1.  Copy the URL from the "Access information" section of the page. You will specify this value as the `{url}`.

    You might want to copy the bearer token also. You will need to pass the token when you make an API call.
1.  From the launched application instance, go to the Assistants page. Click the More options menu for the assistant you want to use, and click **Settings**. 
1.  Click **API details**, and then copy the assistant ID. You will specify this value as the `{assistant_id}`.
1.  Build a URL by using the IDs you copied. For example, the following request creates a session:

    ```
    curl -H "Authorization: Bearer eyJhb<snip>yA9g" -X POST "{url}/v2/assistants/{assistant_id}/sessions?version=2020-04-01 -k"
    ```
    {: codeblock}

- For more information, see [Building a client application](/docs/assistant-data?topic=assistant-data-api-client).
- For more information about the API, see [API overview](/docs/assistant-data?topic=assistant-data-api-overview).
- For API reference documentation, see [API reference](https://cloud.ibm.com/apidocs/assistant/assistant-data-v2){: external}.
