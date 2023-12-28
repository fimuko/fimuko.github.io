---
title: "Azure Function Apps in Python - guide to triggers"
date: 2023-04-10T12:37:18+01:00
# weight: 1
# aliases: ["/first"]
tags: ["azure", "azure function", "python"]
categories: ["devops"]
author: "fimuko"
# author: ["Me", "You"] # multiple authors
summary: "explanation of how to use different Azure Function App triggers" # this shows up on the list
description: "explanation of how to use different Azure Function App triggers" # this shows up on the single page
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false # keep this false as if true it is not showing H1
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---


# Triggers in Azure Function Apps with Python V2

Azure Function Apps offer a variety of triggers that respond to different events, allowing for efficient and responsive cloud-based applications. In this blog post, we will explore the different types of triggers available in Azure Function Apps, specifically for Python V2 programming model. We'll cover HTTP trigger, schedule trigger, blob trigger, and event grid trigger, providing insights and code examples.

## 1. HTTP trigger

The HTTP trigger is perhaps the most straightforward and easy to understand for beginners. It allows your function to be invoked by an HTTP request.

### Code example:
```python
import azure.functions as func
import logging
import datetime

app = func.FunctionApp()


@app.function_name(name="HttpTriggerThis")
@app.route(route="dothis")

def test_function(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger do this')

    return func.HttpResponse(
            "This HTTP triggered do this function and executed it successfully.",
            status_code=200
    )


@app.function_name(name="HttpTriggerThat")
@app.route(route="dothat")
def test_function(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger do that')

    return func.HttpResponse(
            "This HTTP triggered do that function and executed it successfully.",
            status_code=200
    )
```

Please note that each function can only have a sigle trigger.

However, as you can see from the above example, we can define multiple functions within a single Function App, and for those we can define differnet endpoint routes (suffixes). In our example above we defined two of those: `dothis` and `dothat`. 

We can also pass http parameters through this trigger. See the following code example:

```py
import azure.functions as func
import logging

app = func.FunctionApp()

@app.function_name(name="HttpTrigger1")
@app.route(route="hello")
def test_function(req: func.HttpRequest) -> func.HttpResponse:
     logging.info('Python HTTP trigger function processed a request.')

     name = req.params.get('name')
     if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

     if name:
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
     else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )
```

In this example we are retrieving the `name` parameter from the http call that triggered the function.

> *Note:* Execution of functions using HTTP triggers is limited to a maximum of 230 seconds ([Azure Functions Timeout Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=v4&pivots=programming-language-python#timeout)).

## 2. Schedule trigger

Schedule triggers are used to run a function at a specified time, using a CRON expression which has 6 components.


### Code example:
```python
import azure.functions as func
import logging
import datetime

app = func.FunctionApp()

@app.function_name(name="ScheduleTrigger")
@app.schedule(schedule="*/10 * * * * *", arg_name="mytimer", run_on_startup=False,
              use_monitor=True) 
def test_function(mytimer: func.TimerRequest) -> None:
    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if mytimer.past_due:
        logging.info('The timer is past due!')

    logging.info('Python timer trigger function ran at %s', utc_timestamp)
```


### Reference:
- Detailed guide on schedule triggers ([Azure Schedule Trigger Guide](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=python-v2%2Cisolated-process%2Cnodejs-v4&pivots=programming-language-python)).

## 3. Blob trigger

Blob triggers react to events in Azure Blob Storage, such as the creation of a new blob.

### Code example:
```python
import logging
import azure.functions as func

app = func.FunctionApp()

@app.function_name(name="BlobTrigger1")
@app.blob_trigger(arg_name="myblob", 
                  path="mycontainer",
                  connection="BLOB_CONNECTION_STRING")
def test_function(myblob: func.InputStream):
   logging.info(f"Python blob trigger function processed blob \n"
                f"Name: {myblob.name}\n"
                f"Blob Size: {myblob.length} bytes")
```

### Reference:
- Blob Trigger Overview ([Azure Blob Trigger Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger)).
- Blob Storage Trigger Tutorial ([Azure Blob Storage Trigger Tutorial](https://learn.microsoft.com/en-us/azure/applied-ai-services/form-recognizer/tutorial-azure-function?view=form-recog-3.0.0)).
- Creating a Blob-Triggered Function in Azure Portal ([Azure Blob Triggered Function Tutorial](https://learn.microsoft.com/en-us/azure/azure-functions/functions-create-storage-blob-triggered-function)).

> *Note:* 
>- For blob event triggers, the metadata available is limited to `name, length, uri` (as per the [Azure Functions InputStream Documentation](https://learn.microsoft.com/en-us/python/api/azure-functions/azure.functions.inputstream?view=azure-python)).
>- Blob trigger is **not** the only trigger type usable with blob container events.

## 4. Event Grid Trigger

Event Grid triggers respond to events sent to Azure Event Grid. This can be an efficient way to handle events at scale.

Using event grid requires an Event Grid topic, with your Function App Function being subscribed to that topic. If your function needs to handle events from specific container, you need to set a filter on the subscription (all this can be done in Azure Portal, for more details please consult [this resource](https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-python&tabs=isolated-process%2Cnodejs-v4)):

```txt
Subject filters: Enable subject filtering
Subject Begins With: /blobServices/default/containers/<your_container_name>/blobs/
```

### Code example:
```python
import logging
import azure.functions as func
import json

app = func.FunctionApp()

@app.event_grid_trigger(arg_name="azeventgrid")
def EventGridTrigger(azeventgrid: func.EventGridEvent):
    
    result = json.dumps({
        'id': azeventgrid.id,
        'data': azeventgrid.get_json(),
        'topic': azeventgrid.topic,
        'subject': azeventgrid.subject,
        'event_type': azeventgrid.event_type,
    })

    logging.info('Python EventGrid trigger processed an event: %s', result)
```

### Reference:
- Understanding Event Grid ([Event Grid Video Explanation](https://www.youtube.com/watch?v=TujzkSxJzIA)).
- [Tutorial: Trigger Azure Functions on blob containers using an event subscription](https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-python&tabs=isolated-process%2Cnodejs-v4)



## The trigger selection dilemma

Please note that for the same type of event, such as a file upload to a storage container, you can choose from several trigger types depending on your specific requirements. Handling blob storage events efficiently is pivotal for many cloud-based applications. Azure provides several trigger options, each designed to respond to different types of blob storage events, enabling developers to tailor their applications to specific needs.

The primary trigger for blob storage events in Azure Functions is the **Blob Trigger**. This trigger is specifically designed to react when a new blob is added to a container, or an existing blob is updated. It's particularly useful for scenarios like processing data as soon as it arrives in the storage. The Blob Trigger ensures that your function is invoked as soon as the specified event occurs in the blob storage. However, it's important to note that the Blob Trigger may have a slight delay in processing large volumes of blobs due to its polling mechanism.

Another important trigger is the **Event Grid Trigger**, which integrates Azure Functions with Azure Event Grid. This trigger is more event-driven compared to the Blob Trigger and offers near-real-time processing capabilities. It's ideal for scenarios where you need to react promptly to changes in your Azure resources, including blob storage events. Unlike the Blob Trigger, the Event Grid Trigger is pushed-based, meaning it receives events as they happen, which can lead to more responsive and efficient processing.

For a more general-purpose trigger that can handle HTTP requests, the **HTTP Trigger** can be used. Though not directly related to blob storage events, this trigger can be configured to respond to HTTP requests that signify a change or an update in blob storage, often acting as a webhook.

Additionally, the **Queue Trigger** can indirectly respond to blob storage events. In this scenario, a separate process would monitor the blob storage and place messages into a queue whenever a specific event occurs. The Queue Trigger would then respond to these messages, allowing for a decoupled architecture that can be more scalable and manageable.

Each of these triggers has its use cases, strengths, and considerations. The choice of trigger depends on factors such as the required responsiveness to events, the volume of data to be processed, and the overall architecture of the application.

For in-depth understanding and guidance, Microsoft offers extensive documentation and resources:
- [Storage considerations for Azure Functions - Working with blobs](https://learn.microsoft.com/en-us/azure/azure-functions/storage-considerations?tabs=azure-cli#working-with-blobs)
- [Reacting to Blob storage events](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview)


## Advanced notes: trigger syncing

When you change any of your triggers, the Functions infrastructure must be aware of the changes. Synchronization happens automatically for many deployment technologies. However, in some cases, you must manually sync your triggers. 
Please refer to [this resource](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-technologies#trigger-syncing) for more details.