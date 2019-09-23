---
title: 'MetaSanity Demo part 3'
date: 2019-09-18
permalink: /posts/2019/09/MetaSanity-demo-part-3/
tags:
  - MetaSanity
  - demos
---

In this blog, we will continue our work from the [previous demo blog](https://cjneely10.github.io/posts/2019/09/MetaSanity-demo-part-2/). We had just completed both steps of the **MetaSanity** package, and we now have an output directory filled with output from each program and parsed tab-delimited data files.

We also have a working **BioMetaDB** project that contains all of our results, but better.

Now, how do we actually use it?

Part 3 - BioMetaDB
======

Goals
------
In this blog, we will explore the completed results of **MetaSanity**. Specifically,

1. Write simple queries for annotation results.
2. Create more complex queries that incorporate evaluation metrics.
3. Generate complete annotation and evaluation summaries.
4. Output relevant FASTA records to directory.

About
------
**BioMetaDB**, accessible through the script `MetaSanity/BioMetaDB/dbdm.py` of your MetaSanity installation, is a database generator. This package manages FASTA files and data files that describe them, handling all CRUD operations and providing a simple command-line interface.

Let's take a look at BioMetaDB's (admittedly involved) usage statement. You can find additional information [here](https://github.com/cjneely10/BioMetaDB) and additional examples [here](https://github.com/cjneely10/BioMetaDB/tree/master/Example).


BioMetaDB Usage
------

<pre><code>usage: dbdm.py [-h] [-n DB_NAME] [-t TABLE_NAME] [-d DIRECTORY_NAME]
               [-f DATA_FILE] [-l LIST_FILE] [-c CONFIG_FILE] [-a ALIAS] [-s]
               [-v VIEW] [-q QUERY] [-u UNIQUE] [-i] [-w WRITE] [-x WRITE_TSV]
               [-p PATH]
               program

dbdm:	Manage BioMetaDB project

Available Programs:

CREATE: Create a new table in an existing database, optionally populate using data files
		(Req:  --config_file --table_name --directory_name --data_file --alias --silent --integrity_cancel)
DELETE: Delete list of ids from database tables, remove associated files
		(Req:  --config_file --table_name --list_file --alias --silent --integrity_cancel)
FIX: Repairs errors in DB structure using .fix file
		(Req:  --config_file --data_file --silent --integrity_cancel)
INIT: Initialize database with starting table, fasta directory, and/or data files
		(Req:  --db_name --table_name --directory_name --data_file --alias --silent --integrity_cancel)
INTEGRITY: Queries project database and structure to generate .fix file for possible issues
		(Req:  --config_file --table_name --alias --silent)
MOVE: Move project to new location
		(Req:  --config_file --path --integrity_cancel --silent)
REMOVE: Remove table and all associated data from database
		(Req:  --config_file --table_name --alias --silent --integrity_cancel)
REMOVECOL: Remove column list (including data) from table
		(Req:  --config_file --table_name --list_file --alias --silent --integrity_cancel)
SUMMARIZE: Summarize project and query data. Write records or metadata to file
		(Req:  --config_file --view --query --table_name --alias --write --write_tsv --unique)
UPDATE: Update values in existing table or add new sequences
		(Req:  --config_file --table_name --directory_name --data_file --alias --silent --integrity_cancel)

positional arguments:
  program               Program to run

optional arguments:
  -h, --help            show this help message and exit
  -n DB_NAME, --db_name DB_NAME
                        Name of database
  -t TABLE_NAME, --table_name TABLE_NAME
                        Name of database table
  -d DIRECTORY_NAME, --directory_name DIRECTORY_NAME
                        Directory path with bio data (fasta or fastq)
  -f DATA_FILE, --data_file DATA_FILE
                        .tsv or .csv file to add, or .fix file to integrate
  -l LIST_FILE, --list_file LIST_FILE
                        File with list of items, typically ids or column names
  -c CONFIG_FILE, --config_file CONFIG_FILE
                        Config file for loading database schema
  -a ALIAS, --alias ALIAS
                        Provide alias for locating and creating table class
  -s, --silent          Silence all standard output (Standard error still displays to screen)
  -v VIEW, --view VIEW  View (c)olumns or (t)ables with SUMMARIZE
  -q QUERY, --query QUERY
                        Query to pass to SUMMARIZE
  -u UNIQUE, --unique UNIQUE
                        View unique values of column using SUMMARIZE
  -i, --integrity_cancel
                        Cancel integrity check
  -w WRITE, --write WRITE
                        Write fastx records from SUMMARIZE to outfile
  -x WRITE_TSV, --write_tsv WRITE_TSV
                        Write table record metadata from SUMMARIZE to outfile
  -p PATH, --path PATH  New path for moving project in MOVE command</code></pre>

Luckily, **MetaSanity** does most of the database creation and updating that is needed, leaving for us a finalized database that is ready for analysis. We can always add additional tables, or remove tables as needed. But, for this blog, we will focus on the `SUMMARIZE` option of **BioMetaDB**, which provides us an interface for handling all project queries. If you wish to learn more about additional functionality available to **BioMetaDB**, please visit its [github page]([BioMetaDB](https://github.com/cjneely10/BioMetaDB)).

1. Write simple queries for annotation results.
------

Let's take a peek at the results of **PhyloSanity**

`dbdm -c Metagenomes SUMMARIZE`

