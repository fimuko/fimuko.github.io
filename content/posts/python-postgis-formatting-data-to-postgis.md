---
title: "Parsing geometries for Post GIS with Python"
date: 2023-06-15T13:57:48+02:00
# weight: 1
# aliases: ["/first"]
tags: ["python", "postgres", "postgis"]
categories: ["analytics", "data engineering"]
author: "me"
# author: ["Me", "You"] # multiple authors
summary: "Custom data pre-processing so that geometries in postgis can be used" # this shows up on the list
description: "Custom data pre-processing so that geometries in postgis can be used" # this shows up on the single page
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
UseHugoToc: false
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

In the scenario we are going to explore in this post, we have a GIS data in a proprietary format (not geojson), which comes from an API of a data provider. We want to parse this format and insert the data into a Post GIS database, or a Postgresql with `postgis` extension installed.

## Analysing input data

The API response is in `.json` format. After exploration of the data sample, we notice that the related GIS data fields take one of the following forms (examples below).

Example 1:
```json
"points": [
    [
        11.37975,
        11.3847
    ],
    [
        47.37138,
        33.8887
    ],
    
    ...
]
```

Example 2:
```json
"points": [
    {
        "id": 0,
        "lat": 11.76508,
        "lng": 11.39344
    },
    {
        "id": 1,
        "lat": 11.74897,
        "lng": 11.40683
    },

    ...
]
```

Example 3:
```json
"points": [
    [
        {
            "id": 0,
            "lat": 11.46365,
            "lng": 11.58313
        },
        {
            "id": 1,
            "lat": 11.39815,
            "lng": 11.35217
        },
        ...
    ],
    [
        ...
    ],
    
    ...
]
```

So when parsing these records with Python, we have the following different cases:
1. list of lists
2. list of dicts
3. list of lists of dicts
 
After having analysed the API response sample data carefully, and comparing it with the actual output of the web application / service that the API belongs to, we come to the conclusion that these different cases represent different [Post GIS geometries](https://postgis.net/workshops/postgis-intro/geometries.html). And so:

1. **list of lists** represents a `MULTIPOINT` geometry
2. **list of dicts** represents a `LINESTRING` geometry, and we can generalize it to `MULTILINESTRING`
3. **list of lists of dicts** represents `MULTIPOLYGON` geometry

The later insertion of data into Post GIS can be done using a convenient [`ST_GeomFromText`](https://postgis.net/docs/ST_GeomFromText.html) function. This function converts a text string in a specific format into a relevant geometry upon data insertion into PostGIS. But in order to succesfully execute the insert operation, we need to correctly construct those text strings. 

## Converting input data into desired format

Looking at the documentation, for each cases we need to construct strings in the following formats:

```py
#MULTIPOINT ( (lng lat),(lng lat),(lng lat) )

#MULTILINESTRING ( (lng lat,lng lat,lng lat),(lng lat,lng lat,lng lat) )

#MULTIPOLYGON ( ((lng lat,lng lat,lng lat)),((lng lat,lng lat,lng lat)) )
```
where `lng` and `lat` are longitude and latitude of each individual point in the input data. 

Looking at the examples of the text strings that we need to construct, we can see that different parts of geometries are seprated by brackets. We can also categorize brackets into different types as follows:
- **inner brackets** encapsulate each `lng lat` pair representing a point
- **middle brackets** encapsulate collections / groups of points
- **outer brackets** encapsulate the entire geometry definition that follows the word indicating the geometry (`POINT`, `MULTIPOINT` etc).


Looking at the geometries in our input data, we have the following:

```py
#MULTIPOINT ( (lng lat),(lng lat),(lng lat) )
#             ^inner bracket                ^outer bracket

#MULTILINESTRING ( (lng lat,lng lat,lng lat),(lng lat,lng lat,lng lat) )
#                  ^middle bracket           ^middle bracket           ^outer bracket

#MULTIPOLYGON ( ((lng lat,lng lat,lng lat)),((lng lat,lng lat,lng lat)) )
#               ^^middle brackets           ^^middle brackets           ^outer bracket
```

We want our parser / converter function to be generic, so we are going to parametrize numbers of brackets:

- `num_of_brackets_inner`
- `num_of_brackets_middle`
- `num_of_brackets_outer`

This is it as far as brackets are concerned. 

In addition (based on common sense and data insert tests that we carried out), for polygons - each polygon in a multipolygon needs to be "closed" i.e. it needs to be a closed loop, so the last point in a polygon needs to be the same as the first point in the polygon. After inspecting the input data samples, we realize that this is not the case, so this requirement would have to be met "synthetically" i.e. in the output data set we will need to ensure that there is a point that allows to close the polygon. So we need a paramter that will indicate whether we need a "closed" geometry or not:
- `closing_flag`

And lastly, we will also use a parameter that will indicate if we have a "deep list" or not i.e. if we have just list of objects, or the list of list of objects. This will determine if we need to iterate through one or two levels of the list. This parameter will be called:
- `deep_list_flag`

## workflow

In the conversion workflow, when we are parsing a record from a input .json data, we need to first detect what `points` record type we have. This can be realized by a relatively simple `if`:

```py
if isinstance(record["points"][0], list):
    postgis_geometry = "MULTIPOLYGON"
else:
    postgis_geometry = "MULTILINESTRING"
```
(Please note that the code above is just to illustrate the mechanism, and it is not complete as it does not cover the case of "MULTIPOINT" geometry from our scenario.)

Once the geometry is detected, we then pass this parameter along with the record to the next function in the workflow:

```py
value = await construct_geometry(postgis_geometry, record["points"])
```

This function is defined as follows, and it is a higher level function constructing the desired text string representing the geometry:
```py
async def construct_geometry(postgis_geometry: str, points: list):
   
    if postgis_geometry == "MULTIPOLYGON":
        #MULTIPOLYGON ( ((lng lat,lng lat,lng lat)),((lng lat,lng lat,lng lat)) )
        num_of_brackets_outer = 1
        num_of_brackets_middle = 2
        num_of_brackets_inner = 0

        # list of list of dicts
        deep_list_flag = True
        closing_flag = True
        
    elif postgis_geometry == "MULTILINESTRING":
        #MULTILINESTRING ( (lng lat,lng lat,lng lat),(lng lat,lng lat,lng lat) )
        num_of_brackets_outer = 1
        num_of_brackets_middle = 1
        num_of_brackets_inner = 0
                        
        # list of dicts
        deep_list_flag = False
        closing_flag = False
        
    elif postgis_geometry == "MULTIPOINT":     
        # MULTIPOINT ( (lng lat),(lng lat),(lng lat) )
        num_of_brackets_outer = 1
        num_of_brackets_middle = 0
        num_of_brackets_inner = 1

        # list of list of ints or strings
        deep_list_flag = False
        closing_flag = False

    else:
        logging.error(f"function does not yet have a case to handle this type of geometry: {postgis_geometry}")


    value = postgis_geometry+"("*num_of_brackets_outer

    if deep_list_flag:
        for i, points_in_deeper_list in enumerate(points):
            if (postgis_geometry == "MULTIPOLYGON") and (len(points_in_deeper_list) < 3):
                # in this case we are skipping this list of points because it is too short
                # to construct a polygon, which should have a minimum of three corners i.e. be a triangle
                continue
            else:
                value = value + await geom_points_to_text(num_of_brackets_middle, num_of_brackets_inner, points_in_deeper_list, closing_flag)
                # now we need comma before the next polygon (or more general: collection of points), but we need to avoid comma after the last polygon
                if i < (len(points)-1):
                    value = value + ","
    else:
        #this is a one level list
        value = value + await geom_points_to_text(num_of_brackets_middle, num_of_brackets_inner, points, closing_flag)

    value = value + ")"*num_of_brackets_outer

    return value
```

This higher level function above relies in turn on the following function to construct the middle part of the text string:

```py
async def geom_points_to_text(num_of_brackets_middle: int, num_of_brackets_inner:int, points: list, closing_flag: bool):
    # polygon_points should be list of dicts with "lng" and "lat" keys
    value = num_of_brackets_middle*"("

    for j, point_lat_lng in enumerate(points):

        if isinstance(point_lat_lng, dict):   
            value = value + "("*num_of_brackets_inner + str(point_lat_lng["lng"]) + " " +str(point_lat_lng["lat"]) + ")"*num_of_brackets_inner
        elif isinstance(point_lat_lng, list):
            value = value + "("*num_of_brackets_inner + str(point_lat_lng[1]) + " " +str(point_lat_lng[0]) + ")"*num_of_brackets_inner

        # now we need comma before the next point, but we need to avoid comma after the last point
        if j < (len(points)-1):
            value = value + ","
        else:
            # now we reached to the last point in the list, 
            # if we are constructing a polygon geometry
            # we need to ensure we close the polygon
            # so we have to add the first point in the list
            # this is actually assuming each point is in the format dict
            if closing_flag:
                value = value + "," + str(points[0]["lng"]) + " " +str(points[0]["lat"])

    value = value + num_of_brackets_middle*")"

    return value
```
Such conversion function in pretty generic and can be reused / expanded to handle more geometries, should we encounter data sets with similar formats.