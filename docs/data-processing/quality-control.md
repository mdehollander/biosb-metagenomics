!!! important "Learning objectives"
    - Understanding the FastQ format
    - Interpret FastQC reports
    - Create high quality reads by trimmign and filtering with FastP and BBduk

## FastQ format

First let's take a look at the data

    zcat data/reads/sample_0.fq.gz | head


??? done "The output looks like this"
        @gi|1184849861|gb|KY629563.1|-34525/1
        CTGATCAACGTGGGAATGAACATGGAGCAGAGCGGCCAGTGTCTCAACCTGTGCTCCGCCTTCATGCACTCTCGCGGCGCCTTCAAGCCCGGCGACATCGACGTGGAAATTTTCACCATGCCCACCAAGTTCGGGCGCGTCAGCACCACG
        +
        DDDGGEG*DIIGH?KKHJGHGKKKJKKJJ9JDK=JKFKKGJHIKHEIJJJKKGIHKGDE1DKA>KEGBEEEBGI?BBEGHEEEFEEE9ED;FCB3BEEDCBEE5EEC'CA$E9EEEAEEEFEE?FFCEEE;$EE)DBCDEEDDECAAE'$
        @gi|1184849861|gb|KY629563.1|-34523/1
        GCCTTTGTCGGGTAAGGGGTGTGGCCCTCCTCCCGACAAGGCGGGCCACGGTTCGCCAGCGAACTAGGTCGGGGAGCTGGGAAGGAGCCGGAATCGGGTGGCCCCAATTTCGGGGAGAGGTTTGGGCGTCAGCCGCCCGGAAGCTCGTCG
        +
        DDDE2GGGIIIIIKKJKHJKEDJ=AIHKKHKJKKKKJHBIE$KJJJCEKJB=JH@JKEHKGEEEEDAJECDGC?EIFEBEDDF6FGEEDEE$FBDCEEA@EE$4EE$E??CDE?ED;CCDE1DEBE;ECC$?$$AEDA;EDD@$EEDE=F
        @gi|1184849861|gb|KY629563.1|-34521/1
        AGACCGAGCCCTTCCTTATAGTGGATTTCTCCGGTTCCGTCAACCAAATTTGCAGTAATGCGGGAGCGACGCTTCCACGAAATGGTCACTGTCCCGTCCACCTCGGAGAGCTTTACGCTTTCCGGTGTGTAGGGCTTGAGATCGATCGAT

There is a pattern that is repeated every 4 lines. This is the [FASTQ format](https://en.wikipedia.org/wiki/FASTQ_format). It follows this structure:

|Line|Description|
|----|-----------|
|1|Always begins with '@' and then information about the read|
|2|The actual DNA sequence|
|3|Always begins with a '+' and sometimes the same info in line 1|
|4|Has a string of characters which represent the quality scores; must have same number of characters as line 2|

The fourth line shows the quality of the read. To make the sequence and the qaility align well, the numerical score is converted into a code where each individual character represents the numerical quality of an individual nucleotide, following this scheme:

!!! done "FASTQ quality encoding"
            Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
                            |         |         |         |         |
            Quality score:    01........11........21........31........41

NextSeq and NovaSeq data often contains poly-g tails. Let's check if this also is the case for our simulated input

    $ zcat data/reads/sample_0.fq.gz | grep -E "GGGGGGGGGGGGGGGGGG$"

This search does not retun any hits. But often the end of the sequences contain stretches of poly-g. Below is an example how poly-g tails look like in Novaseq data.

??? done "Sequence data with poly-g tails"
        GGCCAGGACCACGCGGTGGAGCAGCGCGCCGCGGCGCCGGGCGCGCTTGACGACCAGTCTCTTATAAACATATCCCAGCCCACGAGACCACCAGTTGCATCTCGTATTCCGTCTTATGCTTGTATATTGGGGGGGGGGGGGGGGGGGGGGG
        GAGTCGATCGAGGAGATGAAGCACGCGGAGAAGGTCATTCACCGCATCCTCTACTTCGATGCTGTCTCTTATACACATCCCGAGCCCACGAGACCACCTGTTGCATCTCGTATGCCGTCTTCTGCTTGAAAAGGGGGGGGGGGGGGGGGGG
        GGCCCGTGCAGTTCGAGATCATCTCCGAGCCCACGAGACGACCGGGTGCTTGGCGGGAGCGGCGGGGTGGTTTTATCTTCGTGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
        GGGCGGCAACGCCGACACCTACCAGCTACAGCAGACGGTGCGCCGCACCATCGGCCGCTGGGTCGGCGGCCGGCTGCGCCGCCGTCCCAAGATAATCCCGGTGGTGGGGGGGGGTGGTTGGGGGGGTTGTGGGGGGGGGGGGGGGGGGGGG
        GGTAGGGCCGCTCGAGAAGCTCGCACAGCATGCGGCCGAATTCGCGGTACATGCATACGTTGACGTCGGCGGGGGGGGGGGGGGGGGGAGAGCACCGGGGGGCGCACAGGGCGGCCGGGTGGGGGTCGTGGTGGGGGGGGGGGGGGGGGGG
        GGGGAGGAGGGAGACGCGCCCGCGGGCGTGCCCGCCGCCGCGGCGATGCCCTTGAAGGCCGCTGGTGTGTCCTTCAGCGCCTGCGCCGCGAGTTCGCCGAACTGCTGCGTCAACGCGCCCCACCACTGCTGCGGGGGGGGGGGGGGGGGGG
        GTACCCGAGCCGCTCCGCGTGCCGCCGGACCTCCGACGCGTGAGGCCCGGAGCCGGTCGCTGACCAGAGGGCCAGGCCGAAGCGATGGGGAGTGGTCGCGACGAACCCGCAGCGCTGTCCGGCGGCGGGGGGGGGGGGGGGGGGGGGGGGG
        GAAGGTGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGCGGGGGGGGGGGGGGGCGGGGGCGGGGGGGGGGGGGGGGGGGGGGGGGGGG
        GCGCCACGCCGAGCACCGACGGCATCATCGGCACCCACACGCCGTGTGTGAACCTGTCTCTTATACACATCTCCGAGCCACGAGACCACCTGTTGCATCTCGTATGCCGTCTTCTGCTTGAAAATGGGGGGGGGGGGGGGGGGGGGGGGGG
        CTCTTCATCCGTTCCGGCGCCTGCATCCATTCCCGCGGCGCCGGTTGGCGGGGGGGGGGGGGGGGGGGGGGGGGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG

## FastQC reports

Let's check the quality of the data with FastQC before quality filtering:

    fastqc data/reads/sample_0.fq.gz

!!! question Exercise
    Open the FastqQC report `data/reads/sample_0_fastqc.html`

## Quality trimming and filtering

Because NovaSeq and NextSeq from Illumina contain often poly-g tails, it is good to use a trimming tool that can detect poly-g tails. Fastp can do that. We keep the defaults like they are, but specify we have interleaved input data. In case you have R1 and R2 file for each sample, you need to enable adapter detection with the `--detect_adapter_for_pe` flag. Execute fastp with this command:

    fastp -i data/reads/sample_0.fq.gz \
          --stdout \
          --interleaved_in \
          -q 25 \
          --cut_front \
          --cut_tail \
          --cut_mean_quality 25 \
          -l 51 \
          --thread 16 \
          --trim_poly_g > sample_0.trim.fastq

Now make a FastQC report again, to see the results of the quality filtering.

    fastqc sample_0.trim.fastq

!!! question "Exercise" 
    - Are there any adapter sequences detected?
    - Take a look at the html report

### Alternative trimming with bbduk

Alternative trimming with bbduk. Compare poly-g tail filtering, adapter trimming.

    bbduk.sh in=data/reads/sample_0.fq.gz  \
        out=sample_0.trim.bbduk.fastq.gz \
        interleaved=true \
        trimpolygright=1 \
        qtrim=w trimq=20 \
        minlength=51 \
        ref=nextera.fa.gz ktrim=r \
        stats=bbduk.stats \
        t=16

!!! question Exercise
    Create also an FastQC report for the trimming with bbduk 

## Remove PhiX sequences

    bbmap.sh ref=phix174_ill.ref.fa.gz \
             in=sample_0.trim.fastq \
             interleaved=true \
             outu=sample_0.nophix.fastq.gz \
             outm=sample_0.phix.fastq.gz \
             t=4


!!! question "Exercise" 
    - How many PhiX sequences are detected?
    - From which samples?
    - Can you confirm the sequences are PhiX?
    - The input data was simulated without adding any PhiX. How could there still PhiX sequences being detected?

Now again for all files with a for loop:

    for file in data/reads/*.gz; do \
        sample=$(basename ${file} .fq.gz);
        fastp -i $file --stdout --interleaved_in -q 25 --cut_front --cut_tail --cut_mean_quality 25 -l 51 --thread 16 --trim_poly_g > $sample.trim.fastq;
    done

## Quickly check what is in this metagenome

    sendsketch.sh --in=sample_0.nophix.fastq.gz threads=4 address=refseq

Now try it locally, using sourmash

    sourmash compute -k51 --scaled 10000 sample_0.nophix.fastq.gz -o sample_0.sig
    sourmash gather sample_0.sig gtdb-rs207.genomic-reps.dna.k51.lca.json.gz
    
Compare with kraken

    kraken2 --db k2_standard_08gb_20220926/ --threads 16 --output sample_0.kraken --report sample_0.kraken.report --gzip-compressed --use-names sample_0.nophix.fastq.gz
