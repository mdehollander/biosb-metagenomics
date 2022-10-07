# Functional annotation - Viromics

We will follow the virus discovery and AMG detection tutorial from the [Sullivan lab](https://www.protocols.io/view/viral-sequence-identification-sop-with-virsorter2-5qpvoyqebg4o/) with the data from this course. In addition, we will do taxonomic classification of the discovered viruses.

## Step 1 - Initial virus discovery

First, we run VirSorter2 with a loose cutoff of 0.5 for maximal sensitivity. We are only interested in phages (dsDNA and ssDNA phage, this is also the default in Virsorter2). A minimal length 5000 bp is chosen since it is the minimum required by downstream viral classification. Note that the``--keep-original-seq`` option preserves the original sequence of circular and (near) fully viral contigs (score >0.8 as a whole sequence) and we are passing them to checkV to trim possible host genes left at ends and handle duplicate segments of circular contigs.

**Note:** The following command takes about 2h. You can also continue with the results in ``/data/precomputed/virome/vs2-pass1/``

    virsorter run --keep-original-seq \
      -i /data/precomputed/assembly/final.contigs.fa \
      -w vs2-pass1 --min-length 5000 -j 2 all

You can find a description of the VirSorter2 output files [here](https://github.com/jiarong/VirSorter2#detailed-description-on-output-files). Look into ``vs2-pass1/final-viral-score.tsv``.

??? done "How many phages of which viral group do you detect?"
    1 ssDNA, 16 dsDNAphage

## Step 2 - Run CheckV

There could be some non-viral sequences or regions in the VirSorter2 results with a minimal score cutoff of 0.5. Here we use CheckV to quality control the VirSorter2 results and also to trim potential host regions left at the ends of proviruses. This command will take about 5 min to complete.

    checkv end_to_end vs2-pass1/final-viral-combined.fa checkv \
      -t 2 -d /data/databases/checkv-db-v1.4

The CheckV output is described [here](https://bitbucket.org/berkeleylab/checkv/src/master/). Look into ``checkv/quality_summary.tsv``.

??? done "How many proviruses do you find and how many viruses?"
    4 proviruses, 13 viruses

??? done "How many low, medium, and high quality viruses do you detect? How many of the viruses have direct terminal repeats?"
    5 low,  3 medium, 7 high quality, of them 3 have DTR

We will work with both the detected viruses and proviruses, so we combine them:

    cat checkv/proviruses.fna checkv/viruses.fna > checkv/combined.fna

## Step 3 - Taxonomic classification

We use vConTACT2 for taxonomic classification. This is done on the protein level and we first need to detect ORFs. This is done with prodigal:

    prodigal -p meta -a checkv/combined-prot.faa \
      -i checkv/combined.fna -f gff -o checkv/combined-prot.gff


Next we parse the prodigal output and prepare it for vConTACT2

    vcontact2_gene2genome -p checkv/combined-prot.faa \
      -o checkv/viral_genomes_g2g.csv -s 'Prodigal-FAA'

Finally, we are ready to cluster our sequences with the viral RefSeq.

**Note:** The following command takes about 2h. You can also continue with the results in ``/data/precomputed/virome/vcontact/``

    vcontact2 --raw-proteins checkv/combined-prot.faa --rel-mode 'Diamond' \
      --proteins-fp checkv/viral_genomes_g2g.csv \
      --db 'ProkaryoticViralRefSeq94-Merged' --pcs-mode MCL \
      --vcs-mode ClusterONE --c1-bin /opt/conda/bin/clusterone \
      --threads 2 --output-dir vcontact

The vConTACT2 output is described [here](https://bitbucket.org/MAVERICLab/vcontact2/wiki/Home). You can load the file ``genome_by_genome_overview.csv`` in a spreadsheet program and check which one of the contigs are clustered (they are in the end of the file). Alternatively, you can check which of the clusters in ``viral_cluster_overview.csv`` contain contigs:

    grep k141 viral_cluster_overview.csv

??? done "Which of the contigs are clustered with phages from RefSeq? What can you say about the taxonomy and host range of the clusters?"
    k141_124 - belongs to cluster VC_105 that has been split in 2 subclusters, the other subcluster contains Bacillus phages of the Myoviridae family  
    k141_1485 - clusters with Bacillus phages of the Siphoviridae family  
    k141_1484 - clusters with Escherichia phages of the Microviridae family  
    k141_1004 - clusters with Stenotrophomonas phages of the Inoviridae family  
    k141_292 - clusters with Sphingobium phages of the Siphoviridae family

## Step 4 - AMG detection

Then we run the checkV-trimmed sequences through VirSorter2 again to generate ``affi-contigs.tab`` files needed by DRAMv to identify AMGs. Note the``--seqname-suffix-off`` option preserves the original input sequence name since we are sure there is no chance of getting >1 proviruses from the same contig in this second pass, and the ``--viral-gene-enrich-off`` option turns off the requirement of having more viral genes than host genes to make sure that VirSorter2 is not doing any screening at this step.

**Note:** The following command takes about 30 min. You can also continue with the results in ``/data/precomputed/virome/vs2-pass2/``

    virsorter run --seqname-suffix-off --viral-gene-enrich-off --provirus-off \
      --prep-for-dramv -i checkv/combined.fna -w vs2-pass2 --min-score 0.5 \
      -j 2 all

Then run DRAMv to annotate the identified sequences, which can be used for manual curation.

**Note:** The DRAM databases are not correctly set up in the course environment, unfortunately. Thus, you will need to continue with the precomputed results in ``/data/precomputed/virome/dramv-annotate/`` and ``/data/precomputed/virome/dramv-distill/``

    DRAM-v.py annotate -i vs2-pass2/for-dramv/final-viral-combined-for-dramv.fa \
      -v vs2-pass2/for-dramv/viral-affi-contigs-for-dramv.tab -o dramv-annotate \
      --skip_trnascan --threads 2 --min_contig_size 5000

Finally, we use ``distill`` to summarize the annotations.

    DRAM-v.py distill -i dramv-annotate/annotations.tsv -o dramv-distill

The summarized output can be found in ``dramv-distill/amg_summary.tsv``.

??? done "Which contigs contain AMG and of which function? Which potential hosts have been identfied for them with the vConTACT2 analysis?"
    We find 2 contigs that contain a dUTP pyrophosphatase: k141_124 (potential Bacillus phage) and k141_1483 (unclustered)

To get an overview of all AMGs detected in the data, you can check ``dram-annotate/annotations.tsv``. The AMG flags (last column) are explained in the [DRAM publication](https://academic.oup.com/nar/article/48/16/8883/5884738).

You can also look into the original [protocol](https://www.protocols.io/view/viral-sequence-identification-sop-with-virsorter2-5qpvoyqebg4o/) for considerations on manual curation.
