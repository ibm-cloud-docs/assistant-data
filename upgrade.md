---

copyright:
  years: 2015, 2020
lastupdated: "2020-08-20"

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

# Upgrading
{: #upgrade}

Learn how to upgrade from one version of {{site.data.keyword.conversationfull}} for {{site.data.keyword.icp4dfull_notm}} to another.
{: shortdesc}

Bulk feature updates are announced as they become available. You can choose whether to upgrade your instance to the latest version or not. If you want continuous updates to be applied to your service instance automatically, consider creating a service instance in the public IBM Cloud.

When you upgrade from one version of the product to another, you can export the following data from a {{site.data.keyword.conversationshort}} service instance:

- Dialog skill training data (intents and entities)
- Dialog skill dialog

You cannot export the following data:

- Search skill
- Assistant

To upgrade your instance, complete these steps:

1.  From the earlier version of the service, [download any dialog skills](/docs/assistant-data?topic=assistant-data-skill-dialog-add#skill-dialog-add-download) that you want to keep. Store them in a repository that will not be impacted when you uninstall the product.

    Alternatively, you can use the `/workspaces` API to export a dialog skill. Include the `export=true` parameter with the GET workspace request. See the [API reference ](https://cloud.ibm.com/apidocs/assistant/assistant-data-v1#getworkspace){: external} for more details.
    
1.  Uninstall the previous version.
1.  Install the new version of the service.
1.  Import the data you downloaded from your previous skill by creating a new dialog skill. 

    At creation time, choose to import a skill, and then upload the JSON file from the skill that was exported earlier. For more details, see [Creating a dialog skill](/docs/assistant-data?topic=assistant-data-skill-dialog-add).

    If you are upgrading to V1.3, in this version of the product, a workspace is now represented by a dialog skill.
    {: note} 

## Re-creating your assistant
{: #upgrade-recreate-assistant}

You can now re-create your assistant. You can then link your imported dialog skill to the assistant.

See [Creating an assistant](/docs/assistant-data?topic=assistant-data-assistant-add) for more details.

## Update your client applications (1.3 and earlier only)
{: #upgrade-api}

When you import a dialog skill that you exported, a new skill is created. The new skill has a new workspace ID. If you have existing client applications that use the v1 API to access this skill, then you must update any workspace ID references to use the new workspace ID instead.

When you re-create your assistant, it is given a new assistant ID. If you have existing client applications that use the v2 API to access the assistant, then you must update any assistant ID references to use the new assistant ID instead.