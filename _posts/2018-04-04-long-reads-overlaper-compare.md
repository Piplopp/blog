---
layout: default
title: State-of-the-art long reads overlaper-compare
date: 2018-04-04
published: true
tags: draft overlaper benchmark
---

{% include setup %}

<script src="https://d3js.org/d3.v5.min.js"></script>
<script src="{{ POST_ASSETS_PATH }}/js/venn.js"></script>
<script src="{{ POST_ASSETS_PATH }}/js/my_venn.js"></script>
<link rel="stylesheet" href="{{ POST_ASSETS_PATH }}/css/venn.css">

# State-of-the-art long-read overlapping tools comparative analysis

## Introduction

In 2017, Chu *et la.* wrote a review [^fn1] to present and compare 5 long-read overlapping tools, on 4 datasets (including 2 synthetic ones).
This paper is very cool and clear. The authors compare overlappers with respect to peak memory, wall clock time, sensitivity and precision. Table 2 from this paper presents sensitivity and precision:

![table 2 of review]({{ POST_ASSETS_PATH }}/img/table_res_review.png)

Overlappers show better results on synthetic datasets than on real data. We can observe an important loss of sensitivity: 59.6-83.8% on the Pacbio real dataset, compared to 88.9-92.4% on the simulated data.

So, ok, overlappers dont't achieve perfect sensibility, but **do they miss the same overlaps**?

## Material & Methods

### Dataset

I selected the two real sequencing datasets in Chu *et al.*[^fn1], because they had the highest variance in sensitivity, so we can see the most extreme effects in how long-read overlappers possibly find different overlaps.

### What is an overlap

I will not bore you with formal definitions :)

We will consider 3 type of overlaps, according to Algorithm 5 presented in the minimap publication [^fn2]

![algorithm 5 in minimap and miniasm article by Heng Li]({{ POST_ASSETS_PATH }}/img/minimap_ovl_filter.png)


Internal match:
: Just a short similarity localized in the middle section of both reads, which is probably due to a repetitive region and not a real overlap

Containment:
: One read is completely contained in the other

Classic overlap:
: Deemed a regular suffix-prefix overlap


We will check the results of overlappers, and for each entry that isn't an internal match nor an containment overlap, we store the pair of reads as elements of the set of all overlaps found by the overlapper.

### Overlappers

We used:

- graphmap v0.5.2 [^fn3]
- hisea 39e01e98ca [^fn4]
- mhab 1.6 and 2.1 [^fn5]
- minimap 0.2-r124 [^fn2]
- minimap2 2.10 [^fn6]

We used parameters recommended by Chu *et al.*[^fn1] and default parameters for HISEA.

### Venn diagram generation 

We used a Python script to parse the output file of each overlapper, filter overlaps, generate a Venn diagram, and compute the Jaccard index.
All scripts and steps to reproduce this analysis are available in this [repository](https://github.com/natir/SOTA-long-read-overlapping-tools-comparative-analysis-data).

## Results

### Nanopore real data

<center>
<div id="venn_nanopore"></div>
</center>

<script>
var sets = [
    {"label": "graphmap", "sets": [0], "size": 10273911},
    {"label": 'hisea', "sets": [1], "size": 10114576},
    {"label": 'mhap', "sets": [2], "size": 11574382},
    {"label": 'minimap', "sets": [3], "size": 10419752},
    {"sets": [0, 1], "size": 9274898},
    {"sets": [0, 2], "size": 10078062},
    {"sets": [0, 3], "size": 10086933},
    {"sets": [1, 2], "size": 9824155},
    {"sets": [1, 3], "size": 9397447},
    {"sets": [2, 3], "size": 10224833},
    {"sets": [0, 1, 2], "size": 9098112},
    {"sets": [0, 1, 3], "size": 9179201},
    {"sets": [0, 2, 3], "size": 9910131},
    {"sets": [1, 2, 3], "size": 9219573},
    {"sets": [0, 1, 2, 3], "size": 9010533},
    ];
    generate_venn(sets, "#venn_nanopore", 800, 800, 25);
</script>

In the center of the above diagram we have the number of overlaps found by all overlappers. We call this set the _core overlaps_. Here for this dataset, core overlaps contain 9,010,533 overlaps.
Around this center, we highlight some of the largest disparities between overlappers:

|---------------------------------+-------------------+-------------------------|
| dataset composition             | number of overlaps |       % of core overlaps |
|:--------------------------------|------------------:|------------------------:|
| core overlaps - hisea overlaps    |       899,598     |     9.97 %              |
| hisea overlaps $$\cap$$ mhap overlaps    |       517,003     |     5.74 %              |
| core overlaps - graphmap overlaps |       209,040     |     2.32 %              |
| core overlaps - mhap overlaps     |       168,668     |     1.86 %              |
|---------------------------------+-------------------+-------------------------+

In addition, out of the 11,574,382 overlaps found by mhap, 5.73 % of these are found only by this overlapper. For hisea, the corresponding value is 1.02 % (out of 10,106,276 overlaps).

|-+-+-+-+-|
| | mhap | minimap | graphmap | hisea |
|-:|-:|-:|-:|-:|
| mhap | | 0.83 | 0.7 | 0.76 | 
| minimap | 0.83 | | 0.66 | 0.74 | 
| graphmap | 0.7 | 0.66 | | 0.74 | 
| hisea | 0.76| 0.74 | 0.74 | | 

The above matrix shows the Jacard similarity coefficient (cardinality of intersection divided by cardinality of union) between pairs of overlappers. 

### Pacbio real data

<center>
<div id="venn_pacbio"></div>
</center>

<script>
var sets = [
    {"label": "graphmap", "sets": [0], "size": 3847631},
    {"label": 'hisea', "sets": [1], "size": 4278872},
    {"label": 'mhap', "sets": [2], "size": 5336610},
    {"label": 'minimap', "sets": [3], "size": 5640643},
    {"sets": [0, 1], "size": 3449577},
    {"sets": [0, 2], "size": 3778499},
    {"sets": [0, 3], "size": 3812461},
    {"sets": [1, 2], "size": 4151564},
    {"sets": [1, 3], "size": 4222400},
    {"sets": [2, 3], "size": 4976545},
    {"sets": [0, 1, 2], "size": 3416628},
    {"sets": [0, 1, 3], "size": 3435767},
    {"sets": [0, 2, 3], "size": 3759953},
    {"sets": [1, 2, 3], "size": 4120738},
    {"sets": [0, 1, 2, 3], "size": 3407577},
    ];
    generate_venn(sets, "#venn_pacbio", 800, 800, 25);
</script>

For the Pacbio dataset, core overlaps contain 3,407,577 overlaps. 
Other large disparities between overlappers are:

|---------------------------------+-------------------+-------------------------|
| dataset composition             | number of overlaps |       % of core overlaps |
|:--------------------------------|------------------:|------------------------:|
| core overlaps - graphmap overlaps |       713,161     |     20.92 %             |
| minimap2-only overlaps                |       538,118     |     15.78 %             |
| mhap overlaps $$\cap$$ minimap overlaps  |       503,431     |     14.76 %             |
| core overlaps - hisea overlaps    |       352,376     |     10.33 %             |
| mhap-only overlaps                    |       319,744     |     9.38 %              |
|---------------------------------+-------------------+-------------------------+

Out of all overlaps found by minimap2 (5,640,643), 9.54% of these overlaps are found only by this overlapper, for mhap the corresponding value is 5.98% (out of 5,336,610 overlaps).

|-+-+-+-+-|
| | mhap | minimap | graphmap | hisea |
|-:|-:|-:|-:|-:|
| mhap | | 0.87 | 0.86 | 0.83 | 
| minimap | 0.87 | | 0.95 | 0.84 | 
| graphmap | 0.86 | 0.95 | | 0.83 | 
| hisea | 0.83 | 0.84 | 0.83 | | 

Again the above matrix shows Jaccard similarity coefficient.


### Comparison across versions

At first we used mhap 2.1, but in [^fn1], Chin. *et al* use mhap 1.6. This version difference yielded strange results: many more overlaps were found only by mhap 2.1.
So we make a comparison between mhap 1.6 and 2.1, in terms of shared and exclusive overlaps.

**Sergey Koren, a co-author of mhap, told us that mhap 1.6 calculates a similarity score between reads and mhap 2.1 calculates a distance between reads, the meaning of the -\-threshold option is different between the two versions, so we can not use the same values.** This point explain strange result we get, below, we plotted the Venn diagrams of overlaps found only by mhap 1.6 (with -\-threshold 0.02 like [^fn1]) and only by mhap 2.1 with -\-threshold 0.98 (1 - 0.02).

<center>
<div id="venn_mhap_pacbio"></div>
Pacbio, Jacard similary 0.7

<div id="venn_mhap_nanopore"></div> 
Nanopore, Jacard similary 0.98
</center>

<script>
var sets = [
    {"label": "minimap", "sets": [0], "size": 4048048},
    {"label": 'minimap2', "sets": [1], "size": 5640643},
    {"sets": [0, 1], "size": 4010485},
    ];
    generate_venn(sets, "#venn_mhap_pacbio", 400, 400, 50);
</script>

<script>
var sets = [
    {"label": "minimap", "sets": [0], "size": 10429792},
    {"label": 'minimap2', "sets": [1], "size": 10419752},
    {"sets": [0, 1], "size": 10339593},
    ];
    generate_venn(sets, "#venn_mhap_nanopore", 400, 400, 50);
</script>

Clearly, mhap 2.1 finds many more overlaps than mhap 1.6.

And another comparison between minimap and minimap2:

<center>
<div id="venn_minimap_pacbio"></div>
Pacbio, Jacard similary 0.7

<div id="venn_minimap_nanopore"></div> 
Nanopore, Jacard similary 0.98
</center>

<script>
var sets = [
    {"label": "minimap", "sets": [0], "size": 4048048},
    {"label": 'minimap2', "sets": [1], "size": 5640643},
    {"sets": [0, 1], "size": 4010485},
    ];
    generate_venn(sets, "#venn_minimap_pacbio", 400, 400, 50);
</script>

<script>
var sets = [
    {"label": "minimap", "sets": [0], "size": 10429792},
    {"label": 'minimap2', "sets": [1], "size": 10419752},
    {"sets": [0, 1], "size": 10339593},
    ];
    generate_venn(sets, "#venn_minimap_nanopore", 400, 400, 50);
</script>




## Conclusion

Overlapper tools behave quite similarly, but on real pacbio datasets[^fn1], sensibility, precision, and the set of overlaps found across tools can be very different.
Such a difference can also exist between two versions of the same tool.

Comparison of overlappers based on a quantitative measurement (sensitivity, precision) is useful but isn't perfect: two tools with the same sensitivity for a given set could still detect a different set of overlaps, see e.g. mhap and Minimap for the nanopore set.

Some publications use quality of error-correction, or results of genome assembly, as quality metrics to compare overlappers. It's a good idea but correction and assembly tools make additional choices in the overlaps they keep, and it's not easy to relate assembly or error-correction imperfections and wrong or missed overlaps.

It could by interesting to study whether certain tools have a bias when finding overlaps, linked to e.g length of reads, mapping length, error rate, %GC, specific kmer composition, etc …
A study like this could possibly reveal some intrinsic properties of the algorithms used in overlappers.

Is it a good idea to create a reconciliation tool for overlappers? We note that the correction and assembly tools seek to reduce the amount of overlaps they use, through e.g. graph transitivity reduction, Best Overlap Graph, the MARVEL approach[^fn7].

##### Acknowledgment

- Sergey Koren
- Rayan Chikhi
- Jean-Stéphane Varré

##### Reference

[^fn1]: Innovations and challenges in detecting long read overlaps: an evaluation of the state-of-the-art by Chu *et al.* doi:[10.1093/bioinformatics/btw811](http://doi.org/10.1093/bioinformatics/btw811)
[^fn2]: Minimap and miniasm: fast mapping and de novo assembly for noisy long sequences by Heng Li doi:[10.1093/bioinformatics/btw152](https://doi.org/10.1093/bioinformatics/btw152)
[^fn3]: Fast and sensitive mapping of nanopore sequencing reads with GraphMap by Sovic *et al.* doi:[10.1038/ncomms11307](https://doi.org/10.1038/ncomms11307)
[^fn4]: HISEA: HIerarchical SEed Aligner for PacBio data by Khiste and Ilie doi:[10.1186/s12859-017-1953-9](https://doi.org/10.1186/s12859-017-1953-9)
[^fn5]: Assembling large genomes with single-molecule sequencing and locality-sensitive hashing Berlin *et al.* doi:[10.1038/nbt.3238](https:doi.org/10.1038/nbt.3238)
[^fn6]: Minimap2: pairwise alignment for nucleotide sequences by Li *et al.* [arxiv](https://arxiv.org/abs/1708.01492v5)
[^fn7]: Supplementary information of The axolotl genome and the evolution of key tissue formation regulators by Nowoshilow *et al.* doi:[10.1038/nature25458](https://doi.org/10.1038/nature25458)
