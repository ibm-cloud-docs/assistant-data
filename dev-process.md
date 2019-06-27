---

copyright:
  years: 2015, 2019
lastupdated: "2019-06-10"

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

# Development process
{: #dev-process}

Use {{site.data.keyword.conversationshort}} to leverage AI as you build, deploy, and incrementally improve a conversational assistant.
{: shortdesc}

![Shows the flow of development steps starting with developing training data and ending with deploying to production](images/dev-process.png)

## Workflow
{: #dev-process-workflow}

The typical workflow for an assistant project includes the following steps:

1.  Define a narrow set of key customer needs that you want the assistant to address on your behalf, including any business processes that it can initiate or complete for your customers. Start small.
1.  Create intents that represent the customer needs you identified in the previous step. For example, intents such as `#about_company` or `#place_order`.

1.  Build a dialog that detects the defined intents and addresses them, either with simple responses or with a dialog flow that collects more information first.
1.  Define any entities that are needed to more clearly understand the user's meaning.

    Mine existing intent user examples for common entity value mentions. Using annotations to define entities captures not only the text of the entity value, but the context in which the entity value is typically used in a sentence.

    For dictionary-based entities, use synonym recommendations to expand your entity definitions.
    {: tip}

1.  Test each function that you add to the assistant in the "Try it" pane, incrementally, as you go.
1.  When you have a working assistant that can successfully handle key tasks, add an integration that deploys the assistant to a development environment. Test the deployed assistant and make refinements.

1.  After you build an effective assistant, take a snapshot of the dialog skill and save it as a version.

    Saving a version when you reach a development milestone gives you something you can go back to if subsequent changes you make to the skill decrease its effectiveness. See [Creating skill versions](/docs/services/assistant-data?topic=assistant-data-versions).
1.  Deploy the version of the assistant into a test environment, and test it.

1.  Monitor the chat transcript logs for your test assistant to determine if you need to make improvements to your training data or dialog.`*`
1.  When you are happy with the performance of your assistant, deploy the best version of the assistant into a production environment.
1.  Monitor the logs from conversations that users have with the deployed assistant.`*`

`*`The process of monitoring the conversations that your assistant has with your customers is your responsibility. Devise ways to analyze these chat transcripts so you can find out where the assistant is misunderstanding users or where its training data needs to be improved. For example, you might need to augment the training data if many customers ask for help with something that your assistant does not know anything about, or you might have two intents that are not distinct enough from one another if your assistant regularly picks the wrong dialog branches for certain topics.
