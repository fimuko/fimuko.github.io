---
title: "Azure data pipeline with Azure Function Apps - Introduction"
date: 2023-07-03T14:07:24+01:00
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

Welcome to my latest blog post where we explore the creation of a monorepo project template, perfectly tailored for a specific data acquisition and processing workload using Azure's robust cloud services. Our goal is to design a system that adheres to modern development standards, harnesses the power of serverless architecture, and efficiently handles data from the web to a database. 

Let's break down this process into manageable steps.

# The Project Overview
Our project template focuses on a two-part workflow:

1. **Data Acquisition**: Triggered on a schedule, this part involves making HTTP(S) calls to an API to fetch data and store it in Azure Blob Storage.
2. **Data Processing**: Upon the arrival of new data in the blob, another function is triggered to preprocess this data and then save it to an SQL database.

This approach ensures a seamless flow from data collection to processing, aligning with the minimalist yet efficient ethos of serverless architectures.

# Data Source

In our journey to build a modern, serverless data pipeline using Azure, we've chosen an intriguing and dynamic data source for our project: the real-time position of the International Space Station (ISS). We will ntegrate data from the [International Space Station API](https://wheretheiss.at/) into our Azure-based data pipeline. 

## The API: A Gateway to the ISS's Journey
The API we're focusing on provides current positional data of the ISS. It's a fascinating dataset that offers a unique glimpse into the path of one of humanity's most significant space endeavors. By tapping into this API, our data acquisition function will retrieve the ISS's location in real-time. 

**API Details**:
- **Endpoint**: `https://api.wheretheiss.at/v1/satellites/25544`
- **Documentation**: https://wheretheiss.at/w/developer
- **Data Offered**: Real-time latitude, longitude, altitude, and velocity of the ISS.
- **Usage**: With simple HTTP(S) requests, we can fetch this data, which our Azure function will process and store.

# Workflow Part 1: Scheduled Data Acquisition
## Setting the Stage with Cron Scheduling
We begin with a time-triggered function. This is akin to a cron job where the function activates at predetermined intervals. This trigger initiates the first part of our workflow. 

Given the remarkable speed of the International Space Station (ISS), orbiting Earth every 90 minutes, there is a wealth of data available for capturing its position. However, to balance the richness of this data with the practical considerations of serverless architecture costs, we have strategically decided to execute our Azure function once every minute. This frequency will provide a detailed enough snapshot of the ISS's position while maintaining a manageable level of resource utilization. This approach ensures we effectively capture the ISS's rapid journey around Earth, without incurring unnecessary costs from overly frequent data acquisition in our serverless setup.

## The Event Handling: Fetch and Store
Upon triggering, the function calls a specified API to acquire data. The retrieved data, in a raw `.json` format, is then written to Azure Blob Storage. This storage acts as our raw data repository, ready for further processing.

# Workflow Part 2: Data Preprocessing and Storage

## Triggered by New Data
As soon as new data lands in the blob, it triggers the second part of our workflow. This event-driven approach ensures near real-time processing of incoming data.

## Preprocessing and Database Entry
The function activated by this new blob event takes on the task of preprocessing. This could involve data cleaning, transformation, enrichment, or any other necessary preparation for database storage. Once processed, the data is then neatly stored in an SQL database, completing our data pipeline. The data is now ready for consumption from the SQL.

# The Serverless Advantage
In line with modern development standards, we chose a serverless framework for this project, specifically leveraging Azure's capabilities. Serverless computing, particularly Azure's consumption-based model, offers several benefits:

- **Cost-Efficiency**: You only pay for the compute time you use, making it an economical choice for workloads that don't require constant server uptime.
- **Scalability**: Azure automatically scales the resources to match the workload demands, ensuring efficient handling of varying data volumes.
- **Simplified Management**: With serverless, the hassle of server maintenance, patching, and administration is handled by Azure, allowing you to focus on the business logic and workflow design.

# Conclusion
By utilizing Azure's serverless offerings, we are aiming to outline a streamlined and effective template for a data pipeline that goes from web data acquisition to database storage. This approach not only aligns with modern development practices but also ensures efficiency, scalability, and cost-effectiveness. Stay tuned for future posts where we will delve deeper into each component of this workflow, offering more insights and detailed implementation guides. Happy coding!