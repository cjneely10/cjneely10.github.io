---
title: 'MetaSanity demo'
date: 2019-08-23
permalink: /posts/2019/08/MetaSanity-demo/
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
	1. **BioMetaDB** project stores output from each pipeline as a SQLite3 table.
	2. All data files and kept in single location.
	3. Interface to command-line to query for fasta records with specific annotations, or to generate `.tsv` summary of all annotations.

MetaSanity demo
======
This blog post will walk through the data analysis pipelines available in **MetaSanity** using a small set of publicly available metagenomic-assembled genomes (MAGs). This post assumes that the Docker version of **MetaSanity** is being used; however, the source code users can reference this post so long as they use the valid source code configuration files from the [github repo](https://github.com/cjneely10/MetaSanity).

For this blog, I have selected a group of genomes of variable quality and completion, including some redundant genomes (%ANI &ge; 98.5). I will use the **PhyloSanity** pipeline to evaluate this subset for high quality (completion &ge; 90%, contamination &le; 5% via CheckM pipeline), non-redundant genomes.

These datasets are available for download at [insert-url-here]().

Installation
------
See the [wiki page](https://github.com/cjneely10/MetaSanity/wiki/2-Installation) for installation instructions.

Project preparation
======
In this blog, we will generate our **MetaSanity** project in the directory `$HOME/test-run`.

`mkdir $HOME/test-run && cd $HOME/test-run`

**MetaSanity** requires that genome files be present in a single directory. We will create a directory and fill it with the datasets provided for this blog post.

`mkdir genomes && cd genomes && wget insert-url-here/*`

Additionally, each **MetaSanity** pipeline requires a user-created configuration file. Default config files are available in the MetaSanity program package in the folder `Config/Docker`. Each config file is broken up into two sections - an upper section of required information, and a lower section of optional information.

PhyloSanity
======

Config file preparation
------
Copy the default configuration file from your program package into your project directory.

`cd $HOME/test-run && cp /path/to/MetaSanity/Config/Docker/PhyloSanity.ini ./`

The config file `PhyloSanity.ini` can be used as-is; however, users may add additional flags or edit existing flags as needed.

![](https://cjneely10.github.io/files/phylosanity-ini.png)

For this blog, I have access to a server that is powerful enough to handle multi-threading and high-memory programs. So, I will increase the number of threads on all of my programs and will remove any options that reduce memory usage. At this step, users should provide settings that are suitable to their working environments.

In the `CHECKM` section, I will change the number of threads from `-t =1` to `-t = 10`. I will also remove the line `FLAGS = --reduced_tree`. Since each `pplacer` thread can add up to 100 GB of RAM usage, I will keep its default value at 1.

The default values in the `FASTANI` and `BIOMETADB` sections are suitable and will not be changed.

In the optional `GTDBTK` section, I will change the line `--cpus = 1` to be `--cpus = 4`. No other changes are needed. Users may choose to omit this evaluation step by 

In the `CUTOFFS` section, users may provide different (inclusive) values for identifying a genome as complete, contaminated, and redundant. This demo will use the above values, so I will change the line `IS_COMPLETE = 50` to `IS_COMPLETE = 90`, changing the required CheckM-determined completion score to a higher value to identify a genome as "complete".

![](https://cjneely10.github.io/files/phylosanity-ini-post.png)
