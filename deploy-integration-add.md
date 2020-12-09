---

copyright:
  years: 2015, 2020
lastupdated: "2020-10-20"

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
{:video: .video}

# Adding integrations
{: #deploy-integration-add}

To deploy your skill, add it to an assistant, and then add integrations to the assistant that publish your bot to the channels where your customers go for help.
{: shortdesc}

The built-in integrations were introduced with version 1.5.0.
{: note}

## Add an integration
{: #deploy-integration-add-task}

There is currently no way to pass an ongoing conversation from one integration channel to another.
{: important}

Follow these steps to add integrations to your assistant:

1.  Click the **Assistants** icon ![Assistants menu icon](images/nav-ass-icon.png).

1.  Click to open the tile for the assistant that you want to deploy.

1.  Go to the Integrations section.

    **Why do I have *Preview link* and *Web chat* integrations?** These integrations are provisioned for you automatically (unless you choose not to enable them). 
    
    The preview link embeds your assistant as a simple chat widget in an IBM-branded web page where you can test your assistant.

    The web chat is a chat window that you can embed in one or more pages on your website to share your assistant with your customers.

1.  Click **Add integration**.

1.  Click the tile for the channel with which you want to integrate the assistant. The options include:

    - [Preview link](/docs/assistant?topic=assistant-deploy-web-link)
    - [Web chat](/docs/assistant-data?topic=assistant-data-deploy-web-chat)
    - [Custom application](/docs/assistant-data?topic=assistant-data-deploy-custom-app)

1.  Follow the instructions that are provided on the screen to complete the integration process.

After you integrate the assistant, test it from the target channel to ensure that the assistant works as expected.

## Integration limits
{: #deploy-integration-add-limits}

You can add 100 integrations per assistant.