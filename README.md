# TERate

**TERate** is a computational pipeline to measure transcription elongation rates (TERs) with 4sUDRB-Seq.

## Features

Measure transcription elongation rates (TERs) with 4sUDRB-Seq.  
Different time points should be calculated separately.

## BAM format file

BAM file was originally mapped form TopHat, Bowtie, Bowtie2 or BWA.

##Prerequisites

###Software / Package

* [bedtools](https://github.com/arq5x/bedtools2)
* [GNU coreutils](http://www.gnu.org/licenses/gpl.html)

## Usage: 

-----------------------------------
To obtain average reads (Hits) distribution of 4sUDRB-Seq, BAM format file was converted to bedgraph format file firstly.
Following BAM format file was illustrated by the case of TopHat (v2.0.9) results of 4sUDRB-Seq 10 minute sample.
All C++ and shell scripts were marked ***'bold italic'***.
* 1. Please add the TERate directory to your **$PATH** first or copy all scripts to your current work directory (**'TERate_output/'**).
```bash
export PATH="~/TERate-master/:$PATH";

or copy all scripts to your current work directory ('TERate_output/')

mkdir TERate_output
cd TERate_output/
cp ~/TERate-master/bam2bedgraph ../TERate_output/
cp ~/TERate-master/gene_to_window ../TERate_output/
cp ~/TERate-master/split_bedgraph.sh ../TERate_output/
cp ~/TERate-master/split_refFlat.sh ../TERate_output/
cp ~/TERate-master/bedgraph_to_hits ../TERate_output/
cp ~/TERate-master/TER_calculate ../TERate_output/
```

* 2. BAM to bedgraph with ***'bam2bedgraph'*** script from bedtools. And Split gene annotation file **refFlat.txt** (Download form UCSC Genome Browser) into 300 bp bins/windows with ***'gene_to_window'*** script.
```bash
./bam2bedgraph accepted_hits.bam > accepted_hits.bedgraph
./gene_to_window refFlat.txt 300 > refFlat_bins.txt
```

* 3. To reduce time consumption of TERate, proposal for split **'bedgraph file'** (accepted_hits.bedgraph) and **'refFlat file'** (refFlat_bins.txt) into each chromosome with ***'split_bedgraph.sh'*** and ***'split_refFlat.sh'*** scripts.
Create split work directory **'split/'** and split bedgraph and refFlat into 300 bp bins/windows with ***'nohup'*** for backstage running.
```bash
mkdir split
cd split/
sh ../split_bedgraph.sh ../accepted_hits.bedgraph
sh ../split_refFlat.sh ../refFlat_bins.txt
```

* 4. After ***'split_refFlat.sh'*** and ***'split_bedgraph.sh'*** finished, then using ***'bedgraph_to_hits'*** to calculate Hits for each bins/windows (~ 3-4 hr time consumption).
Calculate each bin reads number (Hits) with ***'nohup'*** for backstage running.
```bash
ls |grep "bin" |awk -F"_" '{print "nohup ../bedgraph_to_hits "$1"_bedgraph.txt "$1"_bin.txt > "$1"_hits.txt &"}' |sh
```

* 5. When script ***'bedgraph_to_hits'*** finished, return to **'TERate_output/'** directory to combine all hit results and sort with gene name.
```bash
cd ../
cat split/*_hits.txt > combine_hits.txt
sort -k4,4 -k1,1 -k2,2n -k3,3nr combine_hits.txt > sorted_hits.txt
```

* 6. Calculate transcription elongation rate for each gene with ***'calculate_TER'*** script.
```bash
./calculate_TER sorted_hits.txt 10 300 |sort -k1,1 -k4,4nr |awk '{a[$1,++b[$1]]=$0}END{for(i in b)print a[i,1]}' > TERate_output.txt
```
**'TERate_output.txt'** is the result of TERate pipeline.  

-----------------------------------

## Note

Gene annotation file **refFlat.txt** is in the format ([Gene Predictions and RefSeq Genes with Gene Names](https://genome.ucsc.edu/FAQ/FAQformat.html#format9)) below (see details in [the example file](https://github.com/YangLab/TERate/blob/master/example/refFlat.txt)).

| Field       | Description                   |
| :---------: | :---------------------------- |
| geneName    | Name of gene                  |
| isoformName | Name of isoform               |
| chrom       | Reference sequence            |
| strand      | + or - for strand             |
| txStart     | Transcription start position  |
| txEnd       | Transcription end position    |
| cdsStart    | Coding region start           |
| cdsEnd      | Coding region end             |
| exonCount   | Number of exons               |
| exonStarts  | Exon start positions          |
| exonEnds    | Exon end positions            |

## Output

See details in [the example file](https://github.com/YangLab/TERate/blob/master/example/TERate_output_example.txt).

| Field       | Description                           |
| :---------: | :------------------------------------ |
| geneName    | Name of gene                          |
| isoformName | Name of isoform                       |
| strand      | + or - for strand                     |
| TER | Transcription elongation rate (bp/m)          |

## Requirements

* [GCC] gcc version 4.6.1
* [nohup] GNU GPL version 3
* [bedtools] (https://github.com/arq5x/bedtools2) v2.19.0

## Citation

**[Zhang Y\*, Xue W\*, Li X, Zhang J, Chen S, Zhang JL,Yang L# and Chen LL#. The Biogenesis of Nascent Circular RNAs. Cell Rep, 2016](http://www.cell.com/cell-reports/fulltext/S2211-1247(16)30329-1?rss=yes).**

## License

Copyright (C) 2016 YangLab.
See the [LICENSE](https://github.com/YangLab/CIRCpseudo/blob/master/LICENSE)
file for license rights and limitations (MIT).
