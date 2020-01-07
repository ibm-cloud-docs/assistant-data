---

copyright:
  years: 2015, 2020
lastupdated: "2019-11-06"

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

# Service information
{: #services-information}

The assistant that you build is hosted by {{site.data.keyword.icp4dfull_notm}}.
{: shortdesc}

## Limits by artifact
{: #services-information-limits}

For information about artifact limits, see these topics:

- [Assistants](/docs/services/assistant-data?topic=assistant-data-assistant-add#assistant-add-limits)
- [Dialog nodes](/docs/services/assistant-data?topic=assistant-data-dialog-build#dialog-build-node-limits)
- [Entities](/docs/services/assistant-data?topic=assistant-data-entities#entities-limits)
- [Inactivity timeout](/docs/services/assistant-data?topic=assistant-data-assistant-settings#assistant-settings-session-limits)
- [Intents](/docs/services/assistant-data?topic=assistant-data-intents#intents-limits)
- [Skills](/docs/services/assistant-data?topic=assistant-data-skill-add#skill-add-limits)
- [Versions](/docs/services/assistant-data?topic=assistant-data-versions#versions-limits)

## Service API Versioning
{: services-information-api-version}

API requests require a version parameter that takes a date in the format `version=YYYY-MM-DD`. Whenever we change the API in a backwards-incompatible way, we release a new minor version of the API.

Send the version parameter with every API request. The service uses the API version for the date you specify, or the most recent version before that date. Don't default to the current date. Instead, specify a date that matches a version that is compatible with your app, and don't change it until your app is ready for a later version.

- The current version for both V1 and V2 is `2019-02-28`.
- The dialog skill "Try it out" pane uses version `2018-07-10`.
- The search skill "Try it out" pane uses {{site.data.keyword.discoveryshort}} API version `2018-12-03`.

For API usage information, see the API reference documentation. Links for v1 and v2 are available from the table of contents. For an example of using the API, see [Test the search skill](/docs/services/assistant-data?topic=assistant-data-skill-search-add#skill-search-add-test-via-api).
