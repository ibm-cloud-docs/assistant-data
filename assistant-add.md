---

copyright:
  years: 2015, 2019
lastupdated: "2019-11-22"

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

# Creating an assistant
{: #assistant-add}

Create an assistant with the skills it needs to address the business goals of your customers.
{: shortdesc}

Follow these steps to create an assistant:

1.  Click the **Assistants** icon ![Assistants menu icon](images/nav-ass-icon.png).

    **v1.3**: Click the Assistants tab. If you don't see the Assistants tab, click the breadcrumb link in the page header.

1.  Click **Create assistant**.

1.  Add details about the new assistant:

    - **Name**: A name no more than 100 characters in length. A name is required.
    - **Description**: An optional description no more than 200 characters in length.

1.  Click **Create assistant**.

1.  Add a skill to the assistant by choosing one of the following skill types to add.

    **Note**: You can choose to add an existing skill or create a new one.

    - **Add Dialog Skill**: Uses Watson natural language processing and machine learning technologies to understand user questions and requests, and respond to them with answers that are authored by you.

      When you add a dialog skill from here, you get the development version. If you want to add a specific dialog skill version, add it from the skill's *Versions* page instead.

    - **Add Search Skill**: For a given user query, uses the {{site.data.keyword.discoveryfull}} service to retrieve information from a data source that you identify and shares any relevant information that it finds as the response to the user.

      You must have {{site.data.keyword.discoveryshort}} for {{site.data.keyword.icp4dfull_notm}} installed, and an instance provisioned before you can complete the steps to create a search skill.
      {: important}

    See [Creating a skill](/docs/services/assistant-data?topic=assistant-data-skill-add).

## Assistant limits
{: #assistant-add-limits}

| Assistants per service instance |
|---------------------------------|
| 100 |
{: caption="Limit details" caption-side="top"}

For information about the inactivity timeout limit, see [Changing the inactivity timeout setting](/docs/services/assistant-data?topic=assistant-data-assistant-settings).

You can connect one skill of each type to your assistant. 

## Deleting an assistant
{: #assistant-add-delete}

When you delete an assistant, any skills that you added to the assistant are not deleted.

To delete an assistant, follow these steps:

1.  From the Assistants tab, find the assistant that you want to delete.

1.  Click the ![open and close list of options](images/kabob-beta.png) icon, and then choose **Delete**. Confirm the deletion.

## Renaming an assistant
{: #assistant-add-rename}

You can change the name of an assistant and its associated description after you create the assistant.

To rename an assistant, follow these steps:

1.  From the Assistants tab, find the assistant that you want to rename.

1.  Click the ![open and close list of options](images/kabob-beta.png) icon, and then choose **Rename**.

1.  Edit the name, and then click **Rename** to save your changes.

### Changing the skill that is associated with the assistant
{: #assistant-add-swap-skill}

You can add one skill of each skill type to an assistant. If you want to change a skill that your assistant is using, you can swap one skill for another skill.

1.  From the Assistants tab, open the assistant.

1.  Click the ![open and close list of options](images/kabob-beta.png) icon for the skill you want to swap, and then choose **Swap skill**.

    To swap the current dialog skill for a different version of the skill, choose **Change skill version**.

1.  Choose an existing skill to use instead or [create a skill](/docs/services/assistant-data?topic=assistant-data-skill-add).

    You cannot swap search skills currently.
    {: important}

### Switching between service instances
{: #assistant-add-switch-instance}

If you have more than one deployed instance of {{site.data.keyword.conversationshort}} in your cluster, you can switch to a different instance.

To change to a different instance, you must open the other instance from the *My instances* page of the {{site.data.keyword.icp4dfull_notm}} web client.

**V1.3 only**: If you are working within a skill, click the **Skills** breadcrumb link first. The banner displays the current instance name. To switch to a different service instance, click **Change**, and then choose the appropriate instance.
