!!! important "Learning objectives"
    - Being able to create an assembly with megahit
    - Assess the quality of the assembly

Now that our reads are quality trimmed and ready to go is time to start the assembly. We can use [megahit](https://github.com/voutcn/megahit):

    megahit -r reads/sample_0.fq.gz, \
               reads/sample_1.fq.gz, \
               reads/sample_2.fq.gz, \
               reads/sample_3.fq.gz, \
               reads/sample_4.fq.gz, \
               reads/sample_5.fq.gz \
            -t 16 \
            -o megahit_assembly_meta/
            --presets meta-sensitive

This command would take around 50 minutes to complete, to speed up things we pre-assembled the data which is available in the precomputed/assembly/ folder. 

Feel free to inspect the contents of the folder by using

    ls -lh precomputed/assembly/

You will notice the final.contigs.fa file which contains the assembly

Now, we want to find out how well or poorly our assembly went. For this, we use quast, a tool to generate an assembly report.

    quast precomputed/assembly/final.contigs.fa -o quast/

Then, we inspect the output from quast. 

    less quast/report.txt

Not bad! We know our metagenome is not too large and if we care about contiguity, contigs above 5kb represent most of our community