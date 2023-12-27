---
title: "Azure data pipeline with Azure Function Apps - Structure of the git repository"
date: 2023-07-10T14:41:09+01:00
# weight: 1
# aliases: ["/first"]
tags: ["azure", "azure function", "python", "GIS", "data pipeline", "postgres", "postgis"]
categories: ["data engineering"]
author: "fimuko"
# author: ["Me", "You"] # multiple authors
summary: "serverless with Azure Function Apps and PostGIS / data from RESTful API" # this shows up on the list
description: "serverless with Azure Function Apps and PostGIS / data from RESTful API" # this shows up on the single page
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

Welcome to the third installment of our series on building data pipelines with Azure. In this post, we'll dive into the structure of the git repository for our Azure data pipeline, explore reference architectures, and introduce Azure Developer CLI and `azd`. 

## Repository structure: the debate

In the realm of software development, particularly for data pipeline projects, there are primarily two approaches to structuring your project repositories: monorepo and multi-repo. 

A **monorepo approach** involves keeping all your code - encompassing various services, applications, and infrastructure code - in a single, unified repository. This approach facilitates easier management of dependencies, streamlined workflows, and consistency in tooling and standards across different parts of the project. 

On the other hand, the **multi-repo** approach advocates for separating each service, application, or component into its own repository. This structure allows for greater isolation, reducing the scope of changes and potentially simplifying the authorization and security model for each component. Both approaches have their own sets of advantages and trade-offs, and the choice largely depends on the specific needs of the project, team size, and the complexity of the services being developed.

## Embracing the monorepo approach for our data pipeline

Given the scale and scope of our data pipeline project, we have determined that a monorepo approach is more suitable for our needs. This decision is particularly advantageous considering the relatively small size and limited complexity of our pipeline. A monorepo offers a more streamlined and cohesive development experience, which is ideal for smaller projects like ours. It simplifies dependency management and reduces the overhead associated with maintaining multiple repositories. Moreover, having all components of our data pipeline - from infrastructure as code to application logic - in a single repository allows for easier coordination and faster iterations, enhancing overall efficiency and agility in the development process.

## Structure of the project
A typical monorepo project will consist of:
- **Infrastructure Code**: often using `terraform`, or Azure Bicep, an Azure-specific Infrastructure as Code (IaC) language, to define and deploy Azure resources.
- **Function App Code**: the application / logic code for our Azure Function Apps handling data acquisition and processing (in our case this will be Python code).
- **Configuration and Settings**: Centralized configuration for easy management across the pipeline (usually `yaml` files).

But before we jump to explaining in more details the structure of the monorepo project, we would like to introduce the Azure Developer CLI.

## Introducing Azure Developer CLI (`azd`)

Azure Developer CLI (`azd`) is a tool that simplifies the development and deployment of applications on Azure. It streamlines setting up your entire development environment, from infrastructure to application code.

The typical project template provided by the Azure Developer CLI inherently adopts a monorepo structure, which aligns perfectly with the needs of our data pipeline project. This alignment is particularly beneficial as it supports our decision to use a monorepo for its simplicity and efficiency in managing smaller, less complex projects. The Azure Developer CLI facilitates this approach by seamlessly integrating various components of an Azure project – such as infrastructure, application code, and configurations – into a single, cohesive repository. By utilizing both the monorepo structure and the Azure Developer CLI, we can leverage the advantages of a unified development environment while also enjoying the streamlined, automated tooling and deployment capabilities that Azure Developer CLI offers. This combination enhances our project's development workflow, making it more efficient and well-suited for our specific project requirements.

> For more detailed information about Azure Developer CLI and `azd`, check out the official documentation:
> - [Getting Started with Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/get-started?tabs=localinstall&pivots=programming-language-python)
> - [Azure Developer CLI Overview](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/overview)
> - [Azure Developer CLI Templates](https://azure.github.io/awesome-azd/)

### Benefits of `azd` in a monorepo
- **Project Scaffolding**: Quickly generate boilerplate code for your Azure applications.
- **Local Development Support**: Test your application locally before deploying to Azure.
- **Integration with Azure Services**: Seamlessly connect with Azure databases, storage, and more.

## Typical `azd` project structure

Typically the `azd` project has the following structure:

```txt
├── .azdo                      [ For Azure DevOps ]
│   └── azure-dev.yaml         [ Azure Pipelines workflow to deploy to Azure using azd ]
├── .azure                     [ For storing Azure configurations]
│   └── <your environment>     [ For storing all environment-related configurations]
│      ├── .env                [ Contains environment variables ]
│      └── config.json         [ Contains environment configuration ]
├── .devcontainer              [ ?For DevContainer ]
│   ├── devcontainer.json      [ ?For setting up the containerized development environment ]
│   └── Dockerfile             [ ?For building the container image ]
├── .github                    [ ?For GitHub workflow ]
│   └── azure-dev.yaml         [ ?GitHub Actions workflow to deploy to Azure using azd ]
├── infra                      [ For creating and configuring Azure resources ]
│   ├── abbreviations.json     [ Recommended abbreviations for Azure resources ]
│   ├── main.bicep             [ Main infrastructure file ]
│   ├── main.parameters.json   [ Parameters file ]
│   ├── app                    [ Contains Bicep modules]
│   └── core                   [ *Contains Bicep modules copied from azd reference library ]
├── src                        [ This is were applications / functions / services logic goes]
│   ├── <your service 1>       [ ]
│   │  ├── function_app.py     [ ]
│   │  ├── requirements.txt    [ ]
│   │  └── host.json           [ ]
│   ├── ...                    [ ]
│   └── <your service N>       [ ]
│      ├── function_app.py     [ ]
│      └── host.json           [ ]
└── azure.yaml                 [ !Describes the app and type of Azure resources]
```

So the project components outlined in the bullet points from the previous section would be reflected in the `azd` project:

- **Infrastructure as Code (IaC)**: Bicep or ARM templates for defining Azure resources. This will be sitting in the `infra` folder of the `azd` project template.
- **Application Code**: Your actual application code, in our case, the Python code for Azure Function Apps. This will be sitting in the `src` folder. 
- **Configuration Files**: Files for local and Azure environment settings, with the most important one in our project being `azure.yaml`.

The `azd` structure aligns perfectly with our monorepo approach, enabling efficient development and management of our entire Azure data pipeline.

## Conclusion

By choosing a monorepo structure and leveraging Azure Developer CLI (`azd`), we've set a solid foundation for our Azure data pipeline project. This setup not only streamlines our development workflow but also aligns with best practices in Azure application development. In our next posts, we'll delve deeper into setting up our monorepo and utilizing `azd` for efficient development. Stay tuned!

