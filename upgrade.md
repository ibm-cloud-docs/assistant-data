---

copyright:
  years: 2015, 2019
lastupdated: "2019-06-18"

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

To upgrade your instance, complete these steps:

1.  From the earlier version of the service, [export any workspaces](/docs/services/assistant-icp?topic=assistant-private-configure-workspace#exporting-and-copying-workspaces){: external} you want to keep. Store them in a repository that will not be impacted when you uninstall the product.
1.  Uninstall the previous version.
1.  Install the new version of the service.

    With the new version, the notion of a workspace was replaced by a dialog skill.
    {: note} 
    
1.  Import the data you downloaded from your workspace by creating a dialog skill. At creation time, choose to import a skill, and then upload the JSON file from the workspace that exported earlier. For more details, see [Creating a dialog skill](/docs/services/assistant-data?topic=assistant-data-skill-dialog-add).
  
## Update your client applications
{: #upgrade-api}

When you import a workspace as a dialog skill, the new skill has a new workspace ID. If you have existing client applications that use the v1 API to access this workspace, then you must update any workspace ID references to use the new worskpace ID instead.
