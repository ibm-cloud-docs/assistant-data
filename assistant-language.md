---

copyright:
  years: 2015, 2021
lastupdated: "2021-07-01"

keywords: global support, universal language, universal model, another language

subcollection: assistant-data

---

{{site.data.keyword.attribute-definition-list}}

Documentation about **{{site.data.keyword.assistant_classic_full}} for {{site.data.keyword.icp4dfull}}** has moved. For the most up-to-date version, see [Adding support for global audiences](/docs/watson-assistant?topic=watson-assistant-admin-language-support){: external}.
{: attention}

# Adding support for global audiences
{: #assistant-language}

Your customers come from all around the globe. You need an assistant that can talk to them in their own language and in a familiar style. Choose the approach that best fits your business needs.

- **Quickest solution**: The simplest way to add language support is to author the conversational skill in a single language. You can translate each message that is sent to your assistant from the customer's local language to the skill language. Later you can translate each response from the skill language back to the customer's local language.

  This approach simplifies the process of authoring and maintaining the conversational skill. You can build one skill and use it for all languages. However, the intention and meaning of the customer message can be lost in the translation.

  For more information about webhooks you can use for translation, see [Webhook overview](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-webhook-overview).

- **Most precise solution**: If you have the time and resources, the best user experience can be achieved when you build multiple conversational skills, one for each language that you want to support. {{site.data.keyword.assistant_classic_short}} has built-in support for all languages. Use one of 13 language-specific models or the universal model, which adapts to any other language you want to support.

  When you build a skill that is dedicated to a language, a language-specific classifier model is used by the skill. The precision of the model means that your assistant can better understand and recognize the goals of even the most colloquial message from a customer.

  Use the new universal language model to create an assistant that is fluent even in languages that {{site.data.keyword.assistant_classic_short}} doesn't support with built-in models.

  To deploy, attach each language skill to its own assistant that you can deploy in a way that optimizes its use by your target audience.

## Understanding the universal language model
{: #assistant-language-universal}

A skill that uses the universal language model applies a set of shared linguistic characteristics and rules from multiple languages as a starting point. It then learns from the training data that you add to it.

The universal language classifier can adapt to a single language per skill. It cannot be used to support multiple languages within a single skill. However, you can use the universal language model in one skill to support one language, such as Russian, and in another skill to support another language, such as Hindi. The key is to add enough training examples or intent user examples in your target language to teach the model about the unique syntactic and grammatical rules of the language.

Use the universal language model when you want to create a conversation in a language for which no dedicated language model is available, and which is unique enough that an existing model is insufficient.

For more information about feature support in the universal language model, see [Supported languages](/docs/assistant-data?topic=assistant-data-language-support).

## Creating a skill that uses the universal language model
{: #assistant-language-universal-task}

To create a skill that uses the universal language model, complete the following steps:

1.  From the *Skills* page, click **Create skill**.

1.  Name your skill, and optionally add a description. From the *Language* field, choose **Another language**.

    Remember, if the language you want to support is listed individually, choose it instead of using the universal model. The built-in language models provide optimal language support.
    {: tip}

1.  Create the skill.

1.  If your skill will support a language with right-to-left directional text, such as Hebrew, configure the bidirectional capabilities.

    Click the options icon from the skill tile ![Skill options icon](images/kebab.png), and then select **Language preferences**. Click the *Enable bidirectional capabilities* switch. Specify any settings that you want to configure.

    For more information, see [Configuring bidirectional languages](/docs/assistant-data?topic=assistant-data-language-support#language-support-configure-bidirectional).

Next, start building your conversation.

As you follow the normal steps to design a conversational flow, you teach the universal language model about the language you want your skill to support. It is by adding training data that is written in the target language that the universal model is constructed. Add intents, intent user examples, and entities. The universal language model adapts to understand and support your language of choice.

## Integration considerations
{: #assistant-language-integrations}

When your skill is ready, you can add it to an assistant and deploy it. Keep these tips in mind:

- **Search skill**: If you build an assistant that specializes in a single language, be sure to connect it to data collections that are written in that language. For more information about the languages that are supported by {{site.data.keyword.discoveryshort}}, see [Language support](/docs/discovery-data?topic=discovery-data-language-support){: external}.
- **Web chat**: Web chat has some hardcoded strings that you can customize to reflect your target language. For more information, see [Global audience support](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-web-chat-basics#web-chat-basics-global).


