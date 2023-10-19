---

copyright:
  years: 2015, 2020
lastupdated: "2020-10-06"

keywords: integration settings

subcollection: assistant-data

---

{{site.data.keyword.attribute-definition-list}}

Documentation about **{{site.data.keyword.assistant_classic_full}} for {{site.data.keyword.icp4dfull}}** has moved. For the most up-to-date version, see [Adding custom dialog flows for integrations](/docs/watson-assistant?topic=watson-assistant-dialog-integrations){: external}.
{: attention}

# Adding custom dialog flows for integrations
{: #dialog-integrations}

Use the JSON editor in dialog to access information that is submitted from the web chat integration.
{: shortdesc}

The `context` object that is passed as part of the v2 `/message` API request contains an `integrations` object. This object makes it possible to pass information that is specific to a single integration type in the context. For more information about context variables, see [Context variables](/docs/assistant-data?topic=assistant-data-dialog-runtime-context#dialog-runtime-context-variables).

The `integrations` object is available from the v2 API in version `2020-04-01` or later only.
{: important} 

## Web chat: Accessing sensitive data
{: #dialog-integrations-chat-private}

If you enable security for the web chat, you can configure your web chat implementation to send encrypted data to the dialog. Payload data that is sent from web chat is stored in a private context variable named `context.integrations.chat.private.user_payload`. No private variables are sent from the dialog to any integrations. For more information about how to pass data, see [Passing sensitive data](/docs/assistant-data?topic=assistant-data-deploy-web-chat#deploy-web-chat-security-encrypt).

To access the payload data, you can reference the `context.integrations.chat.private.user_payload` object from the dialog node condition. 

You must know the structure of the JSON object that is sent in the payload.
{: note}

For example, if you passed the value `"mvp:true"` in the JSON payload, you can add a dialog flow that checks for this value to define a response that is meant for VIP customers only. Add a dialog node with a condition like this:

| Field | Value |
|-------|-------|
| If assistant recognizes | `$integrations.chat.private.user_payload.mvp` |
| Assistant responds | `I can help you reserve box seats at the upcoming conference!` |
{: caption="Private variable as node condition" caption-side="top"}

## Web chat: Accessing web browser information
{: #dialog-integrations-chat-browser-info}

When you use the web chat integration, information about the web browser that your customer is using to access the web chat is automatically collected and stored. The information is stored in the `context.integrations.chat.browser_info` object. 

You can design your dialog to take advantage of details about the web browser in use. The following properties are taken from the `window` object that represents the window in which the web chat is running:

- `browser_name`: The browser name, such as `chrome`, `edge`, or `firefox`.
- `browser_version`: The browser version, such as `80.0.0`.
- `browser_OS`: The operating system of the customer's computer, such as `Mac OS`.
- `language`: The default locale code of the browser, such as `en-US`.
- `page_url`: Full URL of the web page in which the web chat is embedded. For example: `https://www.example.com/products`
- `screen_resolution`: Specifies the height and width of the browser window in which the web page is displayed. For example: `width: 1440, height: 900`
- `user_agent`: Content from the User-Agent request header. For example: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:80.0) Gecko/20100101 Firefox/80.0`
- `client_ip_address`: IP address of the customer's computer. For example: `183.49.92.42`

### Example: Using page URL information in your dialog
{: #dialog-integrations-chat-browser-info-example}

You might embed the web chat in multiple pages on your website. Let's say you want to route chat transfers from the web chat to different Salesforce agents based on the page from which the customer asks to speak to someone. If the customer is on the insurance plans page of your website (`https://www.example.com/insurance.html`), you want to route them to agents who are experts in your company's insurance plans. If the customer is on the investments web page (`https://www.example.com/invest.html`), you want to route them to agents who are experts in your company's investment opportunities. You can add multiple conditioned responses to the dialog node where the transfer takes place and add logic like this:

#### Conditioned response 1

- Condition: `$integrations.chat.browser_info.page_url.contains('insurance.html')`
- Response type: *Connect to human agent*
- Button ID: `Z23453e25vv` (The button that routes to the insurance experts agent queue)

#### Conditioned response 2

- Condition: `$integrations.chat.browser_info.page_url.contains('invest.html')`
- Response type: *Connect to human agent*
- Button ID: `Z23453j24ty` (The button that routes to the investment experts agent queue)

For more information about implementing chat transfers, see [Adding a *Connect to human agent* response type](/docs/assistant-data?topic=assistant-data-dialog-overview#dialog-overview-multimedia-add). For more information about the `contains()` SpEL expression method, see [Expression language methods](https://cloud.ibm.com/docs/assistant-data?topic=assistant-data-dialog-methods).

## Web chat: Reusing the JWT for webhook authentication
{: #dialog-integrations-chat-jwt}

You can use the same JSON Web Token (JWT) that was used to secure your web chat to authenticate webhook calls. If you specify a token in the `identityToken` property when you add the web chat to your web page, the token is stored in a private variable named `context.integrations.chat.private.jwt`. For more information about passing a JWT, see [Enable security](/docs/assistant-data?topic=assistant-data-deploy-web-chat#deploy-web-chat-security-task).

1.  From your dialog skill, open the **Options>Webhooks** page.
1.  Click **Add authentication**. 
1.  Without adding any values to the fields in the *Basic authorization* modal that is displayed, click **Save**.

    An Authorization header is added for you.
1.  In the **Header value** field, add `$integrations.chat.private.jwt`.
