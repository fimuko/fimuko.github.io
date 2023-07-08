---
title: "Python Postgresql Bulk Insert With Error Handling"
date: 2023-06-02T16:19:06+02:00
# weight: 1
# aliases: ["/first"]
tags: ["postgres", "python"]
categories: ["data engineering"]
author: "me"
# author: ["Me", "You"] # multiple authors
summary: "Inserting data with Python into Postgres - efficiently and with ensuring errors handling" # this shows up on the list
description: "Inserting data with Python into Postgres - efficiently and with ensuring errors handling" # this shows up on the single page
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

When we are working with SQL data bases and inserting data, typically we want to be able to do it efficiently but also we want to ensure error-free execution. In this post we are going to explore how we can do it. We are going to use a python sql client to interact with the DB. 

Please note that you are going to see some `async/await` in the code. This is because this code was developed with the intention to deploy it to Azure Function, which behaves quite asynchronously, and it is therefore a good practice to write your code in an asynchronous manner.

## Background / scenario

The assumed scenario in which the function operates is that the function is triggered by a blob event, when a new blob is created. This blob is of specific format (a specific .json schema). The content of the blob needs to be pared, and all the records in the blob need to extracted and written to a Postgresql database. This scenario assumes a blob will typically contain a few hundred records, so what we want to try is to insert them to the DB all at once. But we only want to attempt to insert records which we know are not yet in the database (so we don't want to "upsert" records). And if the entire bulk insert attempt fails for some reason, we want to still attempt to insert as many records as possible, and output to the log those that for some reason returned error on insert attempt.

## Implementation

We are going to use a popular python library for working with sql: `psycopg2`.

```py
import psycopg2
from psycopg2 import extras as psycopg2_extras
from psycopg2 import sql
```

Let's first define the following utility function. It will manage connecting to the DB:

```py
async def connect_to_db(db_connection_string: str):
    try:
        conn = psycopg2.connect(db_connection_string)
        cursor = conn.cursor()

        return conn, cursor

    except Exception as e:
        logging.error(f"Error establishing connection to DB:")
        logging.error(str(sys.exc_info()))
        return None
```

We also need another utility function that checks if a record with specific primary key is already in our DB:
```py
async def check_for_potential_conflict_on_insert(conn, cursor, table_name: str, ids_to_check: list):
    # Check for conflicts by performing a separate SELECT query
    try: 
        select_query = sql.SQL("SELECT id FROM {} WHERE id IN ({})").format(
            sql.Identifier(table_name),
            sql.SQL(', ').join(sql.Placeholder() * len(ids_to_check))
        )
        
        cursor.execute(select_query.as_string(conn), ids_to_check)
        conflicting_ids = [fetched_rec[0] for fetched_rec in cursor.fetchall()]
        return conflicting_ids

    except Exception as e:
        logging.error(f"ERROR during records reading attempt:")
        logging.error(str(sys.exc_info()))
```


Now, our "write to db" function could have a workflow like this:

```py
async def write_records_to_table(conn, cursor, table_name: str, table_columns: tuple, records_for_db: list, ):
    # records_for_db - this is a list of tuples, where the first element of the tuple is a primary key in the db

    query = "" # this will be our insert query

    try:
        # we attempt to bulk-insert the data
        cursor.executemany(query.as_string(conn), records_for_db)
        conn.commit()

    except:
        logging.error(f"ERROR during operation to insert multiple records to DB, table {table_name}")
        logging.error(str(sys.exc_info()))
        conn.rollback()  # rollback the failed bulk transaction

        logging.info(f"Will attempt to insert records one by one:")
        for record_for_db in records_for_db:
            try:
                logging.info(f".inserting record {record_for_db[0]} to table {table_name}... ")
                # record_for_db[0] that's a primary key

                with conn.cursor() as single_cursor:
                    single_cursor.execute(query.as_string(conn), record_for_db)
                    conn.commit()
                
                logging.info(f".inserting record {record_for_db[0]} to table {table_name}... DONE")
            
            except:
                logging.error(f".ERROR inserting record {record_for_db[0]} to table {table_name}:")
                logging.error(str(sys.exc_info()))
                conn.rollback() # rollback the failed single transaction

                # in case of an error when inserting a single record, we can have yet another try/except block
                # try:
                    # do sth here
                # except:
                #     logging.error(str(sys.exc_info()))
                #     conn.rollback()                    
```

In terms of the insert query above, we can specify it using placeholders:

```py
placeholders = sql.SQL(', ').join(sql.Placeholder() for column in table_columns)

query = sql.SQL("INSERT INTO {} ({}) VALUES ({}) ON CONFLICT DO NOTHING").format(
    sql.Identifier(table_name), 
    sql.SQL(', ').join(sql.Identifier(column) for column in table_columns),
    placeholders
)
```

Now, with such prepared utility functions, in another section of the code we define what happens after the function gets triggerred:

```py
try:
    # first connecting to the DB
    conn, cursor = await connect_to_db(db_connection_string)

    # first detecting potential conflicts, 
    # getting the ids of records which will cause conflict on insert attempt
    # records below is list of records where each record is a dict format
    ids_to_check = [record["id"] for record in records]

    conflicting_ids = await check_for_potential_conflict_on_insert(conn, cursor, table_name, ids_to_check)

    # determining which records need to be inserted
    records_for_insert = []
    for record in records:
        if record["id"] not in conflicting_ids:
            # constructing record for insert
            record_for_insert = {}
            # first adding the id
            record_for_insert["id"] = record["id"]      


    logging.info(f"Summary:")
    logging.info(f".records TOTAL: {len(records)}")
    logging.info(f".records skipped: {len(conflicting_ids)}")
    logging.info(f".records for insert: {len(records_for_insert)}")
        
    # now attempting to insert data
    if len(records_for_insert) > 0: # only attempting to insert if there is something to insert

        logging.info(f"Preparing data for DB insert into table: {table_name} ... ")
        # we have dicts but we need list of tuples
        records_for_db = [] 
        for record in records_for_insert:
            record = *(record[key] for key in table_columns), 
            # * and , are to unpack the generator object
            records_for_db.append(record)

        logging.info(f"Attempting to write data to postgres table {table_name} ... ")
        await write_records_to_table(conn, cursor, table_name, table_columns, records_for_db)
        logging.info(f"Attempting to write data to postgres table {table_name} ... DONE")

except:
    logging.error(f"ERROR during DB write to table {table_name}: ")
    logging.error(str(sys.exc_info()))

finally:
    cursor.close()
    conn.close()
```