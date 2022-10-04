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

If they poly-g tails would be of low quality, they would be trimmed of during quality control. But that is not the case. See this example:

    @A00597:62:HGMVNDRXX:2:1254:30798:1939 1:N:0:CACCTGTTGC+ATTGACACAT
    GAGTCGATCGAGGAGATGAAGCACGCGGAGAAGGTCATTCACCGCATCCTCTACTTCGATGCTGTCTCTTATACACATCCCGAGCCCACGAGACCACCTGTTGCATCTCGTATGCCGTCTTCTGCTTGAAAAGGGGGGGGGGGGGGGGGGG
    +
    :FF:FFF:FFFFFF:FFFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFF:FFF:FFFF:FFFFFFFFF:FFFFFFF,FFFFFFFFFFFFF:F,FF:FF:FF:F,,F::F:FFF,F,:,,,:FF:FFFFFFFFFFFF

The last G's have a quality character F, which stands for a phred score of 37.

Let's check the quality of the data with FastQC:

    fastqc -t 16 sample_0.trim.fastq 

Therefore it is good to use a trimming tool that can detect poly-g tails. Fastp can do that. We keep the defaults like they are, but specify we have interleaved input data. In case you have R1 and R2 file for each sample, you need to enable adapter detection with the `--detect_adapter_for_pe` flag. Execute fastp with this command:

    fastp -i data/reads/sample_0.fq.gz \
          --stdout \
          --interleaved_in \
          -q 30 \
          --thread 16 \
          --trim_poly_g > sample_0.trim.fastq

??? done "Output"
        Streaming uncompressed interleaved reads to STDOUT...
        Enable interleaved output mode for paired-end input.

        Read1 before filtering:
        total reads: 3329784
        total bases: 499467600
        Q20 bases: 455452270(91.1876%)
        Q30 bases: 421569914(84.4039%)

        Read2 before filtering:
        total reads: 3329784
        total bases: 499467600
        Q20 bases: 455447319(91.1866%)
        Q30 bases: 421564098(84.4027%)

        Read1 after filtering:
        total reads: 3329784
        total bases: 499450495
        Q20 bases: 455437723(91.1878%)
        Q30 bases: 421556912(84.4041%)

        Read2 after filtering:
        total reads: 3329784
        total bases: 499450639
        Q20 bases: 455432932(91.1868%)
        Q30 bases: 421551217(84.403%)

        Filtering result:
        reads passed filter: 6659568
        reads failed due to low quality: 0
        reads failed due to too many N: 0
        reads failed due to too short: 0
        reads with adapter trimmed: 532
        bases trimmed due to adapters: 32376

        Duplication rate: 0.0368492%

        Insert size peak (evaluated by paired-end reads): 38

        JSON report: fastp.json
        HTML report: fastp.html

        fastp -i data/reads/sample_0.fq.gz --stdout --interleaved_in -q 30 --thread 16 --trim_poly_g 
        fastp v0.23.2, time used: 26 seconds


!!! question "Exercise" 
    - Are there any adapter sequences detected?
    - Take a look at the html report

### Alternative trimming with bbduk

Alternative trimming with bbduk. Compare poly-g tail filtering, adapter trimming.

    bbduk.sh in=data/sampleA_R1.fastq.gz \
             in2=data/sampleA_R2.fastq.gz \
             out=sampleA_R1.trim.bbduk.fastq.gz \
             out2=sampleA_R2.trim.bbduk.fastq.gz \
             trimpolygright=1 \
             entropy=0.6 \
             entropywindow=50 \
             entropymask=f \
             qtrim=rl \
             trimq=15 \
             minlength=51 \
             ref=nextera.fa.gz \
             ktrim=r \
             stats=bbduk.stats \
             t=4


### Remove PhiX sequences

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

### Quickly check what is in this metagenome

    sendsketch.sh --in=sample_0.nophix.fastq.gz threads=4 -Xmx=500m

??? done "Output"
        Set threads to 4
        Adding /home/nioo/mattiash/.conda/envs/biosb-mg/opt/bbmap-38.98-1/resources/blacklist_refseq_merged.sketch to blacklist.
        0.016 seconds.
        Loaded 1 sketch in 15.006 seconds.

        Query: gi|1184849861|gb|KY629563.1|-34525/1	DB: RefSeq	SketchLen: 108675	Seqs: 6655980 	Bases: 998365454	gSize: 401140952	GC: 0.619	Quality: 0.4589	File: sample_0.nophix.fastq.gz
        WKID	KID	ANI	SSU	Complt	Contam	Matches	Unique	TaxID	gSize	gSeqs	taxName
        100.00%	1.20%	100.00%	.	100.00%	6.33%	1303	1147	1736466	4737944	4	Pseudolabrys sp. Root1462
        100.00%	1.01%	100.00%	.	100.00%	6.52%	1100	784	1736576	3882993	6	Pseudoxanthomonas sp. Root65
        100.00%	0.94%	100.00%	.	100.00%	6.59%	1019	746	1736498	3858949	1	Aeromicrobium sp. Root236
        100.00%	1.16%	100.00%	.	100.00%	6.37%	1262	0	1736476	4726341	5	Sphingopyxis sp. Root154
        100.00%	1.16%	100.00%	.	100.00%	6.37%	1262	0	1736491	4722837	7	Sphingopyxis sp. Root214
        100.00%	0.92%	100.00%	.	100.00%	6.61%	1002	32	1736608	3674685	14	Bacillus sp. Root920
        20.59%	1.05%	94.44%	.	100.00%	6.48%	1146	64	561879	20390K	1884	Bacillus safensis
        31.15%	0.82%	95.89%	.	100.00%	6.71%	891	2	293386	10528K	1299	Bacillus stratosphericus
        71.96%	0.66%	98.83%	.	100.00%	6.87%	721	2	2483356	3693088	38	Bacillus sp. 2D02
        70.54%	0.64%	98.77%	.	100.00%	6.89%	692	0	2512182	3687446	14	Bacillus sp. BK450
        7.88%	0.91%	91.22%	.	100.00%	6.62%	992	37	1408	47190K	8787	Bacillus pumilus
        63.93%	0.59%	98.41%	.	100.00%	6.94%	638	0	1857572	3680333	47	Bacillus sp. I-2
        30.22%	0.51%	95.77%	.	100.00%	7.02%	550	5	756828	6680144	2612	Bacillus sp. WP8
        48.00%	0.44%	97.38%	.	100.00%	7.09%	480	12	2500156	3673416	25	Bacillus sp. SDF0016
        45.53%	0.42%	97.21%	.	100.00%	7.11%	453	0	1739115	3653190	99	Bacillus sp. AM 13(2015)
        43.84%	0.42%	97.07%	.	100.00%	7.11%	452	3	2116541	3786396	149	Bacillus sp. Nf3
        45.57%	0.41%	97.21%	.	100.00%	7.12%	448	0	1706732	3638031	31	Bacillus sp. G1(2015b)
        25.10%	0.23%	95.13%	.	100.00%	7.30%	254	10	1326968	3648044	31	Bacillus australimaris
        22.51%	0.20%	94.75%	.	100.00%	7.33%	219	1	2108538	3632077	18	Bacillus sp. NMCC46
        21.70%	0.20%	94.63%	.	100.00%	7.33%	215	0	2108548	3605292	15	Bacillus sp. YBWC18

        Total Time: 	17.147 seconds.


Kraken2 or sourmash gather are alternatives