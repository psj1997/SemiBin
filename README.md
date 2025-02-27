# SemiBin: Metagenomic Binning Using Siamese Neural Networks for short and long reads

Command tool for metagenomic binning with deep learning, handles both short and long reads.

[![BioConda Install](https://img.shields.io/conda/dn/bioconda/semibin.svg?style=flag&label=BioConda%20install)](https://anaconda.org/bioconda/semibin)
[![Test Status](https://github.com/BigDataBiology/SemiBin/actions/workflows/semibin_test.yml/badge.svg)](https://github.com/BigDataBiology/SemiBin/actions/workflows/semibin_test.yml)
[![Documentation Status](https://readthedocs.org/projects/semibin/badge/?version=latest)](https://semibin.readthedocs.io/en/latest/?badge=latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

_CONTACT US_: Please use [GitHub
issues](https://github.com/BigDataBiology/SemiBin/issues) for bug reports and
the [SemiBin users mailing-list](https://groups.google.com/g/semibin-users) for
more open-ended discussions or questions.

If you use this software in a publication please cite:

>  Pan, S.; Zhu, C.; Zhao, XM.; Coelho, LP. [A deep siamese neural network improves metagenome-assembled genomes in microbiome datasets across different environments](https://doi.org/10.1038/s41467-022-29843-y). *Nat Commun* **13,** 2326 (2022). [https://doi.org/10.1038/s41467-022-29843-y](https://doi.org/10.1038/s41467-022-29843-y)

The self-supervised approach and the algorithms used for long-read datasets (as
well as their benchmarking) are described in

> Pan, S.; Zhao, XM; Coelho, LP. [SemiBin2: self-supervised contrastive learning leads to better MAGs for short- and long-read sequencing](https://doi.org/10.1101/2023.01.09.523201). *bioRxiv preprint* 2023.01.09.523201; [https://doi.org/10.1101/2023.01.09.523201](https://doi.org/10.1101/2023.01.09.523201) (Accepted at *Bioinformatics*)

## SemiBin2

The functionality of SemiBin2 is available already since version 1.4!

- To use the self-supervised learning mode, use options `--self-supervised`
- If you are using long-reads, use option `--sequencing-type=long_read`


## Basic usage of SemiBin

A tutorial of running SemiBin from scrath can be found here [SemiBin tutorial](https://github.com/BigDataBiology/SemiBin_tutorial_from_scratch).

Installation:

```bash
conda create -n SemiBin
conda activate SemiBin
conda install -c conda-forge -c bioconda semibin
```

**The inputs** to the SemiBin are contigs (assembled from the reads) and BAM files (reads mapping to the contigs). In [the docs](https://semibin.readthedocs.io/en/latest/generate/) you can see how to generate the inputs starting with a metagenome.

Running with single-sample binning (for example: human gut samples):

```bash
SemiBin single_easy_bin -i contig.fa -b S1.sorted.bam -o output --environment human_gut
```

(if you are using contigs from long-reads, add the `--sequencing-type=long_read` argument).

Running with multi-sample binning:

```bash
SemiBin multi_easy_bin -i contig_whole.fa -b *.sorted.bam -o output
```

**The output** includes the bins in the `output_recluster_bins` directory (including the bin.\*.fa and recluster.\*.fa).


Please find more options and details below and [read the docs](https://semibin.readthedocs.io/en/latest/usage/). 

## Advanced Installation

SemiBin runs on Python 3.7-3.11.

### Bioconda

The simplest mode is shown above.
However, if you want to use SemiBin with GPU (which is faster if you have one available), you need to install PyTorch with GPU support:

```bash
conda create -n SemiBin
conda activate SemiBin
conda install -c conda-forge -c bioconda semibin
conda install -c pytorch-lts pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch-lts
```

_MacOS note_: **you can only install the CPU version of PyTorch in MacOS with `conda` and you need to install from source to take advantage of a GPU** (see [#72](https://github.com/BigDataBiology/SemiBin/issues/72)).
For more information on how to install PyTorch, see [their documentation](https://pytorch.org/get-started/locally/).

### Source

You will need the following dependencies:

- [MMseqs2](https://github.com/soedinglab/MMseqs2)
- [Bedtools](http://bedtools.readthedocs.org/]), [Hmmer](http://hmmer.org/)
- [Prodigal](https://github.com/hyattpd/Prodigal)
- (optionally) [FragGeneScan](https://sourceforge.net/projects/fraggenescan/)
- [Samtools](https://github.com/samtools/samtools)

The easiest way to install the dependencies is with [conda](https://conda.io):

```bash
conda install -c conda-forge -c bioconda mmseqs2=13.45111 # (for GTDB support)
conda install -c bioconda bedtools hmmer prodigal samtools
conda install -c bioconda fraggenescan
```

Once the dependencies are installed, you can install SemiBin by running:

```bash
python setup.py install
```

## Examples of binning

SemiBin runs on single-sample, co-assembly and multi-sample binning.
Here we show the simple modes as an example.
For the details and examples of every SemiBin subcommand, please [read the docs](https://semibin.readthedocs.io/en/latest/usage/).

## Binning assemblies from long reads

Since version 1.4, SemiBin proposes new algorithm (ensemble based DBSCAN algorithm) for binning assemblies from long reads. 
To use it, you can used the subcommands `bin_long` or pass the option  `--sequencing-type=long_read` to the `single_easy_bin` or `multi_easy_bin` subcommands.

## Self-supervised mode

Since version 1.3, SemiBin supports completely self-supervised learning, which bypasses the need to annotate contigs with MMSeqs2.
In benchmarks, self-supervised learning is both faster (4x faster; using only 11% of RAM at peak) and generates 8.3-21.5% more high-quality bins compared to the version tested in the [manuscript](https://www.nature.com/articles/s41467-022-29843-y)
To use it, pass the option `--training-mode=self` to the `single_easy_bin` or `multi_easy_bin` subcommands.


## Easy single/co-assembly binning mode

Single sample and co-assembly are handled the same way by SemiBin.

You will need the following inputs:

1. A contig file (`contig.fa` in the example below)
2. BAM file(s) from mapping short reads to the contigs, sorted (`mapped_reads.sorted.bam` in the example below)

The `single_easy_bin` command can be used to produce results in a single step.

For example:

```bash
SemiBin \
    single_easy_bin \
    --input-fasta contig.fa \
    --input-bam mapped_reads.sorted.bam \
    --environment human_gut \
    --output output
```

Alternatively, you can train a new model for that sample, by not passing in the `--environment` flag:

```bash
SemiBin \
    single_easy_bin \
    --input-fasta contig.fa \
    --input-bam mapped_reads.sorted.bam \
    --output output
```

The following environments are supported:

- `human_gut`
- `dog_gut`
- `ocean`
- `soil`
- `cat_gut`
- `human_oral`
- `mouse_gut`
- `pig_gut`
- `built_environment`
- `wastewater`
- `chicken_caecum` (Contributed by [Florian Plaza Oñate](https://scholar.google.com/citations?hl=zh-CN&user=-gE5y_4AAAAJ&view_op=list_works&sortby=pubdate))
- `global`

The `global` environment can be used if none of the others is appropriate.
Note that training a new model can take a lot of time and disk space.
Some patience will be required.
If you have a lot of samples from the same environment, you can also train a new model from them and reuse it.

## Easy multi-samples binning mode

The `multi_easy_bin` command can be used in multi-samples binning mode:

You will need the following inputs:

1. A combined contig file
2. BAM files from mapping

For every contig, format of the name is `<sample_name>:<contig_name>`, where
`:` is the default separator (it can be changed with the `--separator`
argument). _NOTE_: Make sure the sample names are unique and  the separator
does not introduce confusion when splitting. For example:

```bash
>S1:Contig_1
AGATAATAAAGATAATAATA
>S1:Contig_2
CGAATTTATCTCAAGAACAAGAAAA
>S1:Contig_3
AAAAAGAGAAAATTCAGAATTAGCCAATAAAATA
>S2:Contig_1
AATGATATAATACTTAATA
>S2:Contig_2
AAAATATTAAAGAAATAATGAAAGAAA
>S3:Contig_1
ATAAAGACGATAAAATAATAAAAGCCAAATCCGACAAAGAAAGAACGG
>S3:Contig_2
AATATTTTAGAGAAAGACATAAACAATAAGAAAAGTATT
>S3:Contig_3
CAAATACGAATGATTCTTTATTAGATTATCTTAATAAGAATATC
```

You can use this to get the combined contig:

```bash
SemiBin concatenate_fasta -i contig*.fa -o output
```

If either the sample or the contig names use the default separator (`:`), you will need to change it with the `--separator`,`-s` argument.

After mapping samples (individually) to the combined FASTA file, you can get the results with one line of code:

```bash
SemiBin multi_easy_bin -i concatenated.fa -b *.sorted.bam -o output
```

## Output

The output folder will contain:

1. Features computed from the data and used for training and clustering
2. Saved semi-supervised deep learning model
3. Output bins
4. Table with basic information about each bin
5. Some intermediate files

By default, reconstructed bins are in `output_recluster_bins` directory.

For more details about the output, [read the
docs](https://semibin.readthedocs.io/en/latest/output/).
