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

# Changing the inactivity timeout setting
{: #assistant-settings}

A single assistant-managed chat session is designed to end after a specific period of time passes in which the user does not interact with the assistant. You can specify the amount of time to wait before a chat session is reset by changing the inactivity timeout setting.
{: shortdesc}

When a chat session is reset, the dialog loses any contextual information that it saved during the previous exchange with the user. For example, if the dialog asks for the user's name and then calls the user by that name throughout the rest of the conversation, then after the chat session ends and a new one begins, the dialog will start by asking for the user's name again.

## Session limit
{: #assistant-settings-session-limits}

| Chat session default inactivity period | Chat session maximum inactivity period |
|---------------------------------|----------------------------|
|                      60 minutes |                   24 hours |
{: caption="Limit details" caption-side="top"}

## Changing the setting
{: #assistant-settings-timeout-task}

1.  Click the menu for your assistant, and then choose **Settings**.

1.  Change the time specified in the **Timeout limit** field.

1.  Close the settings page. Your change is saved automatically.
