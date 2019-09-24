---
title: 'MetaSanity Demo part 3'
date: 2019-09-18
permalink: /posts/2019/09/MetaSanity-demo-part-3/
tags:
  - MetaSanity
  - demos
---

In this blog, we will continue our work from the [previous demo blog](https://cjneely10.github.io/posts/2019/09/MetaSanity-demo-part-2/). We had just completed both steps of the **MetaSanity** package, and we now have an output directory filled with output from each program and parsed tab-delimited data files (TSVs).

We also have a working **BioMetaDB** project that contains all of our results, but better.

Now, how do we actually use it?

Part 3 - BioMetaDB
======

Goals
------
In this blog, we will explore the completed results of **MetaSanity**. Specifically,

1. Link evaluation with annotation.
2. Link putative metabolic function with annotation.
3. Link evaluation and putative metabolic function.
4. Create comprehensive query statements.

About
------
**BioMetaDB**, accessible through the script `MetaSanity/BioMetaDB/dbdm.py` of your MetaSanity installation, is a database generator. This package manages FASTA files and data files that describe them, handling all CRUD operations and providing a simple command-line interface.


**MetaSanity** output
------
If you've been following this blog, both pipelines in **MetaSanity** are now complete at this point, and we have a **BioMetaDB** project names `Metagenomes` to use. Within this directory are two summary database tables as well as an extra table for each genome that was processed. **PhyloSanity** outputs the summary table named `evaluation`, and **FuncSanity** outputs the summary table named `functions` if either the KEGG Pathway annotation or Peptidase annotation were completed.

You can view these tables easily using:

`dbdm -c Metagenomes SUMMARIZE -t evaluation`

`dbdm -c Metagenomes SUMMARIZE -t functions`

While this is definitely useful, **BioMetaDB** offers a far wider range of query options. Let's take a look at BioMetaDB's (admittedly involved) usage statement. You can find additional information [here](https://github.com/cjneely10/BioMetaDB) and additional examples [here](https://github.com/cjneely10/BioMetaDB/tree/master/Example).


BioMetaDB Usage
------

<pre><code>usage: dbdm.py [-h] [-n DB_NAME] [-t TABLE_NAME] [-d DIRECTORY_NAME]
               [-f DATA_FILE] [-l LIST_FILE] [-c CONFIG_FILE] [-a ALIAS] [-s]
               [-v VIEW] [-q QUERY] [-u UNIQUE] [-i] [-w WRITE] [-x WRITE_TSV]
               [-p PATH]
               program

dbdm:   Manage BioMetaDB project

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
                        evaluation ~> genome; function -> gen; eval >> fxn; eval ~> fxn -> gen;
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


`dbdm SUMMARIZE`
------

Let's focus on the section of the above usage statement that relates specifically to `SUMMARIZE`:

<pre><code>...
SUMMARIZE: Summarize project and query data. Write records or metadata to file
    (Req:  --config_file --view --query --table_name --alias --write --write_tsv --unique)
...
  -t TABLE_NAME, --table_name TABLE_NAME
                        Name of database table
  -c CONFIG_FILE, --config_file CONFIG_FILE
                        /path/to/BioMetaDB-project-directory
  -a ALIAS, --alias ALIAS
                        Provide alias for locating and creating table class
  -v VIEW, --view VIEW  View (c)olumns or (t)ables with SUMMARIZE                        
  -q QUERY, --query QUERY
                        evaluation ~> genome; function -> gen; eval >> fxn; eval ~> fxn -> gen;
  -u UNIQUE, --unique UNIQUE
                        View unique values of column using SUMMARIZE                        
  -w WRITE, --write WRITE
                        Write fastx records from SUMMARIZE to outfile
  -x WRITE_TSV, --write_tsv WRITE_TSV
                        Write table record metadata from SUMMARIZE to outfile
  -p PATH, --path PATH  New path for moving project in MOVE command</code></pre>

Viewing a BioMetaDB project requires, at the very minimum, a valid **BioMetaDB** project. Aside from that, users are able to provide filtering to fit their needs.

Before we start working through this blog's goals, let's explore a couple of examples.

Simple examples
------

View a summary of all genomes that were deemed high-quality and non-redundant by **PhyloSanity**.

`dbdm -c Metagenomes SUMMARIZE -t evaluation -q hqnr`

View a summary of gene calls for TOBG-CPC-51 that were given annotations by **FuncSanity**.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-51 -q annotated`

View a summary of all genomes that at least a partial glycolysis pathway.

`dbdm -c Metagenomes SUMMARIZE -t functions -q "glycolysis > 0"`

View a summary of gene calls for TOBG-CPC-51 that have both KO and MEROPS-Pfam annotations.

`dbdm -c Metagenomes SUMMARIZE -t tobg-np-997 -q "ko_annot AND merops_pfam_annot"`

View a list of all unique KO values assigned to TOBG-CPC-96 gene calls.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-96 -u ko`

View contigs in the TOBG-CPC-85 genome that were identified as likely being of phage origin.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-85 -q "num_phage_contigs_1 > 0"`

View a list of all of the tables in the database

`dbdm -c Metagenomes SUMMARIZE -v t`

View a list of all of the columns in each table of the database.

`dbdm -c Metagenomes SUMMARIZE -v c`


Great! Even on a simple level, we can begin to see some potential utility. If we were to couple any of the above examples with `-w out_fna` or `-x prefix`, then we could store all matching fasta records to an external directory, or generate a summary tab-delimited data file of all database table information, respectively.

Let's start working on some of these goals!

1. Link evaluation with annotation
======

Working with metagenomic requires researchers to take added care to ensure genome quality and minimize genome redundancy. **BioMetaDB** makes it very easy to handle these queries directly from the command line. Consider the following command:

`dbdm -c Metagenomes SUMMARIZE -t evaluation -q is_complete`

As expected, this command will query the evaluation table for genomes whose Boolean 'is_complete' value is set to True - e.g., genomes that were determined to be complete as part of the **PhyloSanity** pipeline. Coupling this with `-w out_fna` or `-x complete_genomes.tsv` would output all genomes to a directory named `out_fna` and provide genome evaluation summaries in the file `complete_genomes.tsv`, respectively.

Handy. Let's introduce a **query operator** - a specific delimiter that allows us to link our database tables. Consider if we wanted to gather a summary of all extracellular peptidases that were identified during **FuncSanity**, but we only want this information for our genomes deemed to be complete. This is easily accomplished with the following command:

`dbdm -c Metagenomes SUMMARIZE -q "is_complete ~> is_extracellular"`

Using this command, all genomes determined to be complete are iteratively queried for extracellular peptidases. This data is summarily displayed to the screen by genome. As with above, gene calls can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out`, with a TSV file generated for each genome that was queried.

In general, **we can link `evaluation` and `annotation`** using the format `evaluation query ~> annotation query`.

Here are a few more examples:

`dbdm -c Metagenomes SUMMARIZE -q "hqnr ~> annotated"`

`dbdm -c Metagenomes SUMMARIZE -q "NOT is_contaminated ~> prokka_annot"`

`dbdm -c Metagenomes SUMMARIZE -q "is_non_redundant ~> cazy_annot"`


2. Link putative metabolic function with annotation.
======

For researchers interested in metabolic function, gathering gene calls for genomes predicted to fulfill a particular pathway is useful for further downstream analysis.

Consider the following query:

`dbdm -c Metagenomes SUMMARIZE -t functions -q "chemotaxis > 0"`

As we expect, this query will check the `functions` table for genomes that have at least a partial pathway for chemotaxis. As before, the commands `-w out_fna` and `-x out.tsv` work as expected.

Let's introduce another **query operator**. Since we know that there are some genomes that are predicted to perform chemotaxis, we can narrow our search a bit more. A quick KEGG search reveals that the chemotaxis pathway includes, among others, K03409. Let's go ahead and combine these searches.

`dbdm -c Metagenomes SUMMARIZE -q "chemotaxis > 0 -> ko == 'K03409'"`

Using this command, we query each genome with a partial chemotaxis pathway for this specific KO. If we had not performed the online KEGG search, we could also have gained insight into these genomes with the following command:

`dbdm -c Metagenomes SUMMARIZE -q "chemotaxis > 0 -> ko_annot"`

As with above, gene calls can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out`, with a TSV file generated for each genome that was queried.

In general, **we can link `functions` and `annotation`** using the format `functions query -> annotation query`.

Here are a few more examples:

`dbdm -c Metagenomes SUMMARIZE -q "woodljungdahl > 0 -> prokka_annot"`

`dbdm -c Metagenomes SUMMARIZE -q "transporter_ammonia > 0.5 -> annotated"`

`dbdm -c Metagenomes SUMMARIZE -q "sulfur_assimilation == 1 -> annotated"`


3. Link evaluation and putative metabolic function.
======

In fine-tuning one's data, it may be useful to investigate evaluation and metabolic functions simultaneously. To do this, we introduce one final **query operator** in the following command:

`dbdm -c Metagenomes SUMMARIZE -q "is_complete >> riboflavin_biosynthesis > 0"`

As expected, this command looks for genomes that are determined to be complete and outputs genomes (not gene calls) with at least a partial riboflavin biosynthesis pathway. As with above, gene calls can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out.tsv`.

In general, **we can link `evaluation` and `functions`** using the format `evaluation query >> functions query`.


4. Create comprehensive query statements.
======

Awesome! Our final goal in this blog is to link all three of our database tables together - metagenome evaluation, metabolic functions, and genome-specific annotations.

We will need two of our **query operators** - `evaluation ~> annotation` and `functions -> annotations`.

Consider that our goal was to determine if any high-quality, non-redundant genomes of the kingdom Planctomycetota had at least a partially complete dissimilatory nitratereduction pathway, and to gather all gene calls that were assigned KO values. This is easily accomplished using the following:

`dbdm -c Metagenomes SUMMARIZE -q "hqnr AND kingdom == 'Planctomycetota' ~> dissim_nitrate_reduction > 0 -> ko_annot"`

Amazing! And, as before, we can store the gene calls or tsv data for this query using `-w out_fna` and `-x out`, respectively, outputting one tsv file per genome.

In general, **we can link `evaluation`, `functions`, and `annotations`** using the format `evaluation query ~> functions query -> annotation query`.


Wrapping up
======

In the past several blogs, we have walked through the use of **MetaSanity** to evaluate and annotate metagenomic-assembled genomes (MAGs). In this blog, we walked through **BioMetaDB**, an easy-to-use SQL package that lets users query their data and store results that are useful in their research. If you have any questions, feel free to send me a message!
