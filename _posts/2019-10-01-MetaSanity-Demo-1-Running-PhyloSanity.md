---
title: 'MetaSanity Demo - Running PhyloSanity'
date: 2019-10-01
permalink: /posts/2019/10/MetaSanity-Demo-1-Running-PhyloSanity/
tags:
  - MetaSanity
  - demos
---

**MetaSanity** is a designed for easy generation of dynamic data analysis pipelines. The heart of the software is a queryable SQL database that allows fast and easy retrieval of genomic metadata - in this case, gene annotations and other related genomic data.

Why should I use MetaSanity?
======
**MetaSanity** provides two key benefits for researchers.

1. Customizable and scalable pipelines
	1. Build an annotation pipeline from a variety of commonly used software packages.
	2. Fine tune flags for each program to match research needs.
	3. Re-run specific pipeline sections, or integrate additional analyses into an existing project.
2. Queryable output for project modularization and metadata retention.
	1. **BioMetaDB** project stores output from each pipeline as a SQLite3 database.
	2. All data files are kept in single location, and temporary file creation is minimized.
	3. Interface to command-line to query for FASTA records with specific annotations, or to generate a tab-delimited summary of all annotations.

MetaSanity demo
======
This blog post will walk through the data analysis pipelines available in **MetaSanity**. The results I display is test data - this
blog post is meant to be followed using your own data.
 
This post assumes that the Docker version of **MetaSanity** is being used; however, the source code users can reference this post so long as they use the valid source code configuration files from the [github repo](https://github.com/cjneely10/MetaSanity).

For even more usage examples, see [MetaSanity's wiki page](https://github.com/cjneely10/MetaSanity/wiki).

I will use the **PhyloSanity** pipeline to evaluate a dataset for high quality, non-redundant genomes, which we will initially define as genomes with a completion score &ge;90% and a contamination score &le;5% via the CheckM pipeline. We define "non-redundant" based on a pairwise comparison of genome-wide average nucleotide identity (ANI). A non-redundant genome is identified if 1) no other genome(s) in the dataset have &ge;98.5% ANI, or 2) for any set of genomes that have &ge;98.5% ANI, the genome with the highest percent completion and lowest contamination.

In a follow-up [blog post](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-2-Running-FuncSanity/), I will use **FuncSanity** to design a customized pipeline based on the five available annotation suites, and I will use this pipeline to provide structural and functional annotations for each genome.
In a final [blog post](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-3-BioMetaDB/), I will explore the results of my work by using the **BioMetaDB** SQL interface.

Runtime for **MetaSanity** can be quite long for high numbers of genomes. Users should consider options to run this process in the background. For example, in Linux/Unix environments ‘screen’ can be used - `screen -S test-run-MetaSanity`.

Installation
------
See the [wiki page](https://github.com/cjneely10/MetaSanity/wiki/2-Installation) for installation instructions. This blog post assumes that users have completed instructions for the optional installation.

For simplicity's sake, this blog assumes that **MetaSanity** is installed in your home directory.

Project preparation
======
In this blog, we will generate our **MetaSanity** project in the directory `~/test-run`.

`mkdir ~/test-run && cd ~/test-run`

**MetaSanity** requires that genome files be present in a single directory. We will create a directory and fill it with the datasets provided for this blog post. The following command completes these tasks, assuming that your data is present in `~/DataSet/`.

`mkdir genomes && cp ~/DataSet/* genomes/`

******

Part 1 - PhyloSanity
======

Config file preparation
------
**PhyloSanity** pipeline requires a user-created configuration file. Default files are available in the MetaSanity program package in the folder `Config/Docker`. Each config file is broken up into two sections - a section for required parameters, and a section of optional information.

Copy the default configuration file from your program package into your project directory.

`cd ~/test-run && cp ~/MetaSanity/Config/Docker/PhyloSanity.ini ./`

The config file `PhyloSanity.ini` can be used as-is; however, users may add additional flags or edit existing flags as needed. You can edit the config file in a terminal using `nano PhyloSanity.ini`. The file `Complete-PhyloSanity.ini` in the project package contains the currently supported flags that are available to each program in the **PhyloSanity** pipeline.

<!-- ![](https://cjneely10.github.io/files/phylosanity-ini.png) -->
<pre><code># Default config file for running the FuncSanity pipeline
# DO NOT edit any PATH, DATA, or DATA_DICT variables
# Users are recommended to edit copies of this file only

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following **MUST** be set

[CHECKM]
PATH = /usr/local/bin/checkm
# Do not remove this next flag
--tmpdir = /home/appuser/tmp_dir
--aai_strain = 0.95
-t = 1
--pplacer_threads = 1
FLAGS = --reduced_tree

[FASTANI]
PATH = /usr/bin/fastANI
--fragLen = 1500

[BIOMETADB]
--db_name = MSResults
--table_name = evaluation
--alias = eval
FLAGS = -s

[CUTOFFS]
ANI = 98.5
IS_COMPLETE = 50
IS_CONTAMINATED = 5


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following sections may optionally be set
# Ensure that the entire section is valid,
# or deleted/commented out, prior to running pipeline


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Phylogeny prediction

[GTDBTK]
PATH = /usr/local/bin/gtdbtk
--cpus = 1</code></pre>

For this blog, I have access to a server that is powerful enough to handle multi-threading and high-memory programs. So, I will increase the number of threads on all of my programs and will remove any options that reduce memory usage. At this step, users should provide settings that are suitable to their working environments.

In the `CHECKM` section, I will change the number of threads from `-t = 1` to `-t = 10`. Next, I will remove the line `FLAGS = --reduced_tree`. I will also change the number of pplacer threads from `--pplacer_threads = 1` to `--pplacer_threads = 10`.

The default values in the `FASTANI` and `BIOMETADB` sections are suitable and will not be changed.

In the optional `GTDBTK` section, no changes are needed. Users may choose to omit this evaluation step by commenting out the entire `GTDBTK` section of the config file.

In the `CUTOFFS` section, users may provide different (inclusive) values for identifying a genome as complete, contaminated, and non-redundant. This demo will use the values listed above (completion &ge;90%, contamination &le;5%, ANI &ge;98.5%), so I will change the line `IS_COMPLETE = 50` to `IS_COMPLETE = 90`.

The **BioMetaDB** section provides me the option to name the project that this analysis will generate - in this case, `MSResults`. I also can set this value on the command line by passing the `-b project-name` command. No other changes should be made in this section.

After these changes, the **PhyloSanity** config file should resemble the following:

<!-- ![](https://cjneely10.github.io/files/phylosanity-ini-post.png) -->
<pre><code># Docker/PhyloSanity.ini
# Default config file for running the FuncSanity pipeline
# DO NOT edit any PATH, DATA, or DATA_DICT variables
# Users are recommended to edit copies of this file only

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following pipes **MUST** be set

[CHECKM]
PATH = /usr/local/bin/checkm
# Do not remove this next flag
--tmpdir = /home/appuser/tmp_dir
--aai_strain = 0.95
-t = 10
--pplacer_threads = 10

[FASTANI]
PATH = /usr/bin/fastANI
--fragLen = 1500

[BIOMETADB]
--db_name = MSResults
--table_name = evaluation
--alias = eval
FLAGS = -s

[CUTOFFS]
ANI = 98.5
IS_COMPLETE = 90
IS_CONTAMINATED = 5


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following pipe sections may optionally be set
# Ensure that the entire pipe section is valid,
# or deleted/commented out, prior to running pipeline


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Phylogeny prediction

[GTDBTK]
PATH = /usr/local/bin/gtdbtk
--cpus = 1</code></pre>

Running PhyloSanity
------
The project directory `test-run` should resemble the following structure, substituting your own values:

<pre><code>test-run/
├── genomes/
└── PhyloSanity.ini</code></pre>

From within this directory, run the **PhyloSanity** pipeline. You may redirect log messages to a separate file.

`MetaSanity -d genomes/ -c PhyloSanity.ini PhyloSanity 2>eval.log`

If we liked, we could specify an output directory using the `-o output_directory` flag. We may also overwrite the **BioMetaDB** directory name by using `-b biometadb_name`.

Depending on your system and the number of genomes, this may run for several hours.

Once **PhyloSanity** is complete, the default project directory structure will resemble the following:

<pre><code>test-run/
├── genomes/
├── eval.log
├── out/
  ├── checkm_results
  │ └── checkm_lineageWF_results.qa.txt
  ├── fastani_results
  │ └── fastani_results.txt
  ├── gtdbtk_results
    ├── GTDBTK.ar122.summary.tsv 
  │ └── GTDBTK.bac120.summary.tsv 
  ├── evaluation.list
  ├── evaluation.tsv
├── MSResults/
└── PhyloSanity.ini</code></pre>

By default, the `out/` directory contains the raw data from the pipeline output. The `MSResults` directory contains the `BioMetaDB` project, including the SQL interface, that has all metadata stored in a `SQLite3` database.

Within the `out/` directory are results directories for each output, labeled as `checkm_results`, etc. Also, running **PhyloSanity** generated a citations file that describes all of the analyses that were run in this pipeline.

We can generate a quick summary of this data using a command-line query:

`dbdm SUMMARIZE -c MSResults`

Note that we can omit the `-c MSResults` if we are working in a directory that has no other **BioMetaDB** projects.

`dbdm SUMMARIZE`

Below is the summary of a random dataset:

<pre><code>SUMMARIZE:	View summary of all tables in database
 Project root directory:	MSResults
 Name of database:		MSResults.db

*******************************************************************************************
	     Table Name:	evaluation  
  Number of Records:	10/10        

	        Database	Average             	Std Dev     

	      completion	92.882              	4.817       
	   contamination	3.365               	2.574       
-------------------------------------------------------------------------------------------

	        Database	Most Frequent       	Number    	Total Count 

	          _class	Phycisphaerales     	4         	10          
	          _order	SM1A02              	4         	10          
	          domain	Bacteria            	10        	10          
	          family	Gimesia             	2         	10          
	           genus	Gimesia             	2         	10          
	     is_complete	True                	7         	10          
	 is_contaminated	False               	7         	10          
	is_non_redundant	True                	10        	10          
	         kingdom	Planctomycetota     	10        	10          
	          phylum	Phycisphaerae       	4         	10          
	redundant_copies	[]                  	10        	10          
	         species	sp002683825         	1         	10          
-------------------------------------------------------------------------------------------</code></pre>

The database summary output is fairly self-explanatory - this table is named "evaluation", which was given to it by the settings in the `BIOMETADB` section of the `PhyloSanity.ini` config file that we used to run the pipeline. The number of records corresponds to the number of genomes evaluated. Averages and standard deviations are provided for numerical data categories. For text and boolean data categories, frequency values are provided - e.g., the most frequently occurring value, the number of times that value occurred, and the total number of values in that category.

This is great, but our completion requirement of 90% may be a bit strict for this dataset - only a small number of the initial genomes are of sufficient completion to be used downstream. 

In fact, if we query the results for high quality, non-redundant genomes, we can see that only half of the dataset passed the combined filtering values:

`dbdm -c MSResults/ SUMMARIZE -t evaluation -q hqnr`

<!-- ![](https://cjneely10.github.io/files/phylosanity-ini-2.png) -->
<pre><code>SUMMARIZE:	View summary of all tables in database
 Project root directory:	MSResults
 Name of database:		MSResults.db

*******************************************************************************************
	     Table Name:	evaluation  
  Number of Records:	5/10        

	        Database	Average             	Std Dev     

	      completion	94.978              	2.941       
	   contamination	1.884               	0.893       
-------------------------------------------------------------------------------------------

	        Database	Most Frequent       	Number    	Total Count 

	          _class	Planctomycetales    	2         	5           
	          _order	Planctomycetaceae...	2         	5           
	          domain	Bacteria            	5         	5           
	          family	Gimesia             	2         	5           
	           genus	Gimesia             	2         	5           
	     is_complete	True                	5         	5           
	 is_contaminated	False               	5         	5           
	is_non_redundant	True                	5         	5           
	         kingdom	Planctomycetota     	5         	5           
	          phylum	UBA1135             	2         	5           
	redundant_copies	[]                  	5         	5           
	         species	sp002683825         	1         	5           
-------------------------------------------------------------------------------------------</code></pre>

Quick note about queries:

- In the above command, the flag `-q hqnr` is a query, which filters a database table's results. In this case, the `hqnr` query was stored in the **BioMetaDB** program for convenience, and expands to `-q "is_complete AND NOT is_contaminated AND is_non_redundant"`. We are using this query on the `evaluation` table, so we provide this name using the `-t` flag.

- Notice the structure of this query statement - using the values in the `Database` column of the summary output, we created a filtering statement. We could perform similar queries, such as:
	- `-q "redundant_copies != '[]'"` to get genomes with redundant copies (note the quotes around strings)
	- `-q "completion > 90.0"` for genomes whose completion score is above a certain value (note the lack of quotes around the number)

- In order to prevent name issues with either Python or SQL, the "class" and "order" columns are replaced by "\_class" and "\_order", respectively.

Let's change some of these filtering criteria and rerun **PhyloSanity**. We can adjust the `COMPLETION` value in `PhyloSanity.ini` to be lower - in this case, 70% - by replacing the line `IS_COMPLETE = 90` to `IS_COMPLETE = 70`.

Rerun **PhyloSanity** with this adjusted config file:

`MetaSanity -d genomes/ -c PhyloSanity.ini PhyloSanity`

Notice that this run completes within a few seconds. Here is one of the benefits of **MetaSanity** - since we are only adjusting a filtering criterion, rerunning the pipeline occurs really quickly. As a counter example, if we were to delete the folder `checkm_results` in the output directory, then the pipeline would need to re-run this step, which would take longer to complete.

Let's query the database to see the results of this change in the completion filter:

`dbdm SUMMARIZE -c MSResults -t evaluation`

<!-- ![](https://cjneely10.github.io/files/phylosanity-ini-3.png) -->
<pre><code>SUMMARIZE:	View summary of all tables in database
 Project root directory:	MSResults
 Name of database:		MSResults.db

*******************************************************************************************
	     Table Name:	evaluation  
  Number of Records:	10/10        

	        Database	Average             	Std Dev     

	      completion	92.882              	4.817       
	   contamination	3.365               	2.574       
-------------------------------------------------------------------------------------------

	        Database	Most Frequent       	Number    	Total Count 

	          _class	Phycisphaerales     	4         	10          
	          _order	SM1A02              	4         	10          
	          domain	Bacteria            	10        	10          
	          family	Gimesia             	2         	10          
	           genus	Gimesia             	2         	10          
	     is_complete	True                	10        	10          
	 is_contaminated	False               	7         	10          
	is_non_redundant	True                	10        	10          
	         kingdom	Planctomycetota     	10        	10          
	          phylum	Phycisphaerae       	4         	10          
	redundant_copies	[]                  	10        	10          
	         species	sp002683825         	1         	10          
-------------------------------------------------------------------------------------------</code></pre>

With the new criteria, the number of genomes that are deemed 'complete' has increased. We can see this in the row named `is_complete`, which now verifies that a higher number of genomes passed the lower completion filter.

We can also confirm that the number of high quality, non-redundant genomes has also increased.

`dbdm -c MSResults/ SUMMARIZE -t evaluation -q hqnr`

<!-- ![](https://cjneely10.github.io/files/phylosanity-ini-4.png) -->
<pre><code>SUMMARIZE:	View summary of all tables in database
 Project root directory:	MSResults
 Name of database:		MSResults.db

*******************************************************************************************
	     Table Name:	evaluation  
  Number of Records:	7/10        

	        Database	Average             	Std Dev     

	      completion	93.377              	3.646       
	   contamination	1.980               	0.806       
-------------------------------------------------------------------------------------------

	        Database	Most Frequent       	Number    	Total Count 

	          _class	Planctomycetales    	2         	7           
	          _order	Planctomycetaceae...	2         	7           
	          domain	Bacteria            	7         	7           
	          family	Gimesia             	2         	7           
	           genus	Gimesia             	2         	7           
	     is_complete	True                	7         	7           
	 is_contaminated	False               	7         	7           
	is_non_redundant	True                	7         	7           
	         kingdom	Planctomycetota     	7         	7           
	          phylum	UBA1135             	2         	7           
	redundant_copies	[]                  	7         	7           
	         species	sp002683825         	1         	7           
-------------------------------------------------------------------------------------------</code></pre>

In the [next blog](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-2-Running-FuncSanity/), we will use **FuncSanity** to annotate this dataset!
