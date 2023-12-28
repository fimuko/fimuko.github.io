---
title: "Hosting Azure Function Apps in Docker containers - a quick guide"
date: 2023-05-28T14:37:54+01:00
# weight: 1
# aliases: ["/first"]
tags: ["azure", "azure function", "python"]
categories: ["devops"]
author: "fimuko"
# author: ["Me", "You"] # multiple authors
summary: "an overview of options for using Docker to host logic of your Azure Function App" # this shows up on the list
description: "an overview of options for using Docker to host logic of your Azure Function App" # this shows up on the single page
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


Azure Function Apps offer a flexible way to execute app logic in the cloud, and with the integration of Docker containers, developers can gain even more control over their environment. This blog post delves into how to host your Azure Function Apps in Docker containers, focusing on the different hosting plans and steps involved.

## Understanding hosting plans

### Consumption Plan limitations

First, it's crucial to understand that not all Azure Function App plans support containerization. The Consumption Plan, which offers a pay-as-you-go model, does not currently support hosting Azure Functions in Docker containers. This is outlined in the Azure documentation on [functions scale](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale#operating-systemruntime).

### Premium Plan capabilities

However, the Premium Plan offers this capability. In the Premium Plan, you can deploy container images of functions, providing more flexibility and control over your runtime environment. For more details, see the section on [Docker container deployment](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-technologies#docker-container) in Azure Functions documentation.

## Hosting Azure Functions in containers

### Deploying to Container Apps

It's important to distinguish between deploying Function Apps in containers and using Azure Container Apps. Azure Container Apps is a service for hosting containerized applications, which includes the ability to deploy Azure Functions. For more information, refer to the guide on [hosting functions in Container Apps](https://learn.microsoft.com/en-us/azure/azure-functions/functions-container-apps-hosting).

### Docker Images for Azure Functions

Microsoft provides a range of Docker images for Azure Functions, suitable for various programming languages and scenarios. These images can serve as a starting point for customizing your containerized function app. Find the list of available Docker images [here](https://mcr.microsoft.com/en-us/product/azure-functions/python/about).

### Building and Deploying a Dockerized Function

To deploy an Azure Function in a Docker container, you need to build a Docker image of your function app. The Azure documentation provides a comprehensive [tutorial](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deploy-container-apps?tabs=acr%2Cbash&pivots=programming-language-python) on building and deploying a Dockerized function, which is an excellent resource for getting started.

## Custom Containers for Azure Functions

For advanced scenarios, you might want to create custom containers for your Azure Functions. Azure supports custom containers, allowing you to tailor the runtime environment to your specific needs. The process involves creating a Dockerfile for your function app and deploying it through Azure Container Registry (ACR) or Docker Hub. Detailed instructions can be found in the guide on [how to create custom containers](https://learn.microsoft.com/en-us/azure/azure-functions/functions-how-to-custom-container?tabs=acr%2Cazure-cli&pivots=azure-functions).

## Conclusion

Hosting Azure Function Apps in Docker containers opens up new possibilities for app development and deployment. While the Consumption Plan does not support containerization, the Premium Plan offers this flexibility. By leveraging Docker, you can ensure a consistent environment for your functions, tailor the runtime to your needs, and potentially improve the scalability and manageability of your applications.
