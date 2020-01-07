---

copyright:
  years: 2015, 2020
lastupdated: "2019-06-28"

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

# Installing the 1.2 add-on
{: #install-120-addon}

Use {{site.data.keyword.conversationshort}} for {{site.data.keyword.icp4dfull}} V1.2.0 to build conversational interfaces into any app, device, or channel. 
{: shortdesc}

For the full installation procedure, see [the {{site.data.keyword.icp4dfull_notm}} knowledge center instructions](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/watson/assistant-install.html){: external}.

The following supplemental information applies to the add-on. 

## System requirements
{: #install-120-addon-reqs-over}

See [System requirements](/docs/services/assistant-data?topic=assistant-data-install-120#install-120-reqs-over).

## Creating persistent volumes for a production environment
{: #install-120-addon-create-pvs-prod}

Consider using an {{site.data.keyword.icp4dfull_notm}} storage [add-on](https://www.ibm.com/support/knowledgecenter/SSQNUZ_2.1.0/com.ibm.icpdata.doc/zen/admin/add-ons.html#add-ons__storage){: external} or a storage option that is hosted outside the cluster, such as [vSphere Cloud Provider](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.2/manage_cluster/vsphere_land.html){: external}.
