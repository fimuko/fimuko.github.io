---
title: "Getting Started with Azure Function Apps in Python Using VSCode"
date: 2023-05-02T09:15:35+01:00
# weight: 1
# aliases: ["/first"]
tags: ["azure", "azure function", "python"]
categories: ["devops"]
author: "fimuko"
# author: ["Me", "You"] # multiple authors
summary: "A useful guide explaining how to kickstart local dev with Azure Function Apps." # this shows up on the list
description: "A useful guide explaining how to kickstart local dev with Azure Function Apps." # this shows up on the single page
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

Azure Function Apps offer a serverless computing environment, allowing developers to run event-driven applications without worrying about infrastructure management. This blog post explores how to get started with Azure Function Apps, focusing on Python development within the VS Code environment.

## Python programming models in Azure Function Apps
Before we kick off, we need to explain that Azure Function Apps in Python offer two programming models: V1 and V2.

[The V1 model](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python?tabs=asgi%2Capplication-level&pivots=python-mode-configuration#programming-model), an initial version, uses a `functions.json` file to define functions, whereas the newer V2 model adopts a decorator-based approach, leading to a simpler and more code-centric file structure. 

[The V2 model](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python?tabs=asgi%2Capplication-level&pivots=python-mode-decorators#programming-model) aligns with well-known Python frameworks by using decorators for triggers and bindings, thereby eliminating the need for a separate configuration file. 

This blog post will focus on the Python __V2 programming model__, leveraging its advanced features for a more efficient and streamlined development process in Azure Function Apps.

## Setting up your local development environment

> Note: we are assuming Python and VSCode are already installed on your machine.

When setting up a local development environment for Azure Function Apps in Visual Studio Code (VSCode) on either Windows or Ubuntu, the first step is to add the **Azure Functions extension**. This extension simplifies development by providing integrated tools and templates specifically for Azure Functions. You can install it from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) or directly within VSCode.

After installing the Azure Functions extension, the next crucial step is to install **Azure Functions Core Tools**. The installation process differs slightly between Windows and Ubuntu. For detailed instructions tailored to your operating system, please refer to the official Azure documentation on running Azure Functions locally: [Azure Functions Core Tools installation guide](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-python). This guide provides the necessary commands and steps to ensure a successful setup.

We also need to install **Azurite**. Azurite is an open-source Azure Storage emulator that offers a local environment for testing your data storage applications using Azure Storage APIs. It's particularly useful for Azure Functions development as it emulates Azure Blob, Queue, and Table storage services, allowing developers to test their functions locally without connecting to actual Azure services. Please refer to Microsoft documentation on how to install it: [Azurite installation guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite?tabs=visual-studio-code%2Cblob-storage#install-azurite).


## Initiating a new project 

The initial step in both Windows and Ubuntu is to create a project folder and to activate a Python virtual environment for your project. This environment is essential for managing dependencies and maintaining a consistent development setting. 

In your project's directory within VSCode, use the command: 
```sh
python -m venv .venv
``` 
to create the virtual environment. To activate it on Windows, run 
```ps
.venv\Scripts\activate
```
, and on Ubuntu, use:
```sh
source .venv/bin/activate
```

Now you can initiate your Azure Function App project with the following command:
```sh
func init --python -m V2
```

It is also good to run:
```sh
python -m pip install --upgrade pip
```


## Project folder structure explained

The `func init` command created the following structure within the project folder:

```txt
├── .venv     
│      ├── Lib\site-packages
│      ├── Scripts
│      └── pyvenv.cfg
├── .vscode
├── .funcignore
├── .gitignore
├── function_app.py
├── host.json
├── local.settings.json
└── requirements.txt
```

Now, let's explain what they all are for (**not** in the order of appearance):

### `requirements.txt` file

As for any other Python project, the `requirements.txt` file in an Azure Function App project lists all the Python packages and their specific versions required for the project. This file is used to ensure that all the necessary dependencies are installed, maintaining consistency across different development environments and during deployment. When deploying to Azure, the packages listed in `requirements.txt` are automatically installed to the Azure environment, mirroring the local development setup. This file is crucial for managing dependencies efficiently and ensuring that your function app has all the necessary resources to operate both locally and in Azure.


### `.venv` folder

This is where you virtual environment lives, with your py packages installed to. A `pyvenv.cfg` file is created in the environment directory specifying information about the environment such as the path to Python which was used for creation, its version, and whether packages installed in system Python are copied.

To learn more about `venv` please consult [this excellent resource](https://realpython.com/python-virtual-environments-a-primer/#how-can-you-work-with-a-python-virtual-environment).


### `.vscode` folder

This is a directory used by Visual Studio Code (VSCode) to store project-specific settings and configurations. This folder typically contains a `settings.json` file, which holds various configuration settings tailored for your development environment, such as Python interpreter paths, linter configurations, and formatting preferences. It can also include other files like `launch.json` for debugging configurations and `tasks.json` for defining custom build and deployment tasks. The `.vscode` folder is essential for maintaining a consistent and tailored development environment across different machines and team members.

You can find more information about this folder on the following page - [VSCode project structure guide](https://github.com/microsoft/vscode-azurefunctions/wiki/Project-Structure).


### `.gitignore` file
The `.gitignore` file in a software development project is a crucial file used with Git version control. It tells Git which files or folders to ignore in a project. Typically, it includes temporary files, local configuration files, build outputs, and other files that should not be committed to the repository. In the context of an Azure Function App, `.gitignore` might exclude files such as `.env`, `local.settings.json`, `.vscode/`, and the `.venv/` directory, ensuring that sensitive, non-shareable, or machine-specific configurations aren't included in the version control system.

### `.funcignore` file
The `.funcignore` file in an Azure Function App project functions similarly to a `.gitignore` file. It specifies which files and directories should be ignored when the project is deployed to Azure. This includes files that are not necessary for the function to run or files that contain sensitive information, such as local configuration files, development tools, or personal scripts. By using `.funcignore`, you can control the size and content of your deployment package, ensuring that only essential files are uploaded to Azure. This optimizes the deployment process and maintains the security of your application.


### `local.settings.json` file

This is a configuration file used to store app settings and connection strings for local development. This file is critical for testing your functions locally as it replicates the environment settings found in Azure, such as storage connection strings and application settings. Importantly, `local.settings.json` is not deployed to Azure; it is intended only for the local environment to mimic the Azure settings, ensuring that your function app behaves similarly both locally and when deployed. So make sure you add it to `.funcignore`.

For our purpose of developing with Python, use the following content of the `local.settings.json` file:

```json
{
    "IsEncrypted": false,
    "Values": {
      "FUNCTIONS_WORKER_RUNTIME": "python",
      "AzureWebJobsStorage": "UseDevelopmentStorage=true",
      "AzureWebJobsFeatureFlags": "EnableWorkerIndexing",
      "BLOB_CONNECTION_STRING":"DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;"
    }
  }
```

Please note the `BLOB_CONNECTION_STRING` value. It is a so-called "well known" / standard connection string value enabling local Azure Function App connection to the Azurite storage emulator. We need this setting because in the next paragraph we will want to define a function that monitors files in a blob storage container, and reacts when a new blob is being created.


### `function_app.py` file

This is a file that contains your application logic i.e. your Python code. Let's write a basic Function App that reacts on a new blob within a Storage Container. We will use a blob trigger for that, and we will be relying on the following [code example](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=python-v2%2Cisolated-process%2Cnodejs-v4&pivots=programming-language-python#example).

Our code looks like this:

```py
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

Please note the following two settings in the `@app.blob_trigger` decorator:

```py
path="mycontainer",
connection="BLOB_CONNECTION_STRING"
```
The `connection` setting names an application setting that contains the connection string in your execution environment. Since our execution environment is local, the application settings are being defined within the `local.settings.json` file, so the `BLOB_CONNECTION_STRING` setting is taken from there.

The `path` setting in our case determins the name of the storage container that we want our function to monitor i.e. get notifications of events from. So make sure that within Azurite storage emulator, you create a storage container with the name of the value we provided in our sample code i.e. `mycontainer`.

## Simlating function locally

You need to now start Azurite Queue Service and Azurite Blob Service of your Azurite storage emulator. Those services are required to simulate Azure Function App locally, so make sure you start them before you execute the next command. You can start those services at the bottom of your VSCode window, on the bottom bar, where you can see 
```
[Azurite Table Service]  [Azurite Queue Service]  [Azurite Blob Service]
```

Once all the above is done you can start your local function simulation with:
```sh
func start
```

You should see the function start in the integrated terminal.
This concludes our kick-start guide to developing with Python for Azure Function Apps.

## References

Please find the following resources useful when developing in Python in Azure Function Apps:

- [Quickstart: Create a function in Azure with Python using Visual Studio Code](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-python?pivots=python-mode-decorators)
- [Azure Functions developer guide](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference?tabs=blob&pivots=programming-language-python)
- [Azure Functions Python dev guide](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python?tabs=asgi%2Capplication-level&pivots=python-mode-decorators)
- [Azure Functions overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview?pivots=programming-language-python)
- [Develop Azure Functions locally using Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-python)

## Some other / advanced topics
Listing here just a few notable examples useful to take into consideration further down the line when working with Azure Function Apps.

### Remote build on Azure
Deploying directly to Azure allows for remote building, which is explained in [this guide](https://github.com/microsoft/vscode-azurefunctions/wiki/Remote-Build). It's useful for complex apps that require cloud resources.

### Enabling diagnostics
Monitor and troubleshoot your Function Apps by enabling diagnostics. [The Azure diagnostics guide](https://learn.microsoft.com/en-us/azure/azure-functions/functions-diagnostics#start-azure-functions-diagnostics) provides detailed instructions.

### Managing dependencies
For advanced scenarios like deploying machine learning models, managing dependencies is key. See the [dependency management guide](https://learn.microsoft.com/en-us/azure/azure-functions/bring-dependency-to) for more details.

### Mounting file shares

Mounting file share for functions to use is useful when e.g. we have a pre-trained ML model that the function needs to use (see [here](https://learn.microsoft.com/en-us/azure/azure-functions/storage-considerations?tabs=azure-cli#mount-file-shares)).