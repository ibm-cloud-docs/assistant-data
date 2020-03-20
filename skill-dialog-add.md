---

copyright:
  years: 2015, 2020
lastupdated: "2020-03-20"

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

# Creating a dialog skill
{: #skill-dialog-add}

The natural-language processing for the {{site.data.keyword.conversationshort}} service is defined in a *dialog skill*, which is a container for all of the artifacts that define a conversation flow.
{: shortdesc}

You can add one dialog skill to an assistant.

## Create the dialog skill
{: #skill-dialog-add-task}

You can create a skill from scratch, use a sample skill that is provided by IBM, or import a skill from a JSON file.

To add a skill, complete the following steps:

1.  Click the **Skills** icon ![Skills menu icon](images/nav-skills-icon.png).

    **v1.3**: Click the **Skills** tab. If you don't see the Skills tab, click the breadcrumb link in the page header.    

1.  Click **Create skill**.

1.  Select the *dialog skill* option, and then click **Next**.

1.  Take one of the following actions:

    - To create a skill from scratch, click **Create skill**.
    - To add a sample skill that is provided with the product as a starting point for your own skill or as an example to explore before you create one yourself, click **Use sample skill**, and then click the sample you want to use.

      The sample skill is added to your list of skills. It is not associated with any assistants. Skip the remaining steps in this procedure.

    - To add an existing skill to this service instance, you can import it as a JSON file. Click **Import skill**, and then click **Choose JSON File**, and select the JSON file you want to import.

      Important:

      - The imported JSON file must use UTF-8 encoding, without byte order mark (BOM) encoding.
      - The maximum size for a skill JSON file is 10MB. If you need to import a larger skill, consider importing the intents and entities separately after you have imported the skill. (You can also import larger skills using the REST API. For more information, see the [API Reference](https://cloud.ibm.com/apidocs/assistant/assistant-data-v1?curl=#create-workspace){: external}.)
      - The JSON cannot contain tabs, newlines, or carriage returns.

      Specify the data you want to include:

        - Select **Everything (Intents, Entities, and Dialog)** if you want to import a complete copy of the exported skill, including the dialog.
        - Select **Intents and Entities** if you want to use the intents and entities from the exported skill, but you plan to build a new dialog.

      Click **Import**.

      If you have trouble importing a skill, see [Troubleshooting skill import issues](#skill-dialog-add-import-errors).

1.  Specify the details for the skill:

    - **Name**: A name no more than 100 characters in length. A name is required.
    - **Description**: An optional description no more than 200 characters in length.
    - **Language**: The language of the user input the skill will be trained to understand. The default value is English.

After you create the dialog skill, it appears as a tile on the Skills page. Now, you can start identifying the user goals that you want the dialog skill to address.

- To add prebuilt intents to your skill, see [Using content catalogs](/docs/assistant-data?topic=assistant-data-catalog).
- To define your own intents, see [Defining intents](/docs/assistant-data?topic=assistant-data-intents).

The dialog skill cannot interact with customers until it is added to an assistant and the assistant is deployed. See [Creating an assistant](/docs/assistant-data?topic=assistant-data-assistant-add).

### Troubleshooting skill import issues
{: #skill-dialog-add-import-errors}

If you receive warnings when you try to upload a skill, take a moment to try to fix the issues.

1.  Make a note of the warning message that are displayed when you try to upload the JSON file.
1.  Edit the skill to address the issues.

    - If you have access to a public service instance, open the skill from there and make edits. Export the edited skill by downloading it.
    - Otherwise, open the JSON file in a text editor and make edits. 

    You might need to remove the `@sys-person` and `@sys-location` entities from your training data, and any references to them from dialog nodes, for example.

1.  Try again to import the edited skill.

### Adding the skill to an assistant
{: #skill-dialog-add-to-assistant}

You can add one skill to an assistant. You must open the assistant tile and add the skill to the assistant from the assistant configuration page; you cannot choose the assistant that will use the skill from within the skill configuration page. One dialog skill can be used by more than one assistant.

1.  Click the **Assistants** icon ![Skills menu icon](images/nav-ass-icon.png) to open the Assistants page.

    **v1.3**: Click the Assistants tab.
    
1.  Click to open the tile for the assistant to which you want to add the skill.

1.  Click **Add Dialog Skill**.

1.  Click **Add existing skill**.

    Click the skill that you want to add from the available skills that are displayed.

When you add a dialog skill from here, you get the development version. If you want to add a specific skill version, add it from the skill's *Versions* tab instead.

## Downloading a dialog skill
{: #skill-dialog-add-download}

You can download a dialog skill in JSON format. You might want to download a skill if you want to use the same dialog skill in a different instance of the {{site.data.keyword.conversationshort}} service, for example. You can download it from one instance and import it to another instance as a new dialog skill.

To download a dialog skill, complete the following steps:

1.  Find the dialog skill tile on the Skills page or on the configuration page of an assistant that uses the skill.

1.  Click the ![open and close list of options](images/kabob-beta.png) icon, and then choose **Export**.

    **v1.3**: Choose **Download JSON**.

1.  Specify a name for the JSON file and where to save it, and then click **Save**.

You can export a skill by using the API also. Include the `export=true` parameter with the request. See the [API reference](https://cloud.ibm.com/apidocs/assistant/assistant-data-v1#get-information-about-a-workspace) for more details.
