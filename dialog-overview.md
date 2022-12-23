---

copyright:
  years: 2015, 2020
lastupdated: "2022-12-23"

subcollection: assistant-data

---

{:shortdesc: .shortdesc}
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
{:table: .aria-labeledby="caption"}

# Building a conversational flow
{: #dialog-overview}

The dialog defines what your assistant says in response to customers.
{: shortdesc}

## Creating a dialog
{: #dialog-build-task}

To create a dialog, complete the following steps:

1.  Click the **Dialog** tab, and then click **Create dialog**.

    When you open the dialog editor for the first time, the following nodes are created for you:

    - **Welcome**: The first node. It contains a greeting that is displayed to your users when they first engage with your assistant. You can edit the greeting.

    This node is not triggered in dialog flows that are initiated by users. For example, dialogs that are deployed in environments such as messaging channels where customers start the conversation do not process nodes with the `welcome` special condition.
    {: note}

    - **Anything else**: The final node. It contains phrases that are used to reply to users when their input is not recognized. You can replace the responses that are provided or add more responses with a similar meaning to add variety to the conversation. You can also choose whether you want your assistant to return each response that is defined in turn or return them in random order.
1.  To add more nodes to the dialog tree, click the **More** ![More icon](images/kabob.png) icon on the **Welcome** node, and then select **Add node below**.
1.  In the **If assistant recognizes** field, enter a condition that, when met, triggers your assistant to process the node.

    To start off, you typically want to add an intent as the condition. For example, if you add `#open_account` here, it means that you want the response that you will specify in this node to be returned to the user if the user input indicates that the user wants to open an account.

    As you begin to define a condition, a box is displayed that shows you your options. You can enter one of the following characters, and then pick a value from the list of options that is displayed.

    <table>
    <caption>Condition builder syntax</caption>
    <tr>
      <th>Character</th>
      <th>Lists defined values for these artifact types</th>
    </tr>
    <tr>
      <td>`#`</td>
      <td>intents</td>
    </tr>
    <tr>
      <td>`@`</td>
      <td>entities</td>
    </tr>
    <tr>
      <td>`@{entity-name}:`</td>
      <td>{entity-name} values</td>
    </tr>
    <tr>
      <td>`$`</td>
      <td>context-variables that you defined or referenced elsewhere in the dialog</td>
    </tr>
    </table>

    You can create a new intent, entity, entity value, or context variable by defining a new condition that uses it. If you create an artifact this way, be sure to go back and complete any other steps that are necessary for the artifact to be created completely, such as defining sample utterances for an intent.

    To define a node that triggers based on more than one condition, enter one condition, and then click the plus sign (+) icon next to it. If you want to apply an `OR` operator to the multiple conditions instead of `AND`, click the `and` that is displayed between the fields to change the operator type. AND operations are executed before OR operations, but you can change the order by using parentheses. For example:
    `$isMember:true AND ($memberlevel:silver OR $memberlevel:gold)`

    The condition you define must be less than 2,048 characters in length.

    For more information about how to test for values in conditions, see [Conditions](/docs/assistant-data?topic=assistant-data-dialog-overview#dialog-overview-conditions).
1.  **Optional**: If you want to collect multiple pieces of information from the user in this node, then click **Customize** and enable **Slots**. See [Gathering information with slots](/docs/assistant-data?topic=assistant-data-dialog-slots) for more details.
1.  Enter a response.

    - Add the text or multimedia elements that you want your assistant to display to the user as a response.
    - If you want to define different responses based on certain conditions, then click **Customize** and enable **Multiple responses**.
    - For information about conditional responses, rich responses, or how to add variety to responses, see [Responses](/docs/assistant-data?topic=assistant-data-dialog-overview#dialog-overview-responses).

1.  Specify what to do after the current node is processed. You can choose from the following options:

    - **Wait for user input**: Your assistant pauses until new input is provided by the user.
    - **Skip user input**: Your assistant jumps directly to the first child node. This option is only available if the current node has at least one child node.
    - **Jump to**: Your assistant continues the dialog by processing the node you specify. You can choose whether your assistant should evaluate the target node's condition or skip directly to the target node's response. See [Configuring the Jump to action](/docs/assistant-data?topic=assistant-data-dialog-overview#dialog-overview-jump-to-config) for more details.

1.  **Optional**: If you want this node to be considered when users are shown a set of node choices at run time, and asked to pick the one that best matches their goal, then add a short description of the user goal handled by this node to the **external node name** field. For example, *Open an account*.

    See [Disambiguation](/docs/assistant-data?topic=assistant-data-dialog-runtime#dialog-runtime-disambiguation) for more details.

1.  **Optional**: Name the node.

    The dialog node name can contain letters (in Unicode), numbers, spaces, underscores, hyphens, and periods.

    Naming the node makes it easier for you to remember its purpose and to locate the node when it is minimized. If you don't provide a name, the node condition is used as the name.

1.  To add more nodes, select a node in the tree, and then click the **More** ![More icon](images/kabob.png) icon.

    - To create a peer node that is checked next if the condition for the existing node is not met, select **Add node below**.
    - To create a peer node that is checked before the condition for the existing node is checked, select **Add node above**.
    - To create a child node to the selected node, select **Add child node**. A child node is processed after its parent node.
    - To copy the current node, select **Duplicate**.

    For more information about the order in which dialog nodes are processed, see [Dialog overview](/docs/assistant-data?topic=assistant-data-dialog-build#dialog-build-flow).
1.  Test the dialog as you build it.

    See [Testing your dialog](#dialog-build-test) for more information.

## Conditions
{: #dialog-overview-conditions}

A node condition determines whether that node is used in the conversation. Response conditions determine which response to return to a user.

- [Condition artifacts](#dialog-overview-condition-artifacts)
- [Special conditions](#dialog-overview-special-conditions)
- [Condition syntax details](#dialog-overview-condition-syntax)

For tips on performing more advanced actions in conditions, see [Condition usage tips](/docs/assistant-data?topic=assistant-data-dialog-tips#dialog-tips-condition-usage).

### Condition artifacts
{: #dialog-overview-condition-artifacts}

You can use one or more of the following artifacts in any combination to define a condition:

- **Context variable**: The node is used if the context variable expression that you specify is true. Use the syntax, `$variable_name:value` or `$variable_name == 'value'`.

  For node conditions, this artifact type is typically used with an AND or OR operator and another condition value. That's because something in the user input must trigger the node; the context variable value being matched alone is not enough to trigger it. If the user input object sets the context variable value somehow, for example, then the node is triggered.

  Do not define a node condition based on the value of a context variable in the same dialog node in which you set the context variable value.
  {: tip}

  For response conditions, this artifact type can be used alone. You can change the response based on a specific context variable value. For example, `$city:Boston` checks whether the `$city` context variable contains the value, `Boston`. If so, the response is returned.
  
  For more information about context variables, see [Context variables](/docs/assistant-data?topic=assistant-data-dialog-runtime-context#dialog-runtime-context-variables).

- **Entity**: The node is used when any value or synonym for the entity is recognized in the user input. Use the syntax, `@entity_name`. For example, `@city` checks whether any of the city names that are defined for the @city entity were detected in the user input. If so, the node or response is processed.

  Consider creating a peer node to handle the case where none of the entity's values or synonyms are recognized.
  {: tip}

  For more information about entities, see [Defining entities](/docs/assistant-data?topic=assistant-data-entities).

- **Entity value**: The node is used if the entity value is detected in the user input. Use the syntax, `@entity_name:value` and specify a defined value for the entity, not a synonym. For example: `@city:Boston` checks whether the specific city name, `Boston`, was detected in the user input.

  If you check for the presence of the entity, without specifying a particular value for it, in a peer node, be sure to position this node (which checks for a particular entity value) before the peer node that checks only for the presence of the entity. Otherwise, this node will never be evaluated.
  {: tip}

  If the entity is a pattern entity with capture groups, then you can check for a certain group value match. For example, you can use the syntax: `@us_phone.groups[1] == '617'`
  See [Storing and recognizing pattern entity groups in input](/docs/assistant-data?topic=assistant-data-dialog-tips#dialog-tips-get-pattern-groups) for more information.

- **Intent**: The simplest condition is a single intent. The node is used if, after your assistant's natural language processing evaluates the user's input, it determines that the purpose of the user's input maps to the pre-defined intent. Use the syntax, `#intent_name`. For example, `#weather` checks if the user input is asking for a weather forecast. If so, the node with the `#weather` intent condition is processed.

  For more information about intents, see [Defining intents](/docs/assistant-data?topic=assistant-data-intents).

- **Special condition**: Conditions that are provided with the product that you can use to perform common dialog functions. See the **Special conditions** table in the next section for details.

### Special conditions
{: #dialog-overview-special-conditions}

| Condition syntax     | Description |
|----------------------|-------------|
| `anything_else`      | You can use this condition at the end of a dialog, to be processed when the user input does not match any other dialog nodes. The **Anything else** node is triggered by this condition. |
| `conversation_start` | Like **welcome**, this condition is evaluated as true during the first dialog turn. Unlike **welcome**, it is true whether or not the initial request from the application contains user input. A node with the **conversation_start** condition can be used to initialize context variables or perform other tasks at the beginning of the dialog. |
| `false`              | This condition is always evaluated to false. You might use this at the start of a branch that is under development, to prevent it from being used, or as the condition for a node that provides a common function and is used only as the target of a **Jump to** action. |
| `irrelevant`         | This condition will evaluate to true if the user’s input is determined to be irrelevant by the {{site.data.keyword.conversationshort}} service. |
| `true`               | This condition is always evaluated to true. You can use it at the end of a list of nodes or responses to catch any responses that did not match any of the previous conditions. |
| `welcome`            | This condition is evaluated as true during the first dialog turn (when the conversation starts), only if the initial request from the application does not contain any user input. It is evaluated as false in all subsequent dialog turns. The **Welcome** node is triggered by this condition. Typically, a node with this condition is used to greet the user, for example, to display a message such as `Welcome to our Pizza ordering app.` |
{: caption="Special conditions" caption-side="top"}

### Condition syntax details
{: #dialog-overview-condition-syntax}

Use one of these syntax options to create valid expressions in conditions:

- Shorthand notations to refer to intents, entities, and context variables. See [Accessing and evaluating objects](/docs/assistant-data?topic=assistant-data-expression-language).

- Spring Expression (SpEL) language, which is an expression language that supports querying and manipulating an object graph at run time. See [Spring Expression Language (SpEL) language](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html){: external} for more information.

You can use regular expressions to check for values to condition against.  To find a matching string, for example, you can use the `String.find` method. See  [Methods](/docs/assistant-data?topic=assistant-data-dialog-methods) for more details.

## Responses
{: #dialog-overview-responses}

The dialog response defines how to reply to the user.

You can reply in the following ways:

- [Simple text response](#dialog-overview-simple-text)
- [Rich responses](#dialog-overview-multimedia)
- [Conditional responses](#dialog-overview-multiple)

### Simple text response
{: #dialog-overview-simple-text}

If you want to provide a text response, simply enter the text that you want your assistant to display to the user.

![Shows a node that shows a user ask, Where are you located, and the dialog response is, We have no brick and mortar stores! But, with an internet connection, you can shop us from anywhere.](images/response-simple.png)

To include a context variable value in the response, use the syntax `$variable_name` to specify it. See [Context variables](/docs/assistant-data?topic=assistant-data-dialog-runtime-context#dialog-runtime-context-variables) for more information. For example, if you know that the $user context variable is set to the current user's name before a node is processed, then you can refer to it in the text response of the node like this:

```
Hello $user
```
{: screen}

If the current user's name is `Norman`, then the response that is displayed to Norman is `Hello Norman`.

If you include one of these special characters in a text response, escape it by adding a backslash (`\`) in front of it. If you are using the JSON editor, you need to use two backslashes to escape (`\\`). Escaping the character prevents your assistant from misinterpreting it as being one of the following artifact types:

| Special character | Artifact | Example |
|-------------------|----------|---------|
| `$` | Context variable | `The transaction fee is \$2.` |
| `@` | Entity | `Send us your feedback at feedback\@example.com.` |
| `#` | Intent | `We are the \#1 seller of lobster rolls in Maine.` |
{: caption="Special characters to escape in responses" caption-side="top"}

To include a hypertext link that is rendered in the "Try it out" pane, you can use HTML syntax. For example: `Contact us at <a href="https://www.ibm.com">ibm.com</a>.` (Do *not* try to escape the quotations mark with a backslash `\"`, for example.)
{: note}

#### Learn more about simple responses
{: #dialog-overview-variety}

- [Adding multiple lines](#dialog-overview-multiline)
- [Adding variety](#dialog-overview-add-variety)

#### Adding multiple lines
{: #dialog-overview-multiline}

If you want a single text response to include multiple lines separated by carriage returns, then follow these steps:

1.  Add each line that you want to show to the user as a separate sentence into its own response variation field. For example:

  <table>
  <caption>Multiple line response</caption>
  <tr>
    <th>Response variations</th>
  </tr>
  <tr>
    <td>Hi.</td>
  </tr>
  <tr>
    <td>How are you today?</td>
  </tr>
  </table>

1.  For the response variation setting, choose **multiline**.

    If you are using a dialog skill that was created before support for rich response types was added to the product, then you might not see the *multiline* option. Add a second text response type to the current node response. This action changes how the response is represented in the underlying JSON. As a result, the multiline option becomes available. Choose the multiline variation type. Now, you can delete the second text response type that you added to the response.
    {: note}

When the response is shown to the user, both response variations are displayed, one on each line, like this:

```
Hi.
How are you today?
```
{: screen}

#### Adding variety
{: #dialog-overview-add-variety}

If your users return to your conversation service frequently, they might be bored to hear the same greetings and responses every time.  You can add *variations* to your responses so that your conversation can respond to the same condition in different ways.

In this example, the answer that your assistant provides in response to questions about store locations differs from one interaction to the next:

![Shows a node that shows a user ask, Where are you located, and the dialog has three different responses defined.](images/variety.png)

You can choose to rotate through the response variations sequentially or in random order. By default, responses are rotated sequentially, as if they were chosen from an ordered list.

To change the sequence in which individual text responses are returned, complete the following steps:

1.  Add each variation of the response into its own response variation field. For example:

  <table>
  <caption>Varying responses</caption>
  <tr>
    <th>Response variations</th>
  </tr>
  <tr>
    <td>Hello.</td>
  </tr>
  <tr>
    <td>Hi.</td>
  </tr>
  <tr>
    <td>Howdy!</td>
  </tr>
  </table>

1.  For the response variation setting, choose one of the following settings:

    - **sequential**: The system returns the first response variation the first time the dialog node is triggered, the second response variation the second time the node is triggered, and so on, in the same order as you define the variations in the node.

      Results in responses being returned in the following order when the node is processed:

      - First time:

        ```
        Hello.
        ```
        {: screen}

      - Second time:

        ```
        Hi.
        ```
        {: screen}

      - Third time:
        ```
        Howdy!
        ```
        {: screen}

    - **random**: The system randomly selects a text string from the variations list the first time the dialog node is triggered, and randomly selects another variation the next time, but without repeating the same text string consecutively.

      Example of the order that responses might be returned in when the node is processed:

      - First time:

        ```
        Howdy!
        ```
        {: screen}

      - Second time:

        ```
        Hi.
        ```
        {: screen}

      - Third time:

        ```
        Hello.
        ```
        {: screen}

### Rich responses
{: #dialog-overview-multimedia}

You can return responses with multimedia or interactive elements such as images or clickable buttons to simplify the interaction model of your application and enhance the user experience.

In addition to the default response type of **Text**, for which you specify the text to return to the user as a response, the following response types are supported:

- **Connect to human agent**: The dialog calls a service that you designate, typically a service that manages human agent support ticket queues, to pass off the conversation to a person. You can optionally include a message that summarizes the user's issue to be provided to the human agent. It is the responsibility of the external service to display a message that is shown to the user that explains that the conversation is being transferred. The dialog does not manage that communication itself. The dialog transfer does not occur when you are testing nodes with this response type in the "Try it out" pane. You must access a node that uses this response type from a test deployment to see how your users will experience it.

- **Image**: Embeds an image into the response. The source image file must be hosted somewhere and have a URL that you can use to reference it. It cannot be a file that is stored in a directory that is not publicly accessible.
- **Option**: Adds a list of one or more options. When a user clicks one of the options, an associated user input value is sent to your assistant. How options are rendered can differ depending on where you deploy the dialog. For example, in one integration channel the options might be displayed as clickable buttons, but in another they might be displayed as a dropdown list.
- **Pause**: Forces the application to wait for a specified number of milliseconds before continuing with processing. You can choose to show an indicator that the dialog is working on typing a response. Use this response type if you need to perform an action that might take some time. For example, a parent node makes a Cloud Function call and displays the result in a child node. You could use this response type as the response for the parent node to give the programmatic call time to complete, and then jump to the child node to show the result. This response type does not render in the "Try it out" pane. You must access a node that uses this response type from a test deployment to see how your users will experience it.
- **Search skill**: Searches an external data source for relevant information to return to the user. The data source that is searched is a {{site.data.keyword.discoveryshort}} service data collection that you configure when you add a search skill to the assistant that uses this dialog skill. For more information, see [Creating a search skill](/docs/assistant-data?topic=assistant-data-skill-search-add).

#### Adding rich responses
{: #dialog-overview-multimedia-add}

To add a rich response, complete the following steps:

1.  Click the drop-down menu in the response field to choose a response type, and then provide any required information:

    - **Connect to human agent**. You can optionally add a message to share with the human agent to whom the conversation is transferred.

        You must program the client application to recognize when this response type is triggered.
        {: note}

    - **Image**. Add the full URL to the hosted image file into the **Image source** field. The image must be in .jpg, .gif, or .png format. The image file must be stored in a location that is publicly addressable by URL.

        For example: `https://www.example.com/assets/common/logo.png`.

        If you want to display an image title and description before the embedded image in the response, then add them in the fields provided.

    - **Option**. Complete the following steps:

      1.  Click **Add option**.
      1.  In the **List label** field, enter the option to display in the list. The label must be less than 2,048 characters in length.
      1.  In the corresponding **Value** field, enter the user input to pass to your assistant when this option is selected. The value must be less than 2,048 characters in length. (A current limitation applies a 64-character limit, but is being addressed.)

          Specify a value that you know will trigger the correct intent when it is submitted. For example, it might be a user example from the training data for the intent.
      1.  Repeat the previous steps to add more options to the list.

          You can add up to 20 options.
      1.  Add a list introduction in the **Title** field. The title can ask the user to pick from the list of options.

          Some integration channels do not display the title.
          {: note}

      1.  Optionally, add additional information in the **Description** field. If specified, the description is displayed after the title and before the option list.

      Some integration channels do not display the description.
      {: note}

      For example, you can construct a response like this:

        <table>
        <caption>Response options</caption>
        <tr>
          <th>List title</th>
          <th>List description</th>
          <th>Option label</th>
          <th>User input submitted when clicked</th>
        </tr>
        <tr>
          <td>Insurance types</td>
          <td>Which of these items do you want to insure?</td>
          <td></td>
          <td></td>
        </tr>
        <tr>
          <td></td>
          <td></td>
          <td>Boat</td>
          <td>I want to buy boat insurance</td>
        </tr>
        <tr>
          <td></td>
          <td></td>
          <td>Car</td>
          <td>I want to buy car insurance</td>
        </tr>
         <tr>
          <td></td>
          <td></td>
          <td>Home</td>
          <td>I want to buy home insurance</td>
        </tr>
        </table>

    - **Pause**. Add the length of time for the pause to last as a number of milliseconds (ms) to the **Duration** field.

        The value cannot exceed 10,000 ms. Users are typically willing to wait about 8 seconds (8,000 ms) for someone to enter a response. To prevent a typing indicator from being displayed during the pause, choose **Off**.

        Add another response type, such as a text response type, after the pause to clearly denote that the pause is over.
        {: tip}

    - **Text**. Add the text to return to the user in the text field. Optionally, choose a variation setting for the text response. See [Simple text response](#dialog-overview-simple-text) for more details.

    - **Search skill**. Indicates that you want to search an external data source for a relevant response.

      To edit the search query to pass to the {{site.data.keyword.discoveryshort}} service, click **Customize**, and then fill in the following fields:

        - **Query**: Optional. You can specify a specific query in natural language to pass to {{site.data.keyword.discoveryshort}}. If you do not add a query, then the customer's exact input text is passed as the query.

          For example, you can specify `What cities do you fly to?`. This query value is passed to {{site.data.keyword.discoveryshort}} as a search query. {{site.data.keyword.discoveryshort}} uses natural language understanding to understand the query and to find an answer or relevant information about the subject in the data collection that is configured for the search skill.

          You can include specific information provided by the user by referencing entities that were detected in the user's input as part of the query. For example, `Tell me about @product`. Or you can reference a context variable, such as `Do you have flights to $destination?`. Just be sure to design your dialog such that the search is not triggered unless any entities or context variables that you reference in the query have been set to valid values.

          This field is equivalent to the {{site.data.keyword.discoveryshort}} `natural_language_query` parameter. For more information, see [Query parameters](/docs/discovery?topic=discovery-query-parameters#nlq){: external}.

        - **Filter**: Optional. Specify a text string that defines information that must be present in any of the search results that are returned.

          - To indicate that you want to return only documents with positive sentiment detected, for example, specify `enriched_text.sentiment.document.label:positive`.

          - To filter results to includes only documents that the ingestion process identified as containing the entity `Boston, MA`, specify `enriched_text.entities.text:"Boston, MA"`.

          - To filter results to includes only documents that the ingestion process identified as containing a product name provided by the customer, you can specify `enriched_text.entities.text:@product`.

          - To filter results to includes only documents that the ingestion process identified as containing a city name that you saved in a context variable named `$destination`, you can specify `enriched_text.entities.text:$destination`.

        If you add both a query and a filter value, then the filter parameter is applied first to filter the data collection documents and cache the results. The query parameter then ranks the cached results. 

        This field is equivalent to the {{site.data.keyword.discoveryshort}} `filter` parameter. For more information, see [Query parameters](/docs/discovery?topic=discovery-query-parameters#filter){: external}.

      This response type only returns a valid response if the assistant to which you added this dialog skill also has a search skill associated with it.

1.  Click **Add response** to add another response type to the current response.

    You might want to add multiple response types to a single response to provide a richer answer to a user query. For example, if a user asks for store locations, you could show a map and display a button for each store location that the user can click to get address details. To build that type of response, you can use a combination of image, options, and text response types. Another example is using a text response type before a pause response type so you can warn users before pausing the dialog.

    You cannot add more than 5 response types to a single response. Meaning, if you define three conditional responses for a dialog node, each conditional response can have no more than 5 response types added to it.
    {: note}

    A single dialog node cannot have more than one **Connect to human agent** or more than one **Search skill** response.
    {: note}

1.  If you added more than one response type, you can click the **Move** up or down arrows to arrange the response types in the order you want your assistant to process them.

### Conditional responses
{: #dialog-overview-multiple}

A single dialog node can provide different responses, each one triggered by a different condition.  Use this approach to address multiple scenarios in a single node.

<iframe class="embed-responsive-item" id="youtubeplayer1" title="Adding conditional responses" type="text/html" width="640" height="390" src="https://www.youtube.com/embed/Q5_-f7_Iyvg?rel=0" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen> </iframe>

The node still has a main condition, which is the condition for using the node and processing the conditions and responses that it contains.

In this example, your assistant uses information that it collected earlier about the user's location to tailor its response, and provide information about the store nearest the user. See [Context variables](/docs/assistant-data?topic=assistant-data-dialog-runtime-context#dialog-runtime-context-variables) for more information about how to store information collected from the user.

![Shows a node that shows a user ask, Where are you located, and the dialog has three different responses depending on conditions that use info from the $state context variable to specify locations in those states.](images/multiple-responses.png)

This single node now provides the equivalent function of four separate nodes.

To add conditional responses to a node, complete the following steps:

1.  Click **Customize**, and then click the **Multiple responses** toggle to turn it **On**.

    The node response section changes to show a pair of condition and response fields. You can add a condition and a response into them.
1.  To customize a response further, click the **Edit response** ![Edit response](images/edit-slot.png) icon next to the response.

    You must open the response for editing to complete the following tasks:

    - **Update context**. To change the value of a context variable when the response is triggered, specify the context value in the context editor. You update context for each individual conditional response; there is no common context editor or JSON editor for all conditional responses.
    - **Add rich responses**. To add more than one text response or to add response types other than text responses to a single conditional response, you must open the edit response view.
    - **Configure a jump**. To instruct your assistant to jump to a different node after this conditional response is processed, select **Jump to** from the *And finally* section of the response edit view. Identify the node that you want your assistant to process next. See [Configuring the Jump to action](#dialog-overview-jump-to-config) for more information.

      A **Jump to** action that is configured for the node is not processed until all of the conditional responses are processed. Therefore, if a conditional response is configured to jump to another node, and the conditional response is triggered, then the jump configured for the node is never processed, and so does not occur.

1.  Click **Add response** to add another conditional response.

The conditions within a node are evaluated in order, just as nodes are.  Be sure that your conditional responses are listed in the correct order.  If you need to change the order, select a condition and response pair and move it up or down in the list using the arrows that are displayed.

## Defining what to do next
{: #dialog-overview-jump-to}

After making the specified response, you can instruct your assistant to do one of the following things:

- **Wait for user input**: Your assistant waits for the user to provide new input that the response elicits. For example, the response might ask the user a yes or no question. The dialog will not progress until the user provides more input.
- **Skip user input**:  Use this option when you want to bypass waiting for user input and go directly to the first child node of the current node instead.

  The current node must have at least one child node for this option to be available.
  {: note}

- **Jump to another dialog node**: Use this option when you want the conversation to go directly to an entirely different dialog node. You can use a *Jump to* action to route the flow to a common dialog node from multiple locations in the tree, for example.

  The target node that you want to jump to must exist before you can configure the jump to action to use it.
  {: note}

### Configuring the Jump to action
{: #dialog-overview-jump-to-config}

If you choose to jump to another node, specify when the target node is processed by choosing one of the following options:

- **Condition**: If the statement targets the condition section of the selected dialog node, your assistant checks first whether the condition of the targeted node evaluates to true.
    - If the condition evaluates to true, the system processes the target node immediately.
    - If the condition does not evaluate to true, the system moves to the next sibling node of the target node to evaluate its condition, and repeats this process until it finds a dialog node with a condition that evaluates to true.

    - If the system processes all the siblings and none of the conditions evaluate to true, the basic fallback strategy is used, and the dialog evaluates the nodes at the base level of the dialog tree.

    Targeting the condition is useful for chaining the conditions of dialog nodes. For example, you might want to first check whether the input contains an intent, such as `#turn_on`, and if it does, you might want to check whether the input contains entities, such as `@lights`, `@radio`, or `@wipers`. Chaining conditions helps to structure larger dialog trees.

    Avoid choosing this option when configuring a jump-to from a conditional response that goes to a node situated before the current node in the dialog tree. Otherwise, you can create an infinite loop. If your assistant jumps to the earlier node and checks its condition, it is likely to return false because the same user input is being evaluated that triggered the current node last time through the dialog. Your assistant will go to the next sibling or back to root to check the conditions on those nodes, and will likely end up triggering this node again, which means the process will repeat itself.
    {: note}

- **Response**: If the statement targets the response section of the selected dialog node, it is run immediately. That is, the system does not evaluate the condition of the selected dialog node; it processes the response of the selected dialog node immediately.

  Targeting the response is useful for chaining several dialog nodes together. The response is processed as if the condition of this dialog node is true. If the selected dialog node has another **Jump to** action, that action is run immediately, too.

- **Wait for user input**: Waits for new input from the user, and then begins to process it from the node that you jump to. This option is useful if the source node asks a question, for example, and you want to jump to a separate node to process the user's answer to the question.

## Next steps
{: #dialog-overview-next}

- Be sure to test your dialog as you build it. For more information, see [Testing the dialog](/docs/assistant-data?topic=assistant-data-dialog-tasks).
- For more information about ways to address common use cases, see [Dialog building tips](/docs/assistant-data?topic=assistant-data-dialog-tips).
- For more information about the expression language that you can use to improve your dialog, such as methods that reformat dates or text, see [Expression language methods](/docs/assistant-data?topic=assistant-data-dialog-methods).


**Previous topic:** [Planning the dialog](/docs/assistant-data?topic=assistant-data-dialog-plan)

**Next topic:** [Personalizing the dialog with context](/docs/assistant-data?topic=assistant-data-dialog-runtime-context)
