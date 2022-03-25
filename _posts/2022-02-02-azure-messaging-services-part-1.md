---
layout: post
title: "Azure Messaging Services Part 1 - Azure Event Grid"
author: Martijn van Schie
date: 2022-03-24 12:00:00 +0100
tags: azure event-grid messaging-service
cover-img: "/images/covers/clouds-modern-01.jpg"
thumbnail-img: "/images/icons/azure/event-grid-topic-lg.png"
---

# 1. Azure Messaging Services Series

## 1.1. Introduction

Azure provides an variety of messaging services which can be used to create advanced event driven architectures. The services include

- Azure Event Grid
- Azure Event Hub
- Azure Service Bus

This series will cover all three of these services in more detail and will focus on the core feature, usability and my opinion on how to use them withing your architecture.

Part one of the series focusses on Azure Event Grid so lets get right into it....

## 1.2. Table of content

- [1. Azure Messaging Services Series](#1-azure-messaging-services-series)
  - [1.1. Introduction](#11-introduction)
  - [1.2. Table of content](#12-table-of-content)
  - [1.3. Azure Event Grid](#13-azure-event-grid)
  - [1.4. Resources composition](#14-resources-composition)
    - [Event Publisher](#event-publisher)
    - [1.4.2. Event Grid Topics](#142-event-grid-topics)
    - [1.4.1. Subscribers](#141-subscribers)
    - [1.4.3. Event Handlers](#143-event-handlers)
  - [1.5. Message flow](#15-message-flow)
  - [1.6. Message content](#16-message-content)
  - [1.7. Reliability and Failover](#17-reliability-and-failover)

## 1.3. Azure Event Grid

Azure Event Grid is a messaging service which allows event publishers to send messages to subscribers. This means it can be used in a publish-subscribe architecture pattern.

With Azure Event Grid, messages are pushed to subscribers from the event grid. After receiving the message the subscriber send back an acknowledge response which tells the event grid that the message was delivered successfully.

## 1.4. Resources composition

Event Grid is the service which is between the **publisher** and **subscriber**. Using Event Grid requires a few resources and services working together. The following image shows an example of these resource and how they form the event chain

![Event Grid Message Flow!](/images/posts/20220202/event-grid-message-flow.png "Event Grid Message Flow")

A **resource** acts as a event source and sends a message to an **event topic**. Each Azure Event Topic can have one or more **Event Subscriptions**. These Event subscriptions are abstract resources that route events to target **Event Handlers** (or **subscriber**). 

### Event Publisher

An event publisher is either a Azure Resource which supports Azure Event Grid or any other client which is capable of sending http(s) requests. Microsoft offer a large set of SDK's, for different programming languages, to implement Event Grid messaging in your application. 

> I will dedicate a blog on developing against Azure Event Grid in the future.

### 1.4.2. Event Grid Topics

A topic can be considered a source of an event and is the unique, static, property of the message. Topics can be the name of the service, 

Azure offers two types of topic. 

- Event Grid Topics
- Event Grid System Topics

**Event Grid Topics** are user defined topics and can contain any information as long as it adheres to the [schema](https://docs.microsoft.com/en-us/azure/event-grid/event-schema) definition.

**Event Grid System Topics** are events that are defined by Azure and are send by specific Azure Resources. They contain a predefined `topic` and can only provide a preset of `eventtype`. They are also registered globally.

![Event Grid Message Flow!](/images/posts/20220202/event-grid-system-topic-lg.png "Event Grid Message Flow")

For more information about the available topics check the [System topics in Azure Event Grid](https://docs.microsoft.com/en-us/azure/event-grid/system-topics) page on Microsoft Docs.
### 1.4.1. Subscribers
Event Grid **Subscriptions** are not just routers, they actually offer a lot more functionality which can be configured on a per-subscription basis. This gives event handlers fine grained control over how the messages are delivered.

Below are the most common features of a subscription.

- Filter incoming events before being send to handlers
  - Filter on `subject`
  - Filter on other fixed properties like `id`, `topic`, `subject` or `eventtype`
  - Filter on custom properties inside the data payload
-  Configure the identity used for delivery
-  Configure Retry Policies
-  Enable dead-lettering
-  Configure subscription expiration time

### 1.4.3. Event Handlers

**Event Handlers** are the actually receivers of the event. Behind the scene a subscription is a router that send a https call to an endpoint. 

Event Handlers can be one of the pre-defined Azure Resources. Currently only the following Azure Services can be used as subscribers:

- Azure Function
- Storage Queue
- Event Hub
- Hybrid Connection
- Service Bus Queue
- Service Bus Topic

Other then that you can send the event or a generic Event Hook endpoint which pretty much opens up a lot of possibilities.

## 1.5. Message flow

So what does the message flow look like.

Although this model feels like a publish-subscribe pattern it is slightly different. The event grid has a intermediate concept called 

## 1.6. Message content

Compared to the other messaging services the payload of the Event Grid message is limited and contains only the minimal required information about the event that occurred for subscribers to acts on this event. It also follows a well defines [schema](https://docs.microsoft.com/en-us/azure/event-grid/event-schema) which promoted standardization.

## 1.7. Reliability and Failover 

So what does this all mean for reliability and failover. Well event grid does have a mechanisms for this. It will try to send, and resend, the event message to the subscriber. This is done withing the configured events expiration time which is a max of 24 hours. After that the message is either discarded of dead-lettered.

As you might expect from this pattern, event grid only offers guaranteed delivery withing a specific time frame. Also, it does not guarantee successfully processing nor does it have replay capabilities *after* a successfully delivery.

This means that if the processing of the message would somehow fail on the receivers side, the event message is gone and can not be retrieved. This means requirements like failover and reliability should be implemented as part of the subscribers solution.

One thing you can 