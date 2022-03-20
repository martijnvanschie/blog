---
layout: post
title: "HowTo: Process Azure Subscription System Topics using Azure Function"
author: Martijn van Schie
date: 2022-02-02 12:00:00 +0100
categories: tutorial howto
tags: azure azure-functions event-grid csharp dotnet
cover-img: "/images/azure-cover-01.jpg"
thumbnail-img: "/images/svgexport-114.png"
---

## Introduction

We can use Azure Functions to as event handlers for Event Grid events. The basic implementation is very easy but when you want to handle system events you have to do some manual work to get it to work.

This blog will guide you through the steps on how to create an Azure Function that handles Event Grid System Topic.

### The Azure Function

This is what the content of the function looks like.

```csharp
[FunctionName("EventGridFunction")]
public void Run([EventGridTrigger] EventGridEvent e)
{
    _logger.LogInformation("Event Type:  {type}", e.EventType);
    _logger.LogInformation("Event subject: {subject}", e.Subject);
    _logger.LogInformation("Event topic: {topic}", e.Topic);
    _logger.LogInformation("Event content: {content}", e.Data);

    var evnt = e.Data.ToObjectFromJson<ResourceEvent>(_options);
}
```

### Resource Event Grid

The event grid event passed as an argument has an untyped data property. To get a typed object we make us of the helper function `ToObjectFromJson()`. This will parse the Data property into a custom class. It uses the classed and methods from the `System.Text.Json` namespace so we can use classes like `JsonSerializerOptions()` to control the serialization of the JSON.

```csharp
public class ResourceEvent
{
    public string ResourceProvider { get; set; }
    public string ResourceUri { get; set; }
    public string OperationName { get; set; }
    public string Status { get; set; }
    public string SubscriptionId { get; set; }
    public string TenantId { get; set; }
}
```

### Testing the Azure Function

The following url will send the event to the function `http://localhost:7071/runtime/webhooks/eventgrid?functionName=EventGridFunction`

Headers:

* `aeg-event-type : Notification`

Below is an example request containing an Azure Subscriptin Event.

```json
[
    {
        "topic": "/subscriptions/{subscriptionId}",
        "subject": "/subscriptions/{subscriptionId}/resourceGroups/event-trigger",
        "eventType": "Microsoft.Resources.ResourceWriteSuccess",
        "eventTime": "2022-02-02T17:02:19.6069787Z",
        "id": "{guid}",
        "data": {
            "authorization": {
                "scope": "/subscriptions/{subscriptionId}/resourceGroups/event-trigger",
                "action": "Microsoft.Resources/subscriptions/resourceGroups/write",
                "evidence": {
                    "role": "Subscription Admin"
                }
            },
            "claims": {
                "keys": "......"
            },
            "correlationId": "787ff406-9ae9-4f47-8349-1a976134e65e",
            "httpRequest": {
                "clientRequestId": "738102a5-8cb0-4b66-918f-18ab5e60305a",
                "clientIpAddress": "143.178.104.18",
                "method": "PUT",
                "url": "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/event-trigger?api-version=2014-04-01-preview"
            },
            "resourceProvider": "Microsoft.Resources",
            "resourceUri": "/subscriptions/{subscriptionId}/resourceGroups/event-trigger",
            "operationName": "Microsoft.Resources/subscriptions/resourceGroups/write",
            "status": "Succeeded",
            "subscriptionId": "{subscriptionId}",
            "tenantId": "{tenantId}"
        },
        "dataVersion": "",
        "metadataVersion": "1"
    }
]
```
