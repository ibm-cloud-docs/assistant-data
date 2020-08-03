---

copyright:
  years: 2015, 2020
lastupdated: "2020-08-03"

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

# Adding a skill to your assistant
{: #skill-add}

Customize your assistant by adding to it the skills it needs to satisfy your customers' goals.
{: shortdesc}

You can create the following types of skills:

- **Dialog skill**: Uses Watson natural language processing and machine learning technologies to understand user questions and requests, and respond to them with answers that are authored by you.

- **Search skill**: For a given user query, uses the {{site.data.keyword.discoveryfull}} service to search a data source of your self-service content and return an answer.

Typically, you create a skill of each type first. Then, as you build a dialog for the dialog skill, you decide when to initiate the search skill. For some questions or requests, a hardcoded or programmatically-derived response (that is defined in the dialog skill) is sufficient. For others, you might want to provide a more robust response by returning a full passage of related information (that is extracted from an external data source by using the search skill).

## Dialog skill
{: #skills-dialog-skill}

A dialog skill contains the training data and logic that enables an assistant to help your customers. It contains the following types of artifacts:

- [**Intents**](/docs/assistant-data?topic=assistant-data-intents): An *intent* represents the purpose of a user's input, such as a question about business locations or a bill payment. You define an intent for each type of user request you want your application to support. In the tool, the name of an intent is always prefixed with the `#` character. To train the dialog skill to recognize your intents, you supply lots of examples of user input and indicate which intents they map to.

  A *content catalog* is provided that contains prebuilt common intents you can add to your application rather than building your own. For example, most applications require a greeting intent that starts a dialog with the user. You can add the **General** content catalog to add an intent that greets the user and does other useful things, like end the conversation.

- [**Dialog**](/docs/assistant-data?topic=assistant-data-dialog-build): A *dialog* is a branching conversation flow that defines how your application responds when it recognizes the defined intents and entities. You use the dialog editor in the tool to create conversations with users, providing responses based on the intents and entities that you recognize in their input.

  ![Diagram of a basic implementation that uses intent and dialog only.](images/basic-impl.png)

To enable your dialog skill to handle more nuanced questions, define entities and reference them from your dialog.

- [**Entities**](/docs/assistant-data?topic=assistant-data-entities); An *entity* represents a term or object that is relevant to your intents and that provides a specific context for an intent. For example, an entity might represent a city where the user wants to find a business location, or the amount of a bill payment. In the tool, the name of an entity is always prefixed with the `@` character.

  You can train the skill to recognize your entities by providing entity term values and synonyms, entity patterns, or by identifying the context in which an entity is typically used in a sentence. To fine tune your dialog, go back and add nodes that check for entity mentions in user input in addition to intents.

![Diagram of a more complex implementation that uses intent, entity, and dialog.](images/complex-impl.png)

As you add information, the skill uses this unique data to build a machine learning model that can recognize these and similar user inputs. Each time you add or change the training data, the training process is triggered to ensure that the underlying model stays up-to-date as your customer needs and the topics they want to discuss change.

For help creating a dialog skill, see [Creating a dialog skill](/docs/assistant-data?topic=assistant-data-skill-dialog-add).

## Search skill
{: #skills-search-skill}

When Watson Assistant doesn't have an explicit solution to a problem, it routes the user question to a search skill to find an answer from across your disparate sources of self-service content. The search skill interacts with the {{site.data.keyword.discoveryfull}} service to extract this information from a configured data collection.

If you already have {{site.data.keyword.discoveryshort}} for {{site.data.keyword.icp4dfull_notm}} installed and an instance provisioned, you can mine your existing data collections for source material that you can share with customers to address their questions.

The following diagram illustrates how user input is processed when both a dialog skill and a search skill are added to an assistant. Any questions that the dialog is not designed to answer are sent to the search skill, which finds a relevant response from a Discovery data collection.

![Diagram of how a question is sent to the Discovery service from a search skill.](images/search-skill-diagram.png)

For help creating a search skill, see [Creating a search skill](/docs/assistant-data?topic=assistant-data-skill-search-add).

## Create the skill
{: #skill-add-task}

You can add one skill of each skill type to an assistant.

- [Dialog Skill](/docs/assistant-data?topic=assistant-data-skill-dialog-add)
- [Search Skill](/docs/assistant-data?topic=assistant-data-skill-search-add)

## Skill limits
{: #skill-add-limits}

| Skills per service instance |
|-----------------------------|
|                          50 |
{: caption="Skill limit details" caption-side="top"}

Skill versions do not count toward the skill limit.

## Deleting a skill
{: #skill-add-delete}

You can delete any skill that you can access, unless it is being used by an assistant. If it is in use, you must remove it from the assistant that is using it before you can delete it.

Be sure to check with anyone else who might be using the skill before you delete it.
{: tip}

To delete a skill, complete the following steps:

1.  Find out whether the skill is being used by any assistants. From the Skills page, find the tile for the skill that you want to delete. The **Assistants** field lists the assistants that currently use the skill.

1.  If the skill you want to delete is associated with an assistant, then remove it from the assistant by completing the following steps:

    - Check with the owner of the assistant that is using the skill before you remove the skill from it.
    - Open the Assistants page, and then click to open the assistant tile.
    - Find the tile for the skill that you want to delete. Click the ![open and close list of options](images/kabob-beta.png) icon, and then choose **Remove**.
    - Repeat the previous steps for any other assistants that use the skill.
    - Return to the Skills page, and then find the tile for the skill that you want to delete.

1.  Click the ![open and close list of options](images/kabob-beta.png) icon, and then choose **Delete**. Confirm the deletion.
