Next, we want to map the reads back on the assembly to see how the coverage for each contig changed across samples:

    bwa mem precomputed/assembly/final.contigs.fa \
            reads/sample_0.fq.gz \
            -o bam/sample_0.bam \
            -t 32

Here, we won’t run it for all samples to save time and space, you will find the bam files in the precomputed/bam/ folder. 

!!! question Exercise

    Feel free to inspect the content as done before, do you notice something particular?

Now, many tools need bam files to be sorted in order to work. Therefore, we will use samtools sort to do that.

    samtools sort precomputed/bam/sample_0.bam -o bam/sample_0.sorted.bam

The sorted bam files can also be index, so other tools can quickly extract alignments

    samtools index *.sorted.bam

Now, we will create a depth table, which can be used by binning tools to identify genomic entities (contigs, here) that have similar coverage across samples.

    jgi_summarize_bam_contig_depths --outputDepth depth.tsv bam/sample_0.sorted.bam bam/sample_1.sorted.bam bam/sample_2.sorted.bam bam/sample_3.sorted.bam bam/sample_4.sorted.bam bam/sample_5.sorted.bam

Now, we can run metabat to find our bins:

    metabat2 -a depth.tsv -i precomputed/assembly/final.contigs.fa.gz -o binned_genomes/bin

After this is done, we will now try and figure out how good are our bins, we will use checkm. First we create a set of lineage specific markers for bacterial genomes:

    checkm taxon_set domain Bacteria checkm_taxon_bacteria

Now, it’s time to find out the completeness of your binned genomes:

    checkm analyze -x fa checkm_taxon binned_genomes/ checkm_bac/

This created the checkm_bac/ folder with the information on our bins, to interpret the results we use checkm qa:

    checkm qa checkm_taxon checkm_bac/

!!! question Exercise

    What does it mean? How can we interpret this?

Now, we will let checkm predict the taxonomy of the bins and evaluate their completeness.

    checkm lineage_wf -t 8 -x fa binned_genomes/ checkm_taxonomy/

This command will take some time but it will give us a detailed breakdown of each genome predicted taxonomy and completeness.

With the bam files, a coverage file can also be created with the tool [CoverM](https://github.com/wwood/CoverM)

    coverm contig -b data/precomputed/bam/sample_*.sorted.bam -m mean -t 16 -o coverage.tsv