First let's take a look at the data

    zcat data/sampleA_R1.fastq.gz | head


??? done "The output looks like this"
        @A00597:62:HGMVNDRXX:2:2264:20880:11788 1:N:0:CACCTGTTGC+ATTGACACAT
        GATCAGGCCAGCGAGCAGGTGGAGCAGCGTGCTCTTGCCGCACCCTGACGGACCGATCACCAGGCTGTGGGCGCCACGCTCGACCCGCCATGCCGGCAGCTCGAGCACGGTGCGGCCGCCGAAGACCTGGACCAGATCGCGCACCGCCACG
        +
        FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
        @A00597:62:HGMVNDRXX:2:1231:8278:1892 1:N:0:CACCTGTTGC+ATTGACACAT
        CCTTCGTTTAGCGCTCCAACCCGCCCTGGAAGCGCAGGCATTGCTGGCACACGTCCTGCTGCTGGAGGGAAATTCCGAACAGGCGCTGGCGGAGTACGCGGCGGTGTTGGCATTGGCGGATGGCTTTGACGAAGCGCTCTTCGGGGAAGGA
        +
        FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF,FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFF,
        @A00597:62:HGMVNDRXX:2:1116:19434:6590 1:N:0:CACCTGTTGC+ATTGACACAT
        GTGCACGAACGCCTGGGAGAGCCGGTCGCGGGGGCCGAGCTCCATGCTCCGCAGCTCCGGCTCCGGCGTCGGCGTGCCGAGGTCGTAGTCCTGGAGGTAGCGGTAGCCGTCCTTGTTGAGCAGCCAGCCGCCCTCCGCCCGGGTGGCCTCG

NextSeq and NovaSeq data often contains poly-g tails. Let's check for them

    $ zcat data/sampleA_R1.fastq.gz | grep -E "GGGGGGGGGGGGGGGGGG$" | head

As you can see, the end of the sequences contain stretches of poly-g.

??? done "Output"
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

    fastp --in1 data/sampleA_R1.fastq.gz \
          --in2 data/sampleA_R2.fastq.gz \
          --out1 sampleA_R1.trim.fastq.gz \
          --out2 sampleA_R2.trim.fastq.gz \
          --thread 4 \
          --detect_adapter_for_pe \
          --trim_poly_g \

??? done "Output"

        Detecting adapter sequence for read1...
        No adapter detected for read1

        Detecting adapter sequence for read2...
        No adapter detected for read2

        Read1 before filtering:
        total reads: 50000
        total bases: 7341730
        Q20 bases: 7177389(97.7615%)
        Q30 bases: 6907472(94.0851%)

        Read2 before filtering:
        total reads: 50000
        total bases: 7337452
        Q20 bases: 7093174(96.6708%)
        Q30 bases: 6740687(91.8669%)

        Read1 after filtering:
        total reads: 49298
        total bases: 7232810
        Q20 bases: 7094614(98.0893%)
        Q30 bases: 6836720(94.5237%)

        Read2 after filtering:
        total reads: 49298
        total bases: 7228671
        Q20 bases: 7029003(97.2378%)
        Q30 bases: 6690639(92.557%)

        Filtering result:
        reads passed filter: 98596
        reads failed due to low quality: 1372
        reads failed due to too many N: 0
        reads failed due to too short: 32
        reads with adapter trimmed: 914
        bases trimmed due to adapters: 7068

        Duplication rate: 0.014%

        Insert size peak (evaluated by paired-end reads): 271

        JSON report: fastp.json
        HTML report: fastp.html

        fastp --in1 data/sampleA_R1.fastq.gz --in2 data/sampleA_R2.fastq.gz --out1 sampleA_R1.trim.fastq.gz --out2 sampleA_R2.trim.fastq.gz --thread 4 --detect_adapter_for_pe --trim_poly_g 
        fastp v0.23.2, time used: 6 seconds

Check again for poly-g tails

    zcat sampleA_R1.trim.fastq.gz | grep -E "GGGGGGGGGGGGGGGGGG$"

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
             in1=sampleA_R1.trim.fastq.gz \
             in2=sampleA_R2.trim.fastq.gz \
             outu1=sampleA_R1.nophix.fastq.gz \
             outu2=sampleA_R2.nophix.fastq.gz \
             t=4

!!! question "Exercise" 
    How many PhiX sequences are detected?