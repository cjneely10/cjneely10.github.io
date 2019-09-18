---
title: 'MetaSanity Demo part 1'
date: 2019-08-23
permalink: /posts/2019/08/MetaSanity-demo-part-1/
tags:
  - MetaSanity
  - demos
---

**MetaSanity** is a software designed for easy generation of dynamic data analysis pipelines. The heart of the software is a queryable SQL database that allows fast and easy retrieval of data - in this case, gene annotations and other related genomic data.

Why should I use MetaSanity?
======
**MetaSanity** provides two key benefits that make it a highly recommended tool for researchers.

1. Customizable and scalable pipelines
	1. Build an annotation pipeline from a variety of commonly used software packages.
	2. Fine tune flags for each program to match research needs.
	3. Re-run specific pipeline sections, or integrate additional analyses into an existing project.
2. Queryable output for project modularization and metadata retention.
	1. **BioMetaDB** project stores output from each pipeline as a SQLite3 database.
	2. All data files are kept in single location, and temporary file creation is minimized.
	3. Interface to command-line to query for fasta records with specific annotations, or to generate `.tsv` summary of all annotations.

MetaSanity demo
======
This blog post will walk through the data analysis pipelines available in **MetaSanity** using a small set of publicly available metagenomic-assembled genomes (MAGs). This post assumes that the Docker version of **MetaSanity** is being used; however, the source code users can reference this post so long as they use the valid source code configuration files from the [github repo](https://github.com/cjneely10/MetaSanity).

For even more usage examples, see [MetaSanity's usage page](https://github.com/cjneely10/MetaSanity/wiki/3-Usage).

For this blog, I have selected a group of genomes of variable quality and completion/contamination, including some redundant genomes. I will use the **PhyloSanity** pipeline to evaluate this subset for high quality, non-redundant genomes, which we will initially define as genomes with a completion score &ge;90% and a contamination score &le;5% via the CheckM pipeline. We define "non-redundant" as either 1) having no genomes in the data set with whom its percent average nucleotide identity (%ANI) is greater than or equal to 98.5, or 2) having &ge;1 separate genome share &ge;98.5% ANI, but having the maximum completion percent of the copies.
Once this is complete, I will use **FuncSanity** to design a customized pipeline based on the five available annotation suites, and I will use this pipeline to provide structural and functional annotations for each genome.
Finally. I will explore the results of my work by using the **BioMetaDB** SQL interface.

These datasets are available for download at [insert-url-here]().

Runtime for **MetaSanity** can be quite long for high numbers of genomes. Users are recommended to run all portions of this blog in a separate screen environment. For example, `screen -S test-run-MetaSanity`.

Installation
------
See the [wiki page](https://github.com/cjneely10/MetaSanity/wiki/2-Installation) for installation instructions. This blog post assumes that users have completed instructions for the optional `.bashrc` installation, which allow **MetaSanity** and **BioMetaDB** to be run using the aliases `MetaSanity` and `dbdm`, respectively.

Project preparation
======
In this blog, we will generate our **MetaSanity** project in the directory `$HOME/test-run`.

`mkdir $HOME/test-run && cd $HOME/test-run`

**MetaSanity** requires that genome files be present in a single directory. We will create a directory and fill it with the datasets provided for this blog post.

`mkdir genomes && cd genomes && wget insert-url-here/*`

Additionally, each **MetaSanity** pipeline requires a user-created configuration file. Default files are available in the MetaSanity program package in the folder `Config/Docker`. Each config file is broken up into two sections - an upper section of required information, and a lower section of optional information.

******

Part 1 - PhyloSanity
======

Config file preparation
------
Copy the default configuration file from your program package into your project directory.

`cd $HOME/test-run && cp /path/to/MetaSanity/Config/Docker/PhyloSanity.ini ./`

The config file `PhyloSanity.ini` can be used as-is; however, users may add additional flags or edit existing flags as needed. Edit the config file using `nano PhyloSanity.ini`.

![](https://cjneely10.github.io/files/phylosanity-ini.png)

For this blog, I have access to a server that is powerful enough to handle multi-threading and high-memory programs. So, I will increase the number of threads on all of my programs and will remove any options that reduce memory usage. At this step, users should provide settings that are suitable to their working environments.

In the `CHECKM` section, I will change the number of threads from `-t = 1` to `-t = 10`. Next, I will remove the line `FLAGS = --reduced_tree`. I will also change the number of pplacer threads from `--pplacer_threads = 1` to `--pplacer_threads = 10`.

The default values in the `FASTANI` and `BIOMETADB` sections are suitable and will not be changed.

In the optional `GTDBTK` section, no changes are needed. Users may choose to omit this evaluation step by commenting out the entire `GTDBTK` section of the config file.

In the `CUTOFFS` section, users may provide different (inclusive) values for identifying a genome as complete, contaminated, and non-redundant. This demo will use the values listed above (completion &ge;90%, contamination &le;5%, ANI &ge;98.5%), so I will change the line `IS_COMPLETE = 50` to `IS_COMPLETE = 90`.

![](https://cjneely10.github.io/files/phylosanity-ini-post.png)

Running PhyloSanity
------
The project directory `test-run` should resemble the following structure:

<pre><code>test-run/
├── genomes/
└── PhyloSanity.ini</code></pre>

From within this directory, run the **PhyloSanity** pipeline. You may redirect log messages to a separate file.

`MetaSanity -d genomes/ -c PhyloSanity.ini PhyloSanity 2>eval.log`

Depending on your system, this may run for several hours.

Once **PhyloSanity** is complete, the project directory structure will resemble the following:

<pre><code>test-run/
├── genomes/
├── eval.log
├── out/
├── Metagenomes/
└── PhyloSanity.ini</code></pre>

The `out/` directory contains the raw data from the pipeline output. The `Metagenomes` directory contains the `BioMetaDB` project, including the SQL interface, that has all raw data stored in a `SQLite3` database.

Within the `out/` directory are results directories for each output, labeled as `checkm_results`, etc. Also, running **PhyloSanity** generated a citations file that describes all of the analyses that were run in this pipeline.

We can generate a quick summary of this data using a command-line query:

`dbdm SUMMARIZE -c Metagenomes`

![](https://cjneely10.github.io/files/phylosanity-ini-1.png)

The database summary output is fairly self-explanatory - this table is named "evaluation", which was given to it by the settings in the `BIOMETADB` section of the `PhyloSanity.ini` config file that we used to run the pipeline. The number of records corresponds to the number of genomes evaluated. Averages and standard deviations are provided for numerical data categories. For text and boolean data categories, frequency values are provided - e.g., the most frequently occurring value, the number of times that value occurred, and the total number of nun-null values in that category.

This is great, but our completion requirement of 90% may be a bit strict for this dataset - only a small number of the initial genomes are of sufficient completion to be used downstream. 

In fact, if we query the results for high quality, non-redundant genomes, we can see that very few passed the combined filtering values:

`dbdm -c Metagenomes/ SUMMARIZE -t evaluation -q hqnr`

![](https://cjneely10.github.io/files/phylosanity-ini-2.png)

Quick note about queries:

- In the above command, the flag `-q hqnr` is a query, which filters a database table's results. In this case, the `hqnr` query was stored in the **BioMetaDB** program for convenience, and expands to `-q "is_complete AND NOT is_contaminated AND is_non_redundant"`. We are using this query on the `evaluation` table that we just created, so we provide this name using the `-t` flag.

- Notice the structure of this query statement - using the values in the `Database` column of the summary output, we created a filtering statement. We could perform similar queries, such as:
	- `-q "redundant_copies != '[]'"` to get genomes with redundant copies (note the quotes around strings)
	- `-q "completion > 90.0"` for genomes whose completion score is above a certain value (note the lack of quotes around the number)

Let's change some of these filtering criteria and rerun **MetaSanity**. We can adjust the `COMPLETION` value in `PhyloSanity.ini` to be lower - in this case, 70% - by replacing the line `IS_COMPLETE = 90` to `IS_COMPLETE = 70`.

Rerun **PhyloSanity** with this adjusted config file:

`MetaSanity -d genomes/ -c PhyloSanity.ini PhyloSanity`

Notice that this run completes within a few seconds. Here lies one of the benefits of **MetaSanity** - since we are only adjusting a filtering criteria, rerunning the pipeline occurs really quickly. As a counter example, if we were to delete the folder `checkm_results` in the output directory, then the pipeline would need to rerun this step, which would take longer to complete.

Let's query the database to see the results of this change in the completion filter:

`dbdm SUMMARIZE -c Metagenomes -t evaluation`

![](https://cjneely10.github.io/files/phylosanity-ini-3.png)

With the new criteria, the number of genomes that are deemed 'complete' has increased. We can see this in the row named `is_complete`, which now verifies that a higher number of genomes passed the lower completion filter.

We can also confirm that the number of high quality, non-redundant genomes has also increased.

`dbdm -c Metagenomes/ SUMMARIZE -t evaluation -q hqnr`

![](https://cjneely10.github.io/files/phylosanity-ini-4.png)

Great! Let's output these sequences to a separate folder - in this case, I will name it `HighQuality` - and focus our annotation pipeline on this subset. 

**BioMetaDB** has a very simple command-line interface for saving fasta records and summary tab-delimited data files. We can store fasta records to a folder using the `-w <folder-name>` flag. Combined with a query statement, we can very easily retrieve genomes that match our specific needs:

`dbdm -c Metagenomes/ SUMMARIZE -t evaluation -q hqnr -w HighQuality`

After running this command, your directory structure should resemble the following:

<pre><code>test-run/
├── genomes/
├── HighQuality/
├── eval.log
├── out/
├── Metagenomes/
└── PhyloSanity.ini</code></pre>

In the [next blog](https://cjneely10.github.io/posts/2019/09/MetaSanity-demo-part-2/), we will use **FuncSanity** to annotate this dataset!