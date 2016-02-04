---
title: "NGLess: NGS Processing with Less Work"
---
# NGLess: NGS Processing with Less Work

This is a domain-specific language for NGS (next-generation sequencing data)
processing.

**Note**: This is *pre-release* software, currently available as a preview
only. Please [get in touch](mailto:coelho@embl.de) if you want to use it in
your work (we heartily welcome beta testers). The features listed here all
currently work, but they might have some rough edges (for example, unhelpful
error messages when the inputs are not perfect – again, do get in touch if
you're interested and we can help).

## Example

    # We always declare a specific version at the top
    # this way, the script is 100% reproducible
    ngless "0.0"

    # Load input files
    input = fastq(['ctrl1.fq','ctrl2.fq','stim1.fq','stim2.fq'])

    # Some basic preprocessing
    preprocess(input) using |read|:
        read = read[5:]
        read = substrim(read, min_quality=26)
        if len(read) < 31:
            discard

    # Map (align) using one of the built in references (hg19: human).
    # Built-in references are downloaded the first time they are used.
    # You can, naturally, also use your own references
    mapped = map(input,
                    reference='hg19')

    # Annotate reads to genes. In this case, ngless knows that mapped comes
    # from hg19 and automatically uses the correct annotation file.
    counts = count(mapped,
                    features=[{gene}])

    # Finally write out the output:
    # Ngless is careful to write files out so that *partial results are never written*.
    # If you have a results file, it will be complete.
    write(counts,
            ofile='gene_counts.csv',
            format={csv})

## Basic features

- preprocessing and quality control of FastQ files
- mapping to a reference genome (implemented through [bwa](http://bio-bwa.sourceforge.net/))
- annotation and summarization of the alignments using reference gene annotations

Ngless has builtin support for some model organisms:

1. Homo sapiens (hg19)
2. Mus Muscullus (mm10)
3. Rattus norvegicus (rn4)
4. Bos taurus (bosTau4)
5. Canis familiaris (canFam2)
6. Drosophila melanogaster (dm3)
7. Caenorhabditis elegans (ce10)
8. Saccharomyces cerevisiae (sacCer3)

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

[Frequently Asked Questions (FAQ)](http://ngless.readthedocs.org/en/latest/faq.html)

## Building and installing

Again, please note that this is pre-release software. Thus, we do not provide
any easy to use packages at the moment. However, any comments (including bug
and build reports), are more than welcome.

Cabal version 1.18 or higher is necessary.

Running `make` should build a sandbox and download/install all dependencies
into it.


## Authors

- [Luis Pedro Coelho](http://luispedro.org) (email: [coelho@embl.de](mailto:coelho@embl.de)) (on twitter: [@luispedrocoelho](https://twitter.com/luispedrocoelho))
- Paulo Monteiro
- [Ana Teresa Freitas](http://web.tecnico.ulisboa.pt/ana.freitas/)

