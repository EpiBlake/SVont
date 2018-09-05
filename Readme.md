 # SVants (pronounced sa&#183;vants)

Structural Variant Analysis with Nanopore unTargeted Sequencing

This program is designed to identify structural variants caused by mobile genetic elements (MGEs) or rearrangements in prokaryotic genomes and will work with Oxford Nanopore ultra-long read sequencing data generated by any of their sequencing platforms, as well as PacBio long-read sequencing data. 

The majority of tools available for long-read data rely on assemblies and annotation for the detection of structural variations within bacteria. This method is subject to missed or incorrect determination of structural variants due to assembly artifacts and can also miss population heterogeneity. The benefit of long-read data is we can detect any MGE or variant we are interested in and assess for tandem or inverted duplications and population heterogeneity, all within single reads, which avoids the possibility of assembly artifacts. 

We suggest using Oxford Nanopore data (ONT) over PacBio data as there is an inherent trade-off of length vs. data quality with PacBio data, as well as a theoretical limit of read length. With PacBio data, the longer the sequenced fragment, the lower the quality of the read. This makes using the accuracy of reads as filtering criteria for aligning to transposons but excluding related but divergent transposons difficult. 

Additionally, with the PacBio RSII, the upper limit of read length was around 40,000b, and reported read length limits with the Sequel are 70,000b. While this is excellent, we have routinely gotten read lengths of 150,000-200,000b with ONT and have sequenced quality reads in the 500,000-1,000,000b range on occasion. As the longer the read, the better the context of the structural variant, we choose to strive for longer reads.

## Version - 0.1

## Pipeline commands

Aligning to a reference genome to get accuracy rate and a report of sequencing error types:
```
[Git repo location]/Shell/LastalAlignToGenome.sh [Fasta reads] [Indexed reference genome] [Output location] [Config file]
```

Aligning to an MGE element and then determining the location(s) of the MGE within the reference genome
```
[Git repo location]/Shell/LastalAlignToElement.sh [Config file]
```

## Dependencies
The validated versions of this program are: 
* [Python](https://www.python.org)
	* version - 3.6.5
	* packages - bx-python

* [R](https://www.r-project.org)
	* version - 3.4.2
	* packages - rmarkdown, ggplot2, scales, Biostrings, formatR

* [Last](http://last.cbrc.jp)
	* version - 926

* [Bedtools](https://bedtools.readthedocs.io/en/latest/)
	* version - v2.27.1

## Pipeline installation

```
GIT COMMAND UPDATE!!!
```

## Configuration file

There is a template for the config file called **TEMPLATE.config**

We recommend you edit the template config file to include the locations of the scripts in the bottom half of the config file and then set the config file to read-only. You can then copy and rename the template config for each new sample and edit the top half of the script. This enables reproducible research as you will then always have a record of each location with the parameters you have chosen. 

**The TEMPLATE.config file contains:**
```
FASTAFILE=
ELEMENTREFERENCELOCATION=
GENOMEREFERENCELOCATION=
SUBSETSEQLENGTH=
GENETICELEMENTMATCH=
SEQQUALITY=
OVERLAPLEN=

###Pipeline specific locations (set this and forget it)
FILTERMAFLOCATION="Python/filter_maf.py"
SUBSETMAFLOCATION="Rscripts/Lastal_Filter_MAF_Output.R"
LASTALPROBABILITIES="Shell/LastalProbabilities.sh"
LASTALRSCRIPT_GENERAL="Rscripts/Lastal_General_Seq_Data.R"
RCALLLASTALRSCRIPT="Rscripts/Call_AlignmentStat_Report.R"
LASTALRSCRIPT_ALIGNMENTSTATS="Rscripts/Lastal_AlignmentStat.Rmd"
```
## Config file input files and parameters

**FASTAFILE** denotes the fasta file containing the Oxford Nanopore or PacBio reads. If you are using PacBio reads, we recommend the CCS reads. 

**ELEMENTREFERENCELOCATION** is the location of your selected MGE. If you think this is a single insertion of the MGE, then you can use the sequence of the MGE in fasta format. If you think there may be tandem or inverted repeats, you can create this structure for your element and then use that sequence as a reference. An example can be found in the section below that reproduces the example used in the publication. 

**GENOMEREFERENCELOCATION** is the location of a closely related reference genome. We will also often use a high quality long-read assembly of our sample as the reference here. 

**SUBSETSEQUENCELENGTH** indicates the threshold length of sequences you want to exclude from the analysis. As an example, if you only want to consider reads greater than 2,000 bases, you can use "2000" here. 

**GENETICELEMENTMATCH** indicates the threshold length of the alignment to the element within a single read for the read to be included in the analysis. As an example, if you want to fully match an MGE that is 1,200 bases long, you could use "1200" here.

**SEQQUALITY** denotes the average accuracy rate for your specific sequencing data as compared to the reference genome. Quality can be judged by aligning the ONT reads to a reference genome. This reference genome can be a known reference, but this will likely be inaccurate if the reference is not extremely related to your sample. We often assemble our data with [canu](https://github.com/marbl/canu), polish it with [Pilon](https://github.com/broadinstitute/pilon), and then realign our data to the assembly to get our error rate. We have included a script for this purpose called "LastalAlignToGenome.sh". An example can be found in the section below that reproduces the example used in the publication. 

**OVERLAPLEN** indicates the number of bases required to align to the reference genome in the final step of the pipeline. As an example, if you want to make sure 500 bases align to the reference genome to have confidence in the alignment location, you would use "500" here. 

The script locations in the final 6 lines of the config file are all found in the git repo for this tool. Each one is shown with its respective folder. All you need to do for these is add the path to the git repo on your computer. As an example, if you store the git repo in `~/Documents/git/` `FILTERMAFLOCATION="Python/filter_maf.py"` would become `FILTERMAFLOCATION="~/Documents/git/Python/filter_maf.py"`

## Make sure your reference genome and element sequence are indexed for lastal [lastdb](http://last.cbrc.jp/doc/lastdb.html)
```
lastdb REF REF.fasta
lastdb ELEMENT ELEMENT.fasta
``` 

## Output files
### Output from LastalAlignToGenome.sh

\*\_AlignedToReference.maf - The initial alignment file of each read as compared to the reference. This is stored in maf format.

\*\_AlignedToReference_probs.maf - Probability of the accuracy of reach read alignment against the reference. The method used to determine the probability can be found in the last program documentation under [last-map-probs](http://last.cbrc.jp/doc/last-map-probs.html)

\*\_AlignedToReference_probs.csv - A csv conversion of the alignments that pass the filter from last-map-probs. This is used to determine the overall error profile of your data and to generate the \*\_AlignmentStats.pdf document.

\*\_AlignmentStats.pdf - This is the final output of this script. It contains details and tables on the error profile of your data as aligned to your chosen reference. The overall accuracy rate at the bottom of the report is key to estimating the accuracy in the next script. 

### Output from LastalAlignToElement.sh

\*\_AlignedToElement.maf - The initial alignment file of each read as compared to the MGE. This is stored in maf format.

\*\_AlignedToElement.maf_withheader.maf - The same as the initial alignment file, but with a corrected maf header for subsequent steps. 

\*\_AlignedToElement_filter.csv - List of alignment stats for filtering and processing.

\*\_AlignedToElement_filter_subset.csv - List of reads that have passed filter cutoffs for alignment to the MGE. These cutoffs are SUBSETSEQLENGTH, GENETICELEMENTMATCH, SEQQUALITY, and OVERLAPLEN.

\*\__filtered.fasta - Fasta file that only contains the reads that align to the MGE and pass filtering.

\*\__MaskedForElement.fasta - Fasta file with the bases that align to the MGE soft masked. 

\*\__Filtered_AlignedToGenome.maf - Alignment file of each masked read as compared to the reference genome. This is stored in maf format. 

\*\__finalmapping.csv - This is the final out put of this script. This is a file containing the alignment locations of all of the masked reads, indicating where the MGE is inserted into the reference genome. 


# Reproducing the example from the publication

## Generating simulated reads
**These reads were used in the benchmark described in the publication**
NOTE: These are pre-populated and do not need to be recreated. This is included for reproducibility and transparency. 

### Artificial Reads were simulated using [NanoSim](https://github.com/bcgsc/NanoSim)

### The [NanoSim R9-2D](ftp://ftp.bcgsc.ca/supplementary/NanoSim/) training dataset was chosen as it has a lower error rate than the R9-1D training set. This more closely reflects the error rate of the current 1D kits available from Oxford Nanopore (although it is a higher error rate than our current experimentally observed rates).  
```
#Create 20,000 simulated Oxford Nanopore reads for the artificial chromosome. The size range of these reads is 35b-100,000b
NanoSim-2.1.0/src/simulator.py circular -r Artificial_Ecoli.fasta -c projects/cheny_prj/nanopore/paper/R9/2D/ecoli --max_len 100000

#20,000 reads were generated for the chromosome, and the plasmid is 2.1% the size of the chromosome, so we generate 420 plasmid reads
NanoSim-2.1.0/src/simulator.py circular -r Artificial_Plasmid.fasta -c projects/cheny_prj/nanopore/paper/R9/2D/ecoli --max_len 100000 -n 420 -o simulated_plasmid

#Make directory for simulated dataset and move everything into that location
mkdir Simulated
mv simulated* Simulated/
cd Simulated

#Combine the simulated reads into a combined fasta file
cat simulated*fasta > combined_simulated.fasta
```

## Create reference of MGE
NOTE: this can be any MGE that is inserted into a plasmid or chromosome. Additionally, if you think a plasmid has inserted into a chromosome, the MGE can be a plasmid. 

Given the longest length of the read in our simulated dataset is 100,000 bases, and the Tn9 transposon is 1,142 bases, theoretically we could resolve 87 tandem repeats if our longest possible read covered the repeat structure exclusively. In reality, we hope to see some of the junction between the transposon and the normal sequence of the plasmid or chromosome, and we don't expect to see that many repeats in a row for this example. We also know that the largest number of repeats in our artificial *E. coli* genome is 3 tandem repeats (because we made it). To show the ability to identify the number of repeats (but not overestimate), we have created an artificial pentamer repeat of the Tn9 transposon (example/Tn9_5xRepeat.fasta). This will be our element reference. 

## Index the genome and element references

Make sure to create the lastdb for the reference genome and element reference
```
lastdb Example/ArtificialEcoli/Artificial_Genome Example/ArtificialEcoli/Artificial_Genome.fasta
lastdb Example/ElementReference/Tn9_5xRepeat Example/ElementReference/Tn9_5xRepeat.fasta
```

## Create a config file specific for the publication example

NOTE: Make sure you update the paths of all variables in the config file to reflect where you have installed the git repository. We have placed a truncated path here, only showing the relative paths from the base directory of the git repo. 
```
FASTAFILE=Example/ArtificialEcoli/Simulated/combined_simulated.fasta
ELEMENTREFERENCELOCATION=Example/Tn9_5xRepeat.fasta
GENOMEREFERENCELOCATION=Example/ArtificialEcoli/Artificial_Genome.fasta
SUBSETSEQLENGTH=2000
GENETICELEMENTMATCH=1000
SEQQUALITY=
OVERLAPLEN=500

###Pipeline specific locations (set this and forget it)
FILTERMAFLOCATION="Python/filter_maf.py"
SUBSETMAFLOCATION="Rscripts/Lastal_Filter_MAF_Output.R"
LASTALPROBABILITIES="Shell/LastalProbabilities.sh"
LASTALRSCRIPT_GENERAL="Rscripts/Lastal_General_Seq_Data.R"
RCALLLASTALRSCRIPT="Rscripts/Call_AlignmentStat_Report.R"
LASTALRSCRIPT_ALIGNMENTSTATS="Rscripts/Lastal_AlignmentStat.Rmd"
```

The SUBSETSEQLENGTH was chosen to make sure reads that fully cover at least one copy of the Tn9 transposon will also have some overhang in the sequence to align to the genome. Additionally, we have set the GENETICELEMENTMATCH parameter was set to 1000 to make sure the match to the Tn9 element covers almost the full transposon. Both of these elements can be substantially lowered as long as there is not another MGE present with homology to the genetic element you are interested in. 

## Determine the error rate for your data

Next, we want to check the error rate of our data. This is an important step to get right as the current iteration of the tools will discard any match to the element that has an error rate more than 3% less than your estimate. 

```
Shell/LastalAlignToGenome.sh Example/ArtificialEcoli/Simulated/combined_simulated.fasta Example/ArtificialEcoli/Artificial_Genome Example/Output/ Test.config
```

Now we check the combined_simulated_AlignmentStats.pdf and see that our error rate is 81.9% on the first page. We now set the SEQQUALITY variable in our config file to 82. To reiterate, this is important because the LastalAlignToElement.sh script will discard any matches to your element that have an accuracy less than 79% (82% - 3%). 

The config file should now be complete. It should look like this, but with absolute paths added to all of the paths to reflect the location of the files and scripts on your system.
```
FASTAFILE=Example/ArtificialEcoli/Simulated/combined_simulated.fasta
ELEMENTREFERENCELOCATION=Example/Tn9_5xRepeat.fasta
GENOMEREFERENCELOCATION=Example/ArtificialEcoli/Artificial_Genome.fasta
SUBSETSEQLENGTH=2000
GENETICELEMENTMATCH=1000
SEQQUALITY=82
OVERLAPLEN=500

###Pipeline specific locations (set this and forget it)
FILTERMAFLOCATION="Python/filter_maf.py"
SUBSETMAFLOCATION="Rscripts/Lastal_Filter_MAF_Output.R"
LASTALPROBABILITIES="Shell/LastalProbabilities.sh"
LASTALRSCRIPT_GENERAL="Rscripts/Lastal_General_Seq_Data.R"
RCALLLASTALRSCRIPT="Rscripts/Call_AlignmentStat_Report.R"
LASTALRSCRIPT_ALIGNMENTSTATS="Rscripts/Lastal_AlignmentStat.Rmd"
```

## Align the reads to the Tn9_5xRepeat element sequence, and identify the locations of the element within the reference genome. 
```
Shell/LastalAlignToElement.sh Test.config
```

Using the **combined_simulated_AlignedToElement_filter_subset.csv**, we can count the number of repeats identified in our simulated reads. Remember that this file only contains reads with alignments that pass our filtering thresholds. Because we indicated we only want reads longer than 2,000 bases and alignments that cover at least 1,000 bases of the element, we only get those in this file. We ended up with 86 reads covering our element reference. If we look at the "identity_length", "reference_start", and "reference_stop" columns, we can see how many repeats we have detected.

	NOTE: the first column is "name" and contains the name of the artificial reads that align to the element. In the simulated reads, these include whether the artificial reads come from the plasmid or the chromosome. This will not be the case with your data, this is just a part of the NanoSim output. 

It is also helpful to look at the "query_start" and "query_stop" columns to see where the alignment match within each read matches to the element. We have used this in our own research to show we cross a repeat junction four separate times, indicating at least 5 repeats of our MGE. 

Based upon our data, we have observed up to three repeats within our data.

| Number of bases matched | Number of reads |     Interpretation     |
|:-----------------------:|:---------------:|:----------------------:|
|         ≤ 1,142         |        53       | One repeat detected    |
|       1,143-2,284       |        8        | Two repeats detected   |
|       2,285-3,426       |        25       | Three repeats detected |

Now it is time to see where these individual reads align to in the reference genome. To do this we use the **combined_simulated_Filtered_AlignedToGenome_finalmapping.csv** file. When we do this, we see that 85 of our 86 reads had enough alignment to the reference genome to pass our OVERLAPLEN filter of 500 bases. We also see that 57 of our reads align to the plasmid and 28 reads align to the chromosome. Additionally, the plasmid based insertions show to be in two different locations, around position 8,175 (which is a single insertion when we compare the reads to the number of repeats identified in the previous step), and around position 62,659 (which is a 3x tandem repeat insertion when we compare the reads to the number of repeats identified in the previous step). We also see that the chromosomal insertion is around position 146,348 in the chromosome (a single insertion). Each of these agrees with the artificial genome that was created and used with NanoSim. 

As a final note in re-creating the example from the publication. Due to the relatively high error rate of the data used in this example, the estimates of the insertion site are highly variable and do not accurately point to a specific nucleotide location. To get around this in our own work, we have used Illumina short reads to pinpoint the insertion site. We have done this by first aligning the Illumina data to the long reads that match our MGE of interest, and then filter for those that cross the junctions between the MGE and the plasmid or chromosome. Once we have those, we mask the portion of the reads that match the MGE and align the masked short reads to the reference genome, giving us a highly specific location for the insertion site. This can be done currently without the use of long-reads, but we find the added benefit of the long reads to identify tandem repeats and population heterogeneity useful enough to use both methods. Additionally, with recent increases in accuracy for ONT data to the mid 90% range, we are seeing much better ability to determine the specific insertion points without the need of short read sequencing data. 