# Human splicing diversity across SRA

This repo contains scripts and processed data for reproducing results from (insert preprint here), an analysis of human splicing diversity across 21,504 RNA-seq samples on SRA. The scripts here draw heavily from the scripts used to generate Abhi Nellore's Genome Informatics 2015 talk, available [here](https://github.com/nellore/gi2015).

The Python script `tables.py` generates all the data used to create the paper's figures in the Mathematica 10 notebook `figures.nb`. It depends on a master list of junctions contained in the file `all_SRA_junctions.tsv.gz`. The junction list may itself be reproduced by following the instructions at the end of this document. Results from running `tables.py` are contained in the following files whose formats are described below. See `tables.py`'s docstring for still more information.

### hg19.venn.txt
Intersections between junctions obtained from SEQC protocol and junctions
obtained from Rail for the 1720 samples studied by both SEQC and Rail. See
that file for details.

### hg19.sample_count_submission_date_overlap_geq_20.tsv
#### Tab-separated fields

1. count of samples in which a given junction was found
2. count of projects in which a given junction was found
3. earliest known discovery date (in units of days after February 27, 2009)
    -- this is the earliest known submission date of a sample associated with a
    junction

Above, each junction is covered by at least 20 reads per sample.

### hg19.[type].stats.tsv, where [type] is in {project, sample}
#### Tab-separated fields

1. [type] count
2. Number of junctions found in >= field 1 [type]s
3. Number of annotated junctions found in >= field 1 [type]s
4. Number of exonskips found in >= field 1 [type]s (exon skip: both 5' and 3'
    splice sites are annotated, but not in the same exon-exon junction)
5. Number of altstartends found in >= field 1 [type]s (altstartend: either 5'
    or 3' splice site is annotated, but not both)
6. Number of novel junctions found in >= field 1 [type]s (novel: both 5' and 
    3' splice sites are unannotated)
7. Number of GT-AG junctions found in >= field 1 [type]s
8. Number of annotated GT-AG junctions found in >= field 1 [type]s
9. Number of GC-AG junctions found in >= field 1 [type]s
10. Number of annotated GC-AG junctions found in >= field 1 [type]s
11. Number of AT-AC junctions found in >= field 1 [type]s
12. Number of annotated AT-AC junctions found in >= field 1 [type]s

### hg19.stats_by_sample.tsv
#### Tab-separated fields

1. sample index
2. project accession number
3. sample accession number
4. experiment accession number
5. run accession number
6. junction count
7. annotated junction count
8. count of junctions overlapped by at least 5 reads
9. count of annotated junctions overlapped by at least 5 reads
10. total overlap instances
11. total annotated overlap instances

Mathematica 10 was used to make all plots. See the notebook `figures.nb`.

## Recovering the junction list `all_SRA_introns.tsv.gz` used by `tables.py`

1. An account with Amazon Web Services is required to recover our results. Get one [here](http://aws.amazon.com/).
2. [Download](https://github.com/nellore/rail/raw/master/releases/install_rail-rna-0.1.7a) Rail-RNA v0.1.7a and follow the instructions at http://docs.rail.bio/installation/ to install it. Make sure to install and set up the Amazon Web Services (AWS) CLI as described there.
3. Familiarize yourself with how Rail-RNA works by reviewing the [tutorial](http://docs.rail.bio/tutorial/).
4. Download and install [PyPy 2.4](http://doc.pypy.org/en/latest/release-2.4.0.html).
5. Clone this repo, `runs`.
6. At the command line, enter

        cd /path/to/runs/sra/hg19
; that is, change to the `sra/hg19` subdirectory of your clone.
7. Run

        python create_runs.py --s3-bucket s3://[bucket] --region [AWS region] --c3-2xlarge-bid-price [lower price] --c3-8x-large-bid-price [higher price]
, where `[bucket]` is some S3 bucket you own where results will be dumped, `[AWS region]` is an AWS region (e.g., "us-east-1"), and `[lower price]`/`[higher price]` is an appropriate bid price for a c3.2xlarge/c3.8xlarge instance, respectively.
8. Several scripts will be (over)written; they will be named

        sra_batch_X_sample_size_K_prep.sh
        sra_batch_X_sample_size_K_itn.sh
, for `X` an integer between `0` and `42` inclusive and some `K`. Each script corresponds to a different job flow to be run on [Amazon Elastic MapReduce](https://aws.amazon.com/elasticmapreduce/). The `prep` script downloads FASTQs from the [EMBL-EBI](https://www.ebi.ac.uk/) server and dumps preprocessed versions of them on S3. The `itn` script aligns the data to find junctions. For each `X`, execute `sh sra_batch_X_sample_size_K_prep.sh`, wait until the job flow is done, and then execute `sh sra_batch_X_sample_size_K_itn.sh`. The scripts that are overwritten are the scripts we ultimately ran. We fiddled with bid prices in individual scripts because the [spot market](https://aws.amazon.com/ec2/spot/) was unpredictable.
9. Download the hg19 Bowtie index [here](ftp://ftp.ccb.jhu.edu/pub/data/bowtie_indexes/hg19.ebwt.zip) and unpack it.
10. Run

        sh retrieve_and_combine_results.sh [output] [bowtie1 idx] [bucket]
, where `[output]` is some output directory on your local filesystem (20 GB required), `[bowtie1 idx]` is the basename of the Bowtie index you just downloaded, and `[bucket]` is the S3 bucket you specified in step 4. The file `all_SRA_introns.tsv.gz`, which is used by `tables.py`, will be written to `[output]`.
