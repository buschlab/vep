# Containerized Variant Effect Predictor (VEP) + Cache
[![Twitter](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?hashtags=Ensembl,VEP,Singularity,Docker&url=https://github.com/buschlab/vep)

&nbsp;+ [Introduction](#Introduction) \
&nbsp;+ [Building image with Singularity](#Building-image-with-Singularity) \
&nbsp;+ [Run VEP](#Run-VEP) \
&nbsp;&nbsp;&nbsp;&nbsp;|-- [More options](#More-options) \
&nbsp;&nbsp;&nbsp;&nbsp;|-- [Examples](#Examples) \
&nbsp;+ [Post-processing](#Post-processing) \
&nbsp;&nbsp;&nbsp;&nbsp;|-- [Split VEP](#Split-VEP) \
&nbsp;&nbsp;&nbsp;&nbsp;|-- [Filtering by VEP annotations](#Filtering-by-VEP-annotations) \
&nbsp;+ [VEP plugins](#VEP-plugins) \
&nbsp;+ [Build & run VEP with Docker](#Build-and-run-VEP-with-Docker) \
&nbsp;+ [Acknowledgements](#Acknowledgments)


## Introduction

This repository was forked from https://github.com/matmu/vep, who established the workflow. Old releases (prior v106.1) are provided by https://github.com/matmu/vep. Future releases will be publish on https://github.com/buschlab/vep.

This documentation describes the usage of the Docker image at https://hub.docker.com/r/buschlab/vep which contains the bioinformatics tool **Ensembl Variant effect predictor (VEP)** for annotating genetic variants. The image comes with

* Merged cache including RefSeq and Ensembl transcripts (VEP parameter --merged required)
* Reference genome and index
* Plugins (annotation data is not included)


## Available versions
**Human**

[![106.1-GRCh38](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCh38.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCh38.yml) 

[![106.1-GRCh37](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCh37.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCh37.yml)

[![109-GRCh38](https://github.com/buschlab/vep/actions/workflows/docker.109-GRCh38.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.109-GRCh38.yml)

**Mouse**

[![102.0-GRCm38](https://github.com/buschlab/vep/actions/workflows/docker.102-GRCm38.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.102-GRCm38.yml) 

[![106.1-GRCm39](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCm39.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.106-GRCm39.yml) 

[![109-GRCm39](https://github.com/buschlab/vep/actions/workflows/docker.109-GRCm39.yml/badge.svg)](https://github.com/buschlab/vep/actions/workflows/docker.109-GRCm39.yml)

You can also visit https://hub.docker.com/r/matmu/vep/tags to get a list of available versions (prior v106.1) and https://hub.docker.com/r/buschlab/vep/tags for newer releases.

**Note:** If you require a container for a species not mentioned above, feel free to contact us or create an issue.


## Build image with Singularity
```bash
singularity build vep.<version>.simg docker://buschlab/vep:<version>
```

`<version>` is a tag representing the Ensembl version and the species + version of the reference genome. 


## Run VEP
To run VEP execute
```bash
singularity exec vep.<version>.simg vep [options]
```
whereby `<version>` is replaced by a respective version (see above), e.g. `106-CRCh38`. It is essential to add the VEP option `--merged` when using an image with merged Ensembl/Refseq cache. For species except homo sapiens, also the parameter `--species` (e.g. `--species mus_musculus`), has to be set as well.


### More options
The options for base cache/plugin directories, species and assembly are set to the right values by default and do not need to be set by the user.

Visit http://www.ensembl.org/info/docs/tools/vep/script/vep_options.html for detailed information about all VEP options. Detailed information about **input/output formats** can be found at https://www.ensembl.org/info/docs/tools/vep/vep_formats.html#defaultout. 


### Examples

#### Minimum (output format: compressed tab delimited)
```bash
singularity exec vep.106-GRCh38-merged.simg vep --dir /opt/vep/.vep --merged --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.txt.gz --tab --compress_output bgzip
```

```bash
singularity exec vep.106-GRCh38.simg vep --dir /opt/vep/.vep --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.txt.gz --tab --compress_output bgzip
```

```bash
singularity exec vep.106-GRCm39.simg vep --dir /opt/vep/.vep --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.txt.gz --tab --compress_output bgzip -species mus_musculus
```


#### Minimum (output format: compressed vcf)
```bash
singularity exec vep.106-GRCh38.simg vep --dir /opt/vep/.vep --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.vcf.gz --vcf --compress_output bgzip
```

#### Full annotation
```bash
singularity exec vep.106-GRCh38.simg vep --dir /opt/vep/.vep --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.vcf.gz --vcf --compress_output bgzip --everything --nearest symbol        
```

## Post-processing

### Split VEP
There is a plugin for `bcftools` that allows to split VEP annotations as well as sample information in a VCF file and convert it to a text file: http://samtools.github.io/bcftools/howtos/plugin.split-vep.html.


### Filtering by VEP annotations
If you chose to output the VEP annotations as text file, any command line tool (e.g. `awk`) or even `Excel` can be used for filtering the results. For VCF files, the image includes a VEP filtering script which can be executed by
```bash
singularity exec vep.<version>.simg filter_vep [options]
```


#### Options
Visit https://www.ensembl.org/info/docs/tools/vep/script/vep_filter.html for detailed info about available options.


#### Filtering examples

##### Filter for rare variants
```bash
singularity exec vep.<version>.simg filter_vep --input_file <filename>.vcf --output_file <filename>.filtered.vcf --only_matched --filter "(IMPACT is HIGH or IMPACT is MODERATE or IMPACT is LOW) and (BIOTYPE is protein_coding) and ((PolyPhen > 0.446) or (SIFT < 0.05)) and (EUR_AF < 0.001 or gnomAD_NFE_AF < 0.001 or (not EUR_AF and not gnomAD_NFE_AF))" 
```


## VEP plugins
VEP allows several other annotations sources (aka Plugins). Their respective Perl modules are included in the image, the annotation files have to be added seperately, however. The list of plugins as well as instructions on how to download and pre-process the annotation files can be found at: http://www.ensembl.org/info/docs/tools/vep/script/vep_plugins.html.

```bash
singularity exec vep.106-GRCh38-merged.simg vep --dir /opt/vep/.vep --merged --offline --cache --input_file <filename>.vcf[.gz] --output_file <filename>.txt.gz --tab --compress_output bgzip --plugin CADD,/path/to/ALL.TOPMed_freeze5_hg38_dbSNP.tsv.gz
```


## Build and run VEP with Docker
To pull the image and run the container with Docker use 
```
docker run buschlab/vep:<version> vep [options]
```

Unlike Singularity, the directories of **Plugin** annotation files (e.g. `/path/to/dir`) have to be explicitely bound to a target directory (e.g. `/opt/data`) within the container with option `-v`:
```
docker run -v /path/to/dir:/opt/data buschlab/vep:<version> vep [options]
```


## Acknowledgments
This document has been created by **Julia Remes** & **Matthias Munz**, **University of Lübeck**, **Germany** and was updated by **Axel Künstner**, member of **Medical Systems Biology Group** (**University of Lübeck**, **Germany**).

