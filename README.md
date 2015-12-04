A simple tool for formulating and submitting batch jobs on Cray XE6 machines.
It is intended to be used in one-liners with GNU parallel. 


```
Usage: cpar [options] qsub_walltime output_directory

Options:
        -A alloc         - project allocation for the qsub -A parameter
        -c chunk         - number of nodes per qsub job (D = 5)
        -dry             - dry run
        -n nodes         - max total nodes to use (D = 10)
        -N name          - job name for the qsub -N parameter
        -q queue         - job queue for the qsub -q parameter
        -redo            - environment setting file to source 
        -s env           - env file to source before each job
        -t threads       - number of threads each command uses (D = 1)

Examples:

  > for i in {1..60}; do echo "echo \$i; sleep 60"; done | cpar -t 8 -n 10 -c 5 3:00 test

    The first part of this pipe tries to launch 60 dummy jobs each
    sleeping for one minute.  With "-c 5", we parallelize it by
    launching qsub jobs each using 5 nodes.  The "-t 8" specifies 8
    cores to be allocated for each command; thus, on a machine with
    24-core nodes, each aprun line will execute (24/8=) 3 commands on
    the same node. This means our 5-node qsub can handle at least
    (5*3=) 15 commands. Since we are allowed to use as many as 10
    nodes (-n 10), we will submit two such qsubs, which gives us a
    throughput of 30 commands in one iteration. The cpar tool will
    thus instruct each qsub to have two iterations (10 aprun
    lines). The completion time is thus 2 * 60 seconds plus overhead.
    Our 3-minute allocation should be sufficient.

  > find /path/to/query -name *.fasta | parallel --dry-run blastall -a 6 -p blastp -d /NR/nr -i {} | cpar -t 6 1:00:00 output

    This pipeline will launch 1 hour jobs on no more than 10 nodes to
    blast every query file against the NR. A large number of query files
    will result in multiple sequential aprun lines on each node. So the
    qsub job length needs to be adjusted manually to reflect that.
```
