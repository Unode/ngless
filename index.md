---
title: NGLess: NGS Processing with Less Work
---
# NGLess: NGS Processing with Less Work

This is a domain-specific language for NGS (next-generation sequencing data)
processing.

**Note**: This is *pre-release* software, currently available as a preview
only. Please [get in touch](mailto:coelho@embl.de) if you want to use it in
your work.

## Example

    ngless "0.0"
    input = fastq(['ctrl1.fq','ctrl2.fq','stim1.fq','stim2.fq'])
    preprocess(input) using |read|:
        read = read[5:]
        read = substrim(read, min_quality=26)
        if len(read) < 31:
            discard

    mapped = map(input,
                    reference='hg19')
    annotated = annotate(mapped,
                    features=[{gene}])
    write(count(annotated, count={gene}),
            ofile='gene_counts.csv',
            format={csv})

## Traditional Unix command line usage

`ngless` can be used as a traditional command line transformer utility, using
the `-e` argument to pass an inline script on the command line.

The `-p` (or `--print-last`) argument tells ngless to output the value of the
last expression to `stdout`.

### Converting a SAM file to a FASTQ file

    $ ngless -pe 'as_reads(samfile("file.sam"))' > file.fq

This is equivalent to the full script:

    ngless "0.0" # <- version declaration, optional on the command line
    samcontents = samfile("file.sam") # <- load a SAM/BAM file
    reads = as_reads(samcontents) # <- just get the reads (w quality scores)
    write(reads, ofname=STDOUT) # <- write them to STDOUT (default format: FASTQ)

### Getting aligned reads from a SAM file as FASTQ file

Building on the previous one-liner. We can add a `select()` call to only output
unmapped reads:

    $ ngless -pe 'as_reads(select(samfile("file.sam"), keep_if=[{mapped}]))' > file.fq

This is equivalent to the full script:

    ngless "0.0" # <- version declaration, optional on the command line
    samcontents = samfile("file.sam") # <- load a SAM/BAM file
    samcontents = select(samcontents, keep_if=[{mapped}]) # <- select only *mapped* reads
    reads = as_reads(samcontents) # <- just get the reads (w quality scores)
    write(reads, ofname=STDOUT) # <- write them to STDOUT (default format: FASTQ)

### Reading from STDIN

For a true Unix-like utility, the input should be read from standard input.
This can be achieved with the special file `STDIN`. So the previous example now
reads

    $ cat file.sam | ngless -pe 'as_reads(select(samfile(STDIN), keep_if=[{mapped}]))' > file.fq

Obviously, this example would more interesting if the input were to come from another
programme (not just `cat`).

[Full documentation](http://ngless.readthedocs.org/en/latest/)

## Building and installing

Again, please note that this is pre-release software. Thus, we do not provide
any easy to use packages at the moment. However, any comments (including bug
and build reports), are more than welcome.

Cabal version 1.18 or higher is necessary.

Running `make` should build a sandbox and download/install all dependencies
into it.


## Authors

- [Luis Pedro Coelho](http://luispedro.org) [coelho@embl.de](mailto:coelho@embl.de) [@luispedrocoelho](https://twitter.com/luispedrocoelho)
- Paulo Monteiro
- [Ana Teresa Freitas](http://web.tecnico.ulisboa.pt/ana.freitas/)

