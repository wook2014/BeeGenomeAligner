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
python maf_parser.py -i alignments -s AMEL -t AFLO,EMEX,BIMP,BTER,MQUA,HLAB,CCAL,MROT,LALB,DNOV -f gffs \
-w 500 -z 250 -o AMEL_base -m 8 -c bees.tree -p 14 -k AMEL_all_scafs.txt
```

```alignments/``` includes the pairwise alignment files generated using LAST. Here, *Apis mellifera* (```AMEL```) is used as the backbone for the alignment. The backbone species has to be the first species listed in the name of the maf file generated by LAST. The taxa listed in the ```-t``` parameter are those to be included in the final multiple alignment. There need to be files present in ```alignments/``` of the form ```AMEL.AFLO.sing.maf``` for every species included. ```gffs/``` needs to have gff files for each species. All entries of type ```CDS``` will be masked in the alignments. ```-w``` is the window size and ```-z``` is the window step size. ```-o``` is the output directory, ```-m``` is the minimum number of taxa required to include a multiple sequence alignment in the final dataset, and ```-p``` is the number of threads to use. ```-c``` is the newick-formatted phylogeny that will be used for estimating branch lengths for each locus. Only the topology is relevant. Branch lengths will be estimated. ```-k``` is simply a text file listing the scaffold sequences from the backbone species to include in the analysis. This is a useful way to limit the analysis to a subset of sequences so that multiple iterations can be launched simultaneously to speed up the process. All scaffolds should be included in the final pass.

You will need ```biopython``` (https://biopython.org/) and ```ete3``` (http://etetoolkit.org/) installed. ```RAxML``` (https://cme.h-its.org/exelixis/web/software/raxml/index.html), ```BASEML``` (from ```PAML```: http://abacus.gene.ucl.ac.uk/software/paml.html), ```Trimal``` (http://trimal.cgenomics.org/), and ```FSA``` (http://fsa.sourceforge.net/) also need to be available. You need to specify paths to the binaries at the top of ```utils.py```. Note that ```FSA``` should be compiled with the ```--with-mummer``` option so that it is optimized for long sequences. ```FSA``` will use ```MUMmer``` and ```Exonerate``` so these will also need to be in your path.

The basic procedure followed by ```maf_parser.py``` is to read in whole genome alignments of all taxa against the backbone species (specified by ```-s```). These alignments are filtered so as to be likely syntenic regions between species (i.e. if just a small fraction of a scaffold aligns to a particular scaffold in background, it is discarded). Adjacent alignments separated by fewer than 500 bases are then merged together to reduce the total number of individual aligning regions. Coding sequence is softmasked at this stage. It is only softmasked to allow its inclusion in the realignment step. Species are then added to the alignment one at a time based on the coordinates of the backbone species. Once all species have been added to the alignment of a particular backbone species, all sequences are realigned using ```FSA``` with the ```--anchored``` and ```--exonerate``` and ```--softmasked``` options. 

###Motif presence file

```motif_presence.txt.gz``` contains information on the motif predictions from Rubin et al. (2019). Column definitions are as follows:
```NCAR_locus```: name of NCAR
```Complex_eusocial_RER_result```: Indication of whether the relative rates test found that the complex eusocial lineages  at this locus were evolving significantly (p < 0.05) faster ("fast") or slower ("slow") than other lienages.
```Motif_name```: Consensus motif sequence.
```AMEL...DNOV```: Indication of whether this motif was present (1) or absent (0) in the NCAR sequence for each species.
```Greater_abundance_in_complex_or_other```: Whether the focal motif is present in a greater proportion of complex eusocial ("complex") or other ("other") species.
```#complex_with_motif```: Number of complex eusocial taxa with predicted presence of focal motif.
```#other_with_motif```: Number of other taxa with predicted presence of focal motif.
```prop._with_motif```: Proportion of complex eusocial taxa with predicted presence of focal motif.
```prop._with_motif```: Proportion of other taxa with predicted presence of focal motif.
```Chi2_p```: p-value of chi-square test for a difference in the presence of the focal motif between complex eusocial and other taxa.

