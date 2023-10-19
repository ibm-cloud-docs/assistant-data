---

copyright:
  years: 2015, 2020
lastupdated: "2020-01-29"

subcollection: assistant-data

---

{{site.data.keyword.attribute-definition-list}}

Documentation about **{{site.data.keyword.assistant_classic_full}} for {{site.data.keyword.icp4dfull}}** has moved. For the most up-to-date version, see [Planning your assistant](/docs/watson-assistant?topic=watson-assistant-plan-assistant){: external}.
{: attention}

# Assistants
{: #assistants}

An assistant is a cognitive bot that you can customize for the unique needs of your business. You teach it about the types of things your customers want to know and do, and then design a script for the assistant to follow as it converses with your customers to help them accomplish their business goals.
{: shortdesc}

![Skills](images/skill-icon.png)  An assistant routes your customer queries to a skill, which then provides the appropriate response. Dialog skills return responses that are authored by you to answer common questions, while search skills search for and return passages from existing self-service content to answer more complex inquiries.

## Dialog skill
{: #assistants-dialog-skill}

A dialog skill can understand and address questions or requests that your customers typically need help with. You provide information about the subjects or tasks your users ask about, and how they ask about them, and the product dynamically builds a machine learning model that is tailored to understand the same and similar user requests.

| Dialog tree | Graphical user interface |
|-------------|-------------------------:|
| You can use graphical tools to create a dialog for your assistant to read from when interacting with your users, a dialog that simulates a real conversation. The dialog keys off the common customer goals that you teach it to recognize, and provides useful responses. | ![A sample dialog tree with example content](images/dialog-depiction.png) |

The dialog skill itself is defined in text, but you can integrate it with Watson Speech to Text and Watson Text to Speech services that enable users to interact with your assistant verbally.

![Out-of-the-box training data](images/oob.png)  If you want to get started quickly, add prebuilt training data to your dialog skill so your assistant can start helping your customers with the basics.

## Search skill
{: #assistants-search-skill}

A search skill leverages information from existing corporate knowledge bases or other collections of content authored by subject matter experts to address unanticipated or more nuanced customer inquiries.

See [Creating assistants](/docs/assistant-data?topic=assistant-data-assistant-add) to get started.
