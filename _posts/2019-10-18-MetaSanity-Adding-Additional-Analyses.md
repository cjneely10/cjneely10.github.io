---
title: 'MetaSanity v1.1 - Adding additional analyses'
date: 2019-10-18
permalink: /posts/2019/10/MetaSanity-Adding-Additional-Analyses/
tags:
  - MetaSanity
  - demos
---

**MetaSanity v1.1** releases 10/18/2019 with some new features! See the [changelog](https://github.com/cjneely10/MetaSanity/blob/v0.0.4/CHANGELOG.md) for a complete list of changes in version 1.1. 

We also incorporated a more standardized analysis for determining genome quality ([Bowers et al](https://www.nature.com/articles/nbt.3893)). This blog will provide a walk-through for how we implemented this protocol.

Why do we want to explain this? **BioMetaDB** is designed to be updated, and the underlying SQL engine handles these updates easily.

Once a **BioMetaDB** project is created from a **MetaSanity** run, we can access this project from our own code.

MetaSanity v1.1 - Adding genome quality labels
======

Goals
------
Build an analysis that incorporates a **BioMetaDB** project. 

The script that we will generate in this blog is in your **MetaSanity** installation in the `Accessories` directory should you wish to complete the analysis without this mini-lesson in Python scripting.

In this blog, we will be:

- Adding a column to our **BioMetaDB** database table that describes a genome's quality as `high`, `medium`, `low`, or `incomplete`.
- Update existing data for `incomplete` genomes.


Preparing
------
Prior to running this script, ensure that **BioMetaDB** is on your python path. You can add it to your path by appending the following line to your `.bashrc` file and restarting your bash session:

`export PYTHONPATH=/path/to/MetaSanity/BioMetaDB:$PYTHONPATH`


Backing up your project
------
Since this particular update is released by our lab, a backup is not needed, assuming that you have not radically changed your project's architecture since running **MetaSanity**. 

However, this blog provides you with the tools to begin drastically changing your **BioMetaDB** project on your own. Users are STRONGLY advised to create a copy of their **BioMetaDB** project prior to running any update that is not directly released from this lab.


Part 1 - `RecordList` and `UpdateData`
------
Without getting too deep into OOP concepts, we can think of a `RecordList` as:

- A list (e.g. [record-1, record-2] etc.) AND a dictionary (e.g. {record1: metadata} etc.) of the metadata in our project
- A way to query our BioMetaDB project for specific gene calls, annotations, evaluation scores, etc.
- A way to get FASTA records or metadata
- A method to update the underlying database structure

We can think of `UpdateData` as an empty data frame - we put into it specific genome ids that we wish to update, and we add data categories and values as needed. This class generates a `.tsv` file, which **BioMetaDB** uses to update its database table schema.

Moreover, `UpdateData` is used to add *new* columns to our **BioMetaDB** project. If we are seeking to add a data column to our table (as we are here, the column "quality") *that does not already exist*, then we must use `UpdateData`. Otherwise, a `RecordList` returned by the function `get_table` will suffice for making updates to columns *that already exist* or for genome records *that already exist*.

Let's begin writing a small script. I will name this script `bowers_et_al_2017.py`.

Begin by adding these lines:

<pre><code>#!/usr/bin/env python3
import sys
from BioMetaDB import get_table, UpdateData

assert len(sys.argv) == 2, "usage: python3 bowers_et_al_2017.py biometadb-project"

evaluation_data = get_table(sys.argv[1], "evaluation")
evaluation_data.query()

dt = UpdateData()</code></pre>


The first two lines are pretty standard - we have our bash shebang for if we make this script executable, and we import the `sys` package for accessing command-line arguments. 

The following line imports the function `get_table`, which generates a `RecordList` for viewing our data. The data structure `UpdateData` is also imported. The next line is a check for users who run this script. Note that we have not checked that the path actually exists (we'll leave that to you).

Next, we use the function `get_table` to look at the **BioMetaDB** project the user passed for the table named "evaluation", and query this table for all records. Note: this requires that **PhyloSanity** ran successfully.

Finally, we create an `UpdateData` object named dt to store our data prior to updating the **BioMetaDB** project.


Part 2 - Gathering data
------
Now that all of our variables are initialized, we can search our project for valid data.

We will assign `high`, `medium`, `low`, and `incomplete` quality labels to our genomes based on the provisions established in the Bowers paper. In that paper, these qualities are defined as:

- High - Genome codes for 23S and 16S rRNA and at least 18 tRNAs. Genome is >90% complete and <5% contaminated based on `CheckM` output (presence of single copy genes).
- Medium - Genome completion &ge;50% complete and <10% contaminated.
- Low - Genome completion <50% complete and <10% contaminated.

We will define `incomplete` to be anything not meeting these criteria.

With that in mind, let's look at the next section of the script:

<pre><code>for genome in evaluation_data.keys():
    genome_id = genome.rstrip(".fna")
    genome_rl = get_table(sys.argv[1], table_name=genome_id)
    genome_rl.query("prokka LIKE 'tRNA%'")
    num_tRNAs = len(genome_rl)
    genome_rl.query("prokka == '23S ribosomal RNA'")
    has_23 = len(genome_rl) > 0
    genome_rl.query("prokka == '16S ribosomal RNA'")https://raw.githubusercontent.com/cjneely10/MetaSanity/v0.0.5/install.py
    has_16 = len(genome_rl) > 0
    has_23_16_rRNA = has_23 and has_16
    completion = evaluation_data[genome].completion
    contamination = evaluation_data[genome].contamination
    # Bowers et al determinations for MAG/SAG assembly quality
    if (num_tRNAs >= 18 and has_23_16_rRNA) and completion > 90 and contamination < 5:
        dt[genome].setattr("quality", "high")
    elif completion >= 50 and contamination < 10:
        dt[genome].setattr("quality", "medium")
    elif completion < 50 and contamination < 10:
        dt[genome].setattr("quality", "low")
    else:
        dt[genome].setattr("quality", "incomplete")
        evaluation_data[genome].is_complete = False</code></pre>

We are using a `for` loop to check each genome for our search criteria. By default, every key returned by `keys` is the filename of a FASTA file. **FuncSanity** generated a database table for every genome it analyzed, which stores out annotations. By calling `get_table` with this table name (less the extension), we get the associated annotations.

From here. we can query the `RecordList` stored in `genome_rl` just as we did the one stored in `evaluation_data`. We first check for tRNA annotations and store the number of records that match out query using the `len` function. We then query for 23S rRNA and check if any were returned. We do the same for 16S rRNA. The next line combines these two values for a single check - a bit superfluous, but it is better to be clear!

Next, we check our genome's completion value. Since `evaluation_data` is a list/dict, we can access it and manipulate the underlying stored values. In this case, we are getting the stored `completion` value. We do the same for contamination in the next line.

Finally, we assign qualities to our data. Each portion of the `if` statement refers to one of the Bowers quality assignments.

As I mentioned at the beginning of this blog, our goal is to update the underlying database with an additional column (the quality labels), and we wish to adjust any incomplete genomes to change their **PhyloSanity**-determined `is_complete` value to reflect their newly determined status.

The `RecordList` class handles changes to the existing architecture; but, if we want to add additional column data, we must incorporate an `UpdateData` object. In each portion of the `if` statement, we directly assign a genome's corresponding quality score by using `UpdateData`. In the `else` portion of the `if` block, we access the `RecordList` object and change its existing `is_complete` value.

Part 3 - Saving data and wrapping up
------

The final portion of our code consists of two lines:

<pre><code>evaluation_data.save()
evaluation_data.update(data=dt)</code></pre>

The first line updates all values that were changed in the existing database architecture.
The second line stores the new database info to the **BioMetaDB** project. Optionally, a folder of FASTA records could be provided to add to a database table by passing `directory_name="/path/to/genome-folder"`.

The `update` function will destroy the existing project architecture; so, if we want to use our new project structure, we must then call `get_table` again to get the new project data as a `RecordList`.

And that is it! We have just incorporated a genome quality analysis for all of our genomes in less than 40 lines of code.

Here is the complete script:

<pre><code>#!/usr/bin/env python3
import sys
from BioMetaDB import get_table, UpdateData

assert len(sys.argv) == 2, "usage: python3 bowers_et_al_2017.py biometadb-project"

evaluation_data = get_table(sys.argv[1], "evaluation")
evaluation_data.query()

dt = UpdateData()
for genome in evaluation_data.keys():
    genome_id = genome.rstrip(".fna")
    genome_rl = get_table(sys.argv[1], table_name=genome_id)
    genome_rl.query("prokka LIKE 'tRNA%'")
    num_tRNAs = len(genome_rl)
    genome_rl.query("prokka == '23S ribosomal RNA'")
    has_23 = len(genome_rl) > 0
    genome_rl.query("prokka == '16S ribosomal RNA'")
    has_16 = len(genome_rl) > 0
    has_23_16_rRNA = has_23 and has_16
    completion = evaluation_data[genome].completion
    contamination = evaluation_data[genome].contamination
    # Bowers et al determinations for MAG/SAG assembly quality
    if (num_tRNAs >= 18 and has_23_16_rRNA) and completion > 90 and contamination < 5:
        dt[genome].setattr("quality", "high")
    elif completion >= 50 and contamination < 10:
        dt[genome].setattr("quality", "medium")
    elif completion < 50 and contamination < 10:
        dt[genome].setattr("quality", "low")
    else:
        dt[genome].setattr("quality", "incomplete")
        evaluation_data[genome].is_complete = False

evaluation_data.save()
evaluation_data.update(data=dt)</code></pre>


There is much more that can be incorporated through the use of `UpdateData` and `RecordList`! See the [BioMetaDB](https://github.com/cjneely10/BioMetaDB) page for more information!


Citations
======
Bowers, Robert M et al. “Minimum information about a single amplified genome (MISAG) and a metagenome-assembled genome (MIMAG) of bacteria and archaea.” *Nature biotechnology* vol. 35,8 (2017): 725-731. doi:10.1038/nbt.3893. [https://www.nature.com/articles/nbt.3893](https://www.nature.com/articles/nbt.3893)
