A simple tool for formulating and submitting batch jobs on Cray XE6 machines.
It is intended to be used in one-liners with GNU parallel. 


Usage: $0 [options] qsub_walltime output_directory

Options:
        -A alloc         - project allocation for the qsub -A parameter
        -c chunk         - number of nodes per qsub job (D = 5)
        -dry             - dry run
        -n nodes         - max total nodes to use (D = 10)
        -N name          - job name for the qsub -N parameter
        -q queue         - job queue for the qsub -q parameter
        -t threads       - number of threads each command uses (D = 1)
        -redo            - overwrite project directory

Examples:

```
  > for i in {1..60}; do echo "echo \$i; sleep 60"; done | cpar -t 8 -j 5 -n 10 3:00 test
```

    This pipeline will launch 2 qsub jobs using a total of 10 nodes (240 cores).
    Each aprun line will execute (24/8=) 3 commands on the same node. 
    Each qsub job will use 5 nodes with (60/10*5/3=) 10 aprun lines.
    This means each node will be mapped to 2 aprun lines and thus 2 minutes.
    Given the qsub overhead, our 3-minute-long qsub jobs may be adequate. 

```
  > find /path/to/query -name *.fasta | parallel --dry-run blastall -a 6 -p blastp -d /NR/nr -i {} | cpar -t 6 1:00:00 output
```
    This pipeline will launch 1 hour jobs on no more than 10 nodes to
    blast every query file against the NR. A large number of query files
    will result in multiple sequential aprun lines on each node. So the
    qsub job length needs to be adjusted manually to reflect that.

