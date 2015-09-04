# Updating-kgp-IDs-to-rs-IDs-for-SNPs-on-Illumina-HumanOmni2.5M-array

Adapted from: http://rstudio-pubs-static.s3.amazonaws.com/9686_5d78b74c4b9743cfb5ac63aba8d309b5.html

**Step 1: Create a file of genomic coordinates from your map/bim file**

You should first separate your genomic file according to chromosome. This can be done using the following command in PLINK:

e.g. plink --file yourdata --chr12 --recode --out yourdata_chr12

OR

e.g. plink --file yourdata --chr12 --make-bed --out yourdata_chr12

For conversion using the UCSC browser, your map file should be organized according to the following columns:

Chr | Position | Position +1 | rs ID

For example,

chr1 4158539 4158540 kgp499505

Using a linux environment, you can rearrange existing columns by using the following commands:

awk '{print "chr"$1,$4-1,$4,$2}' yourdata.bim > genomicCoordinates.txt

or

awk '{print "chr"$1,$4-1,$4,$2}' yourdata.map > genomicCoordinates.txt

Note: .bim and .map files provide genomic coordinates according to PLINK format.

Step 2: Navigate to UCSC Table Browser

http://genome.ucsc.edu/cgi-bin/hgTables

Step 3: Options selection

Original Genome: Human Original Assembly: Feb. 2009 (GRCh37/hg19) Group: Variation and Repeats Track: All SNPs(137) Table: snp137 Output format: selected fields from primary and related tables

Note: Make sure your genomic coordinates are according to hg19.

Step 4: Upload input file

On the “region” line, click the “define regions” button. Upload “genomicCoordinates.txt” file generated in step 1 by clicking “Choose File” button. Then click the “submit” button.

Note: There is a limit of 1,000 defined regions. To circumvent this you may choose to chunk your files according to chromosomal regions.

Step 5: Select output format

On the “output format” line, select “selected fields from primary and related tables”.

Step 6: Get output

You can enter a filename on the “output file” line or leave it blank to see the results on the screen. Enter “output.txt” and click the “get output” button.

In the new opened page “Select Fields from hg19.snp137”, check the “chrom”, “chromStart”, “chromEnd” and “name” checkboxes. Then click the “get output” button.

Step 7: What to do with insertions/deletions?

It is possible for your output to contain multiple records for one search. For example:

Input:

chr12 72650313 72650314 rs201804413 And the output:

chr12 72650313 72650317 rs66927394 chr12 72650313 72650314 rs201804413

Depending on your analysis, you may only want the second record which is not an insertion or deletion. Therefore you can simply remove the first record.

Using a linux environment you can remove all first records in duplicate outputs by using the following command:

awk '$3-$2==1{print}' output.txt > output2.txt

Note: “output.txt” is the output in step 6.

Step 8: Update IDs

To obtain the updated rs IDs, you can use the following commands using linux:

sort -k1 genomicCoordinates.txt > genomicCoordinates.sort.txt

sort -k1 output2.txt > output.sort.txt

join genomicCoordinates.sort.txt output.sort.txt | awk '{if($2==$5 && $3==$6) print $4,$7}' > updateIDs.txt

**Step 9: Rearrange output for PLINK format**

To rearrange the output file with the updated rs IDs so that they may be used for analyses using PLINK, use the following commands in linux:

e.g. awk '{print $1 =12, $4, $3*0, $3-1}' updateIDs.txt > newdata.map

Alternatively, you may use the following command in PLINK to updated your genomic SNP IDs:

plink --file yourdata -–update-map updateIDs.txt –-update-name
