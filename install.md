---

copyright:
  years: 2015, 2021
lastupdated: "2021-01-11"

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
{:download: .download}
{:gif: data-image-type='gif'}

# Installation overview
{: #install}

Learn how to install {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull}}.
{: shortdesc}

The {{site.data.keyword.icp4dfull_notm}} environment is a Kubernetes-based container platform that can help you quickly modernize and automate workloads that are associated with the applications and services you use. You can develop and deploy on your own infrastructure and in your data center which helps to mitigate risk and minimize vulnerabilities.

The installation process differs depending on the version you are installing. The following table shows the available versions.

| Watson Assistant chart |  Cluster | Installation checklist |
|------------------------|---------------------------|-------------------|
| 1.5.0 | {{site.data.keyword.icp4dfull_notm}} 3.0.1 or 3.5 | [Installing 1.5.0](/docs/assistant-data?topic=assistant-data-install-150) |
| 1.4.2 | {{site.data.keyword.icp4dfull_notm}} 2.5 or 3.0.1 (**I do not have a cluster**)  | [Installing {{site.data.keyword.conversationshort}} 1.4.2](/docs/assistant-data?topic=assistant-data-install-142) |
| 1.4.2 | {{site.data.keyword.icp4dfull_notm}} 3.0.1 (**I already have a cluster**) | [Installing the {{site.data.keyword.conversationshort}} service](https://www.ibm.com/support/knowledgecenter/SSQNUZ_3.0.1/cpd/svc/watson/assistant-install.html){: external} |
| 1.4.1 | {{site.data.keyword.icp4dfull_notm}} 2.1.0.2 or 2.5 (**I do not have a cluster**)  | [Installing {{site.data.keyword.conversationshort}} 1.4.1](/docs/assistant-data?topic=assistant-data-install-141) |
| 1.4 - 1.4.2 | {{site.data.keyword.icp4dfull_notm}} 2.5 (**I already have a cluster**) | [Installing the {{site.data.keyword.conversationshort}} service](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.5.0/cpd/svc/watson/assistant-install.html){: external} |
| 1.4 | {{site.data.keyword.icp4dfull_notm}} 2.1.0.2 or 2.5 (**I do not have a cluster**)  | [Installing {{site.data.keyword.conversationshort}} 1.4](/docs/assistant-data?topic=assistant-data-install-140) |
| 1.3 or 1.2 | {{site.data.keyword.icp4dfull_notm}} 2.1.0.0 - 2.1.0.2 (**I already have a cluster**) | [Installing the {{site.data.keyword.conversationshort}} add-on](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/watson/assistant-install.html){: external} |
| 1.3 | {{site.data.keyword.icp4dfull_notm}} 2.1.0.1 -2.1.0.2 (**I do not have a cluster**)  | [Installing 1.3](/docs/assistant-data?topic=assistant-data-install-130) |
| 1.2 | {{site.data.keyword.icp4dfull_notm}} 2.1.0.0 (**I do not have a cluster**)  | [Installing 1.2](/docs/assistant-data?topic=assistant-data-install-120) |
{: caption="Available versions" caption-side="top"}

## Support matrix
{: #install-support-matrix}

The following tables describes which versions of are supported on which versions of and .

| {{site.data.keyword.conversationshort}} version | {{site.data.keyword.icp4dfull}} version | Red Hat OpenShift (RHOS) version | Notes |
| ----------------------------------|----------------|----------------|----------|
| 1.5.0 | 3.5 | 4.5 | Analytics feature works with this configuration only. |
| 1.5.0 | 3.5 | 3.11 | N/A |
| 1.5.0 | 3.0.1 | 4.5 | N/A |
| 1.5.0 | 3.0.1 | 3.11 | N/A |
| 1.4.2 | 3.0.1 | 4.5 | N/A |
| 1.4.2 | 3.0.1 | 3.11 | N/A |
| 1.4.2 | 2.5 | 3.11 | N/A |
| 1.4.1 | 2.5 | 3.11 | N/A |
{: caption="Support matrix" caption-side="top"}