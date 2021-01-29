---

copyright:
  years: 2015, 2021
lastupdated: "2021-01-22"

subcollection: assistant-data

content-type: faq

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
{:faq: data-hd-content-type='faq'}

# FAQ
{: #faqs}

Find answers to frequently-asked questions and quick fixes for common problems.
{: shortdesc}

## What's a...
{: #faqs-answers}
{: faq}

| Term | Definition |
|------|------------|
| Assistant | Container for your skills. You add skills to an assistant, and then deploy the assistant when you are ready to start helping your customers. [Learn more](/docs/assistant-data?topic=assistant-data-assistants). |
| Condition | Logic that is defined in the *If assistant recognizes* section of a dialog node that determines whether the node is processed. The dialog node conditions is equivalent to an If statement in If-Then-Else programming logic.
| Content catalog | A set of prebuilt intents that are categorized by subject, such as customer care. You can add these intents to your skill and start using them immediately. Or you can edit them to complement other intents that you create. [Learn more](/docs/assistant-data?topic=assistant-data-catalog). |
| Context variable | A variable that you can use to collect information during a conversation, and reference it later in the same conversation. For example, you might want to ask for the customer's name and then address the person by name later on. A context variable is used by the dialog skill. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-runtime-context#dialog-runtime-context-variables).|
| Dialog | The component where you build the conversation that your assistant has with your customers. For each defined intent, you can author the response your assistant should return. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-overview). |
| Digression | A feature that gives the user the power to direct the conversation. It prevents customers from getting stuck in a dialog thread; they can switch topics whenever they choose. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-runtime#dialog-runtime-digressions). |
| Disambiguation | A feature that enables the assistant to ask customers to clarify their meaning when the assistant isn't sure what a user wants to do next. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-runtime#dialog-runtime-disambiguation). |
| Entity | Information in the user input that is related to the user's purpose. An intent represents the action a user wants to do. An entity represents the object of that action. [Learn more](/docs/assistant-data?topic=assistant-data-entities). |
| Integrations | Ways you can deploy your assistant to existing platforms or social media channels. [Learn more](/docs/assistant-data?topic=assistant-data-deploy-integration-add). |
| Intent | The goal that is expressed in the user input, such as answering a question or processing a bill payment. [Learn more](/docs/assistant-data?topic=assistant-data-intents). |
| Message | A single turn within a conversation that includes a single call to the `/message` API endpoint and its corresponding response. |
| Monthly active user (MAU) | A single unique user who interacts with an assistant one or many times in a given month. |
| Preview link | An integration that builds embeds your assistant in a chat window that is displayed on an IBM-branded web page. From the preview link integration, you can test how a conversation flows through any and all skills that are attached to your assistant, from end to end. [Learn more](/docs/assistant-data?topic=assistant-data-deploy-web-link). |
| Response | Logic that is defined in the *Assistant responds* section of a dialog node that determines how the assistant responds to the user. When the node's condition evaluates to true, the response is processed. The response can consist of an answer, a follow-up question, a webhook that sends a programmatic request to an external service, or slots which represent pieces of information that you need the user to provide before the assistant can help. The dialog node response is equivalent to a Then statement in If-Then-Else programming logic. |
| Skill | Does the work of the assistant. A dialog skill has the training data and dialog that your assistant uses to chat with customers. An actions skill is a new way to build a conversation. Actions offers step-by-step flows for a range of simple or complex conversations and is made so that anybody can build. A search skill is configured to search the appropriate external data sources for answers to customer inquiries. [Learn more](/docs/assistant-data?topic=assistant-data-skill-add). |
| Skill version | Versions are snapshots of a skill that you can create at key points during the development lifecycle. You can deploy one version to production, while you continue to make and test improvements that you make to another version of the skill. [Learn more](/docs/assistant-data?topic=assistant-data-versions). |
| Slots | A special set of fields that you can add to a dialog node that enable the assistant to collect necessary pieces of information from the customer. For example, the assistant can require a customer to provide valid date and location details before it gets weather forecast information on the customer's behalf. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-slots). |
| System entity | Prebuilt entities that recognize references to common things like dates and numbers. You can add these to your skill and start using them immediately. [Learn more](/docs/assistant-data?topic=assistant-data-system-entities). |
| Try it out | A chat window that you use to test as you build. For example, from the dialog skill's "Try it out" pane, you can mimic the behavior of a customer and enter a query to see how the assistant responds. You can test only the current skill; you cannot test your assistant and all attached skills from end to end. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-tasks). |  
| Web chat | An integration that you can use to embed your assistant in your company website. [Learn more](/docs/assistant-data?topic=assistant-data-deploy-web-chat). |
| Webhook | A mechanism for calling out to an external program as part of the dialog. For example, if a customer asks the assistant to translate a string from English to French, the dialog can call an external language translation service to translate the phrase and return the translation to the customer in the course of the conversation. [Learn more](/docs/assistant-data?topic=assistant-data-dialog-webhooks). |

## I don’t see the Analytics page
{: #faqs-view-analytics}
{: faq}

You can only view the Analytics page if the system administrator enabled the feature in your deployment. A prerequisite service that is required by the feature is only available on OpenShift Red Hat 4.5; it is not available on 3.11.

## Where can I find an example for creating my first assistant?
{: #faqs-get-started}
{: faq}

Follow the steps in the [Getting started with {{site.data.keyword.conversationshort}}](/docs/assistant-data?topic=assistant-data-getting-started) tutorial for a product introduction and to get help creating your first assistant.

## Can I export the user conversations from the Analytics page?
{: #faqs-export-conversation}
{: faq}

You cannot directly export conversations from the User conversation page.  You can, however, use the `/logs` API to list events from the transcripts of conversations that occurred between your users and your assistant. For more information, see the [API reference](https://cloud.ibm.com/apidocs/assistant-data-v1#listlogs){: external} and the [Filter query reference](/docs/assistant-data?topic=assistant-data-filter-reference).

## Can I export and import dialog nodes?
{: #faqs-nodes}
{: faq}

No, you cannot export and import dialog nodes from the product user interface.

If you want to copy dialog nodes from one skill into another skill, follow these steps:

1.  Download as JSON files both the dialog skill that you want to copy the dialog nodes from and the dialog skill that you want to copy the nodes to.
1.  In a text editor, open the JSON file for the dialog skill that you want to copy the dialog nodes from.
1.  Find the `dialog_nodes` array, and copy it.
1.  In a text editor, open the JSON file for the dialog skill that you want to copy the dialog nodes to, and then paste the `dialog_nodes` array into it.
1.  Import the JSON file that you edited in the previous step to create a new dialog skill with the dialog nodes you wanted.

## How long are log files kept for a workspace?
{: #faqs-assistant-logs}
{: faq}

Messages are retained for 90 days. For more information, see [Log limits](/docs/assistant-data?topic=assistant-data-logs#logs-limits).

## How do I create a webhook?
{: #faqs-webhook-how}
{: faq}

To define a webhook and add its details, open the skill where you want to add the webhook. Open the **Options** page, and then click **Webhooks** to add details about your webhook. To invoke the webhook, call it from one or more of your dialog nodes. For more information, see [Making a programmatic call from dialog](/docs/assistant-data?topic=assistant-data-dialog-webhooks).

## Can I have more than one entry in the URL field for a webhook?
{: #faqs-webhook-url}
{: faq}

No, you can define only one webhook URL for a dialog skill. For more information, see [Defining the webhook](/docs/assistant-data?topic=assistant-data-dialog-webhooks#dialog-webhooks-create).

## Can I extend the webhook time limit?
{: #faqs-webhook-timeout}
{: faq}

No. The service that you call from the webhook must return a response in 8 seconds or less, or the call is canceled. You cannot increase this time limit. For more information about strategies for handling complex actions with a webhook, watch the video in [Making a programmatic call from dialog](/docs/assistant-data?topic=assistant-data-dialog-webhooks).
