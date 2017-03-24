---
title: Key concepts in the Bot Builder SDK for Node.js | Microsoft Docs
description: Learn about key concepts in the Bot Builder SDK for Node.js.
keywords: Bot Framework, Node.js, Bot Builder, SDK, key concepts, core concepts
author: DeniseMak
manager: rstand
ms.topic: develop-nodejs-article
ms.prod: botframework
ms.service: Bot Builder
ms.date: 03/08/2017
ms.reviewer:
#ROBOTS: Index
---

# Key concepts in the Bot Builder SDK for Node.js

This article introduces key concepts in the Bot Builder SDK for Node.js.

## Connector

> [!NOTE]
> This content will be updated.

The Bot Framework Connector is a service that connects your bot to multiple channels such as Skype, Facebook
, Slack, and more. 
It facilitates communication between bot and user, by relaying messages from bot to channel and from channel to bot. 

The Bot Builder SDK for Node.js provides the **UniversalBot** and **ChatConnector** classes for configuring the bot to send and receive messages through the Bot Framework Connector.
For an example that demonstrates using these classes, see [Create a bot with the Bot Builder SDK for Node.js](bot-framework-nodejs-getstarted.md).


## Messages and Dialogs

Messages can consist of text strings, attachments, and rich cards. You use the [send][SessionSend] method to send messages in response to a message from the user. Your bot may call **send()** as many times as it likes in response to a message from the user. <!--TODO: What does "as many times mean"? --> For a guide to how to use dialogs and message handlers to manage conversation flow, see [Manage conversation flow](bot-framework-nodejs-howto-manage-conversation-flow.md).

For an example that demonstrates how to send a rich graphical card containing interactive buttons that the user clicks to initiate an action, see [Send a rich card](bot-framework-nodejs-howto-send-card-buttons.md). For an example that demonstrates how to send and receive attachments, see [Send attachments](bot-framework-nodejs-howto-send-receive-attachments.md).

<!-- TODO: Make Dialogs it's own section -->

## Trigger actions
You'll want to design your bot to be able to handle interruptions like requests for cancellation or help at any time during the conversation flow. The Bot Builder SDK for Node.js provides global message handlers that trigger actions like cancellation or the invokation of other dialogs. 
 See [Handling cancel](bot-framework-nodejs-howto-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-framework-nodejs-howto-manage-conversation-flow.md#confirming-interruptions) and [Trigger actions using global handlers](bot-framework-nodejs-howto-global-handlers.md) for examples of how to use [triggerAction][triggerAction] handlers.


## Recognizers
When the users ask your bot for something, like "help" or "find news", your bot needs to understand what the user is asking for. You can design your bot to recognize a set of intents that interpret the user’s input in terms of the intention it conveys. You can use implement a custom recognizer, use the built-in regular expression recognizer that the Bot Builder SDK provides, or call an external service such as the LUIS API to determine the user's intent.
See [Recognize user intent](bot-framework-nodejs-howto-recognize-intent.md) for an example of implementing a custom recognizer.

<!-- TODO -- move Message ordering to separate 300-level topic -->

## Saving State

A key to good bot design is to track the context of a conversation, so that your bot remembers things like the last question the user asked. 
Bots built using Bot Builder SDK are designed to be be stateless so that they can easily be scaled to run across multiple compute nodes. The Bot Framework provides a storage system that stores bot data, so that the bot web service can be scaled. Because of that you should generally avoid the temptation to save state using a global variable or function closure. Doing so will create issues when you want to scale out your bot. Instead, use the following properties of your bot's [session][Session] object to save data relative to a user or conversation:

* **userData** stores information globally for the user across all conversations.
* **conversationData** stores information globally for a single conversation. This data is visible to everyone within the conversation so care should be used to what’s stored there. It’s disabled by default and needs to be enabled using the bots [persistConversationData][PersistConversationData] setting.
* **privateConversationData** stores information globally for a single conversation but it is private data specific to the current user. This data spans all dialogs so it’s useful for storing temporary state that you want cleaned up when the conversation ends.
* **dialogData** persists information for a single dialog instance. This is essential for storing temporary information in between the steps of a [waterfall](bot-framework-nodejs-howto-manage-conversation-flow.md#ask-questions) in a dialog.

See [Saving user data](bot-framework-nodejs-howto-save-user-data.md) for an example that demonstrates how to save user data.


## Next steps

Build your first bot by following the steps at [Get started](bot-framework-nodejs-getstarted.md).


## Additional Resources

* [Manage conversation flow](bot-framework-nodejs-howto-manage-conversation-flow.md)
* [Triggering actions](bot-framework-nodejs-howto-global-handlers.md)
* [Recognize user intent](bot-framework-nodejs-howto-recognize-intent.md)
* [Send a rich card](bot-framework-nodejs-howto-send-card-buttons.md)
* [Send attachments](bot-framework-nodejs-howto-send-receive-attachments.md)
* [Saving user data](bot-framework-nodejs-howto-save-user-data.md)
* [How to send a typing indicator](bot-framework-nodejs-howto-send-typing-indicator.md)
* UniversalBot
* ChatConnector
* [session object][Session]
* [session.send][SessionSend]
* [session.sendTyping][SessionSendTyping]
* [triggerAction][triggerAction]


<!-- TODO: Update links to point to new docs -->

[PersistConversationData]:(https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata)

[Session]: (https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[SessionSendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#sendtyping
[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]:(../articles/bot-framework-nodejs-howto-manage-conversation-flow.md#ask-questions)
[SaveUserData]:(../articles/bot-framework-nodejs-howto-save-user-data.md)
[GetStarted]:(../articles/bot-framework-nodejs-getstarted.md)


