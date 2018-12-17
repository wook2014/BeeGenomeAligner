# README

Scripts for generating genome alignments used in Rubin et al. (2019)

## Getting Started

### Repeat Masking

First, you'll need to have RepeatModeler (http://www.repeatmasker.org/RepeatModeler/) and RepeatMasker (http://www.repeatmasker.org/) installed. We used RepeatModeler version 1.0.4 and RepeatMasker version 4.0. We also used a local installation of NCBI BLAST version 2.5.0+. The processing of the repeats identified by RepeatModeler also depends on CD-HIT (http://weizhongli-lab.org/cd-hit/). We used version 4.6.

Our procedure for identifying repetitive elements in a genome and then masking those elements is given in repeats_run.sbatch. This was the SLURM script used for running repeat masking on *Apis florea* but the same process was used for each genome individually.

For comparing identified repetitive elements to known proteins, we downloaded all manually reviewed proteins in UniProtKB (http://www.uniprot.org/downloads) on Dec. 2, 2016. We used FlyBase release 5.50 for comparison to *Drosophila melanogaster* proteins. 

The newly identified repetitive elements were combined with all arthropod-derived repetitive sequences present in Repbase (https://www.girinst.org/) downloaded on March 8, 2017.

### Whole-genome alignments

We used LAST version 9.1.4 (http://last.cbrc.jp/) to generate pairwise alignments between all repeat-masked genomes and *Apis mellifera* using the last_run.sbatch SLURM script. This particular script aligns *Apis florea* against *Apis mellifera* but the same workflow was used for all taxa.

### Building multiple genome alignments

The rest of the python scripts synthesize these pairwise genome alignments into multiple genome alignments across taxa. ```maf_parser.py``` launches the pipeline. We used the command:

```
python maf_parser.py -i alignments -s AMEL -t AFLO,EMEX,BIMP,BTER,MQUA,HLAB,CCAL,MROT,LALB,DNOV -f gffs -w 500 -z 250 -o AMEL_base -m 8 -c bees.tree -p 14 -k AMEL_all_scafs.txt
```

```alignments/``` includes the pairwise alignment files generated using LAST. Here, *Apis mellifera* (```AMEL```) is used as the backbone for the alignment. The backbone species has to be the first species listed in the name of the maf file generated by LAST. The taxa listed in the ```-t``` parameter are those to be included in the final multiple alignment. There need to be files present in ```alignments/``` of the form ```AMEL.AFLO.sing.maf``` for every species included. ```gffs/``` needs to have gff files for each species. All entries of type ```CDS``` will be masked in the alignments. ```-w``` is the window size and ```-z``` is the window step size. 