# Functional annotation - Viromics

We will now perform the identification of viral sequences from metagenomic data. In addition, we will assign the taxonomy and a potential bacterial host for each of the identified viruses.

# Before starting

This practical requires Linux OS with "conda" installed. If you do not have access to any Linux OS, you can use virtual machines such as VirtualBox or Vagrant in MacOS or Windows.

Once "conda" is available, we need to install several different tools that will be required during the viral identification and annotation pipeline:
    - geNomad
    - CheckV
    - iPHOP

## Step 1 - Initial virus discovery

First, we run **geNomad** to identify viral sequences in our data. We use the ``--enable-score-calibration`` option to compute false discovery rates, allowing us to set a threshold to achieve a desired proportion of false positives (here we will use an FDR < 0.05). Further, we add the use the ``--cleanup`` flag to remove intermediate files and specify the number of threads using ``--cleanup``. As we are not using the ``--disable-find-proviruses`` option, geNomad will perform an initial prunning to remove potential contaminant host regions from proviral sequences. 

**Note:** The following command takes about Xh. You can also continue with the results in ``/data/precomputed/virome/vs2-pass1/``.

      genomad end-to-end \
          --enable-score-calibration \
          --disable-find-proviruses \
          X_metaspades_contigs.fa \
          geNomad_results \
          --cleanup \
          --threads 4 \
          /scratch/hb-llnext/databases/geNomad_db/


You can find a description of the geNomad output files [here]([https://github.com/jiarong/VirSorter2#detailed-description-on-output-files]).

**Quiz**: Look into ``geNomad_results/X_metaspades_contigs_virus_summary.tsv``. 

??? done "1. What is the length of the largest viral genome that we have detected?"
    X bases

??? done "2. Regarding the viral taxonomy, which is the most common viral class among identified viruses?"
    Caudoviricetes

??? done "3. Do we identify any single-stranded DNA virus (ssDNA)?"
    Yes, there is 1 ssDNA (X).     

 **Note:** An useful resource where you can find all the information about the viral taxonomy is [here]([https://ictv.global/taxonomy]).    

## Step 2 - Run CheckV

While geNomad performs well at identifying proviruses from metagenomic data, CheckV is specifically designed to identify host-virus boundaries with high precision. Further, CheckV allows to estimate the completeness of the predicted viral genomes based on their comparison to a database of complete viral genomes. Therefore, here we use CheckV to quality control the geNomad results and also to trim potential host regions left at the ends of proviruses. This command will take about 5 min to complete.

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

**Note:** The following command takes about 2h. You can also continue with the results in ``/data/precomputed/virome/vcontact/``.

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

**Note:** The following command takes about 30 min. You can also continue with the results in ``/data/precomputed/virome/vs2-pass2/``.

    virsorter run --seqname-suffix-off --viral-gene-enrich-off --provirus-off \
      --prep-for-dramv -i checkv/combined.fna -w vs2-pass2 --min-score 0.5 \
      -j 2 all

Then run DRAMv to annotate the identified sequences, which can be used for manual curation.

**Note:** The following command takes about 10 min. You can run it or continue with the precomputed results in ``/data/precomputed/virome/dramv-annotate/``.

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
