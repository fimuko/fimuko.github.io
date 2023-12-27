---
title: "Azure data pipeline with Azure Function Apps - Choice of tech stack"
date: 2023-07-05T14:41:09+01:00
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

# Choosing the Right Tech Stack for Our Azure-Based Data Pipeline

Welcome to the second installment of our blog series where we delve into the technical architecture of our Azure-based data pipeline project. In this post, we'll explore the optimal tech stack choices for our needs, considering the specific requirements and constraints of our application.

# Azure's Offering for Hosting Application Logic: Assessing the Options

Azure presents a range of services that can be harnessed for various application needs. For our project, which involves sporadic execution of functions and Python-based data transformation, we have several considerations:

1. **Azure Logic Apps**: While powerful for workflow automation, Logic Apps are less suited for our scenario. They lack the native capability to run complex Python code, a crucial requirement for our data transformations.

2. **Azure Function Apps and Azure Durable Functions**: These offer more flexibility and are ideal for serverless computing scenarios like ours. Durable Functions, an extension of Azure Functions, enable stateful functions in a serverless environment. However, the question arises: Do we need the advanced orchestration and state management features of Durable Functions, or can we achieve our objectives with standard Function Apps?

3. **App Service and Azure Container Apps**: While these services offer extensive capabilities, especially for Kubernetes-based applications, they are an overkill for our requirement, which involves limited and sporadic function execution.

4. **Virtual Machines (VMs)**: VMs provide complete control and flexibility but would incur continuous costs, even when idle. This doesn't align with our need for sporadic execution and cost efficiency.

## Our Choice: Azure Function Apps

After evaluating the options, Azure Function Apps emerge as the most fitting choice for our project, for several reasons:

1. **Python Support**: Azure Function Apps support Python, allowing us to handle complex data transformations seamlessly.

2. **Cost-Effective**: Given their serverless nature, Azure Function Apps are cost-effective for our use case of sporadic executions. We only pay for the compute resources we use when the functions are running.

3. **Local Development**: Azure Function Apps can be developed and tested locally without incurring any costs. This aligns with our need for a development environment that allows for testing application logic without additional expenses.

4. **Simplicity and Scalability**: With Azure Function Apps, we can create two separate functions within a single Function App to handle our two-step workflow (data acquisition and processing). This setup could simplifie deployment and management while still providing the scalability benefits of a serverless architecture. However, upon further consideration, we have decided to utilize two separate Azure Function Apps, each dedicated to one step of our two-part workflow. 

### Durable Functions: Do We Need Them?

Durable Functions are ideal for complex, stateful workflows. However, for our project, which involves a straightforward two-step process, standard Azure Function Apps should suffice. Durable Functions would be more relevant if our workflow involved complicated state management or extensive orchestration of multiple functions, which is not the case here.

### Our selected set-up: Two Separate Azure Function Apps
We have decided to utilize two separate Azure Function Apps, each dedicated to one step of our two-part workflow. 
1. **First Azure Function App**: This will be responsible for the data acquisition phase, where it will periodically call the API to fetch the ISS's position and store this data in Azure Blob Storage.

2. **Second Azure Function App**: This will handle the data processing phase. Triggered by the arrival of new data in the blob storage, it will preprocess the data and store it in the Azure Database for PostgreSQL.

### Advantages of Two Separate Apps Over a Single App with Multiple Functions
- **Isolation of Responsibilities**: Each app focuses on a specific task (data acquisition or processing), which simplifies debugging and maintenance since the functions are more contained and less interdependent.
- **Scalability** : Each app can be scaled independently based on its specific workload and resource consumption. This is particularly advantageous when the data acquisition and processing demands vary significantly.
- **Enhanced Reliability**: In case one app encounters an issue, it won't affect the other app's operation, thus ensuring better overall reliability of the system.
- **Flexibility in Development and Deployment**: Developing and deploying each part of the workflow separately allows for more focused updates and testing, reducing the risk of changes in one function impacting another.
This setup with two separate Azure Function Apps provides a more segmented and potentially more robust architecture for our workflow, enhancing maintainability, scalability, and reliability.


# Selecting the Optimal Database Solution

A critical component of our Azure-based data pipeline is the choice of database for storing and managing the processed data. In our project, the choice of database plays a pivotal role in handling and storing the geographic information system (GIS) data effectively.

Azure offers a variety of other database options, each with its own unique capabilities. Here's an overview of some other notable database services available in Azure:

- **Azure SQL Database**: A general-purpose relational database service based on the latest stable version of Microsoft SQL Server Database Engine. It's highly scalable, supports a wide range of applications, and offers built-in intelligence and security.

- **Azure Cosmos DB**: A globally distributed, multi-model database service that supports schema-less data, which makes it a great option for applications that need to handle large volumes of diverse data types, like JSON, graph, and key-value.

- **Azure Database for MySQL**: A fully managed, scalable MySQL database service for app developers. It provides strong performance, security, and compatibility and is ideal for applications that rely on MySQL.

- **Azure Table Storage**: A NoSQL data store for semi-structured data. It's a good choice for flexible datasets like user data for web applications, address books, device information, and other metadata.

- **Azure Blob Storage**: Primarily used for storing massive amounts of unstructured data, such as text or binary data, which is ideal for serving images or documents directly to a browser, storing files for distributed access, and streaming video and audio.

- **Azure Cache for Redis**: A fully managed implementation of Redis, offering high-performance data caching to accelerate the performance of applications by providing a highly accessible data store in the cloud.

- **Azure MariaDB**: A fully managed MariaDB database service, which is a community-developed fork of MySQL. It’s a great choice for open-source applications that require a fully compatible SQL relational database.


## Our Choice: Azure Database for PostgreSQL

After careful consideration, we have selected Azure Database for PostgreSQL for this purpose. This particular database service stands out due to its PostGIS extension, a powerful addition that significantly enhances its capabilities in managing and querying GIS data. PostGIS is widely recognized and utilized in various applications for its robust spatial and geographic data processing functionalities. Moreover, Azure Database for PostgreSQL aligns seamlessly with our serverless architecture. It offers a serverless option, providing auto-scaling capabilities and on-demand pricing, which makes it not only efficient for handling fluctuating workloads but also cost-effective. This combination of advanced GIS capabilities with a serverless model makes Azure Database for PostgreSQL an ideal choice for our project, ensuring that we can store and manipulate the ISS location data with precision and efficiency.

# Conclusion

In our quest to construct an efficient and scalable data pipeline, Azure Function Apps and Azure Database for PostgreSQL emerge as the most suitable choices. Azure Function Apps provide the flexibility and cost-effectiveness needed for our serverless architecture, supporting Python for complex data transformations and allowing local development without incurring costs. Complementing this, Azure Database for PostgreSQL, with its PostGIS extension, stands out as the ideal database solution. It brings robust GIS data processing capabilities and aligns with our serverless approach through its scalable and cost-effective serverless offering. This combination of Azure Function Apps and Azure Database for PostgreSQL embodies the perfect blend of functionality, efficiency, and scalability, tailor-made for our project's requirements. As we move forward, we're excited to harness the full potential of these Azure services to bring our data pipeline project to fruition. Stay tuned for more updates and insights as we navigate through this exciting journey with Azure.
