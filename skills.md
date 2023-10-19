---

copyright:
  years: 2015, 2019
lastupdated: "2019-09-03"

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

# Skills
{: #skills}

A skill is a container for the artificial intelligence that enables an assistant to help your customers.
{: shortdesc}

An assistant directs requests down the optimal path for solving a customer problem. Add skills so your assistant can provide a direct answer to a common question or reference more generalized search results for something more complex.

## Skill types
{: #skills-types}

You can add the following type of skill to your assistant:

- **[Dialog skill](#skills-dialog-skill)**: Understands typical questions or requests from users and answers or fulfills them by following a dialog that is scripted by you.

- **[Search skill](#skills-search-skill)**: Answers a user's question by searching for relevant information from an external data source, extracting the passage, and returning it as the assistant's response.

### Dialog skill
{: #skills-dialog-skill}

A dialog skill contains the training data and logic that enables an assistant to help your customers. It contains the following types of artifacts:

- [**Intents**](/docs/services/assistant-data?topic=assistant-data-intents): An *intent* represents the purpose of a user's input, such as a question about business locations or a bill payment. You define an intent for each type of user request you want your application to support. In the tool, the name of an intent is always prefixed with the `#` character. To train the dialog skill to recognize your intents, you supply lots of examples of user input and indicate which intents they map to.

  A *content catalog* is provided that contains prebuilt common intents you can add to your application rather than building your own. For example, most applications require a greeting intent that starts a dialog with the user. You can add the **General** content catalog to add an intent that greets the user and does other useful things, like end the conversation.

- [**Dialog**](/docs/services/assistant-data?topic=assistant-data-dialog-build): A *dialog* is a branching conversation flow that defines how your application responds when it recognizes the defined intents and entities. You use the dialog editor in the tool to create conversations with users, providing responses based on the intents and entities that you recognize in their input.

  ![Diagram of a basic implementation that uses intent and dialog only.](images/basic-impl.png)

To enable your dialog skill to handle more nuanced questions, define entities and reference them from your dialog.

- [**Entities**](/docs/services/assistant-data?topic=assistant-data-entities); An *entity* represents a term or object that is relevant to your intents and that provides a specific context for an intent. For example, an entity might represent a city where the user wants to find a business location, or the amount of a bill payment. In the tool, the name of an entity is always prefixed with the `@` character.

  You can train the skill to recognize your entities by providing entity term values and synonyms, entity patterns, or by identifying the context in which an entity is typically used in a sentence. To fine tune your dialog, go back and add nodes that check for entity mentions in user input in addition to intents.

![Diagram of a more complex implementation that uses intent, entity, and dialog.](images/complex-impl.png)

As you add information, the skill uses this unique data to build a machine learning model that can recognize these and similar user inputs. Each time you add or change the training data, the training process is triggered to ensure that the underlying model stays up-to-date as your customer needs and the topics they want to discuss change.

For help creating a dialog skill, see [Creating a dialog skill](/docs/services/assistant-data?topic=assistant-data-skill-dialog-add).

### Search skill
{: #skills-search-skill}

When Watson Assistant doesn't have an explicit solution to a problem, it routes the user question to a search skill to find an answer from across your disparate sources of self-service content. The search skill interacts with the {{site.data.keyword.discoveryfull}} service to extract this information from a configured data collection.

If you already have {{site.data.keyword.discoveryshort}} for {{site.data.keyword.icp4dfull_notm}} installed and an instance provisioned, you can mine your existing data collections for source material that you can share with customers to address their questions.

The following diagram illustrates how user input is processed when both a dialog skill and a search skill are added to an assistant. Any questions that the dialog is not designed to answer are sent to the search skill, which finds a relevant response from a Discovery data collection.

![Diagram of how a question is sent to the Discovery service from a search skill.](images/search-skill-diagram.png)

For help creating a search skill, see [Creating a search skill](/docs/services/assistant-data?topic=assistant-data-skill-search-add).