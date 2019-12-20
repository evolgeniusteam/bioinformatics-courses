Run BLAST locally
================

## prerequisites

  - Operating Systems should be Linux (Ubuntu or similar distributions) or MacOS
  - Basic knowledge about Linux OS

## for window users

Please consult the NCBI official tutorial for details: https://www.ncbi.nlm.nih.gov/books/NBK52637/.

## 1\. Download and install BLAST

### 1.1 install on Ubuntu

Run the following command in Ubuntu terminal：

``` sh
blastp
```

If the following message shows up：

``` sh
Command 'blastp' not found, but can be installed with:

apt install ncbi-blast+
```

please use the following command to install BLAST tools:

``` sh
sudo apt install ncbi-blast+
```

**NOTE** root or superuser priviledge is required

if BLAST has been installed, you will see：

``` sh
BLAST query/options error: Either a BLAST database or subject sequence(s) must be specified
Please refer to the BLAST+ user manual.
```


### 1.2 install on MacOS

  - go to the NCBI BLAST FTP website：
    <ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/>

  - choose the latest version for your OS to download. Usually, the file name should be like: `ncbi-blast-#####+-x64-macosx.tar.gz`. In which
    `#####` is the version ID，e.g. `2.9.0`。

  - download to local folder, unzip the file

  - enter terminal, use the following command to copy executables to `/usr/local/bin`:

``` sh
cp /path/to/NCBI_blast_folder/bin/* /usr/local/bin
```

please replace `/path/to/NCBI_blast_folder` with the actual folder in which the BLAST executables were unzipped to, e.g.
`/Users/wchen/Downloads/ncbi-blast-2.9.0+`.

  - then you can run the following command at anywhere in the ```terminal``` app:

``` sh
blastp -h
```



## 2\. prepare the database file and create index files

We will use all mouse `protein coding genes` as the database.

  - first, go to the NCBI FTP site : <ftp://ftp.ncbi.nlm.nih.gov/refseq/M_musculus/mRNA_Prot/>

  - find and download `mouse.1.protein.faa.gz`

  - unzip it, and get file: `mouse.1.protein.faa`

  - go to the folder where `mouse.1.protein.faa` can be found in  `terminal`

  - run the following command to create index:

``` sh
makeblastdb -in mouse.1.protein.faa -dbtype prot
```

`makeblastdb` is the tool to create index, it has two parameters in this example:

  - `-in` ： in put file in  `fasta` format

  - `-dbtype` ：sequence type； `prot` : protein， `nucl`  : nucleotide

use command: `makeblastdb -help` to get all options/parameters and their respective usages

If run successfully, the following new files will be generated:

``` sh
mouse.1.protein.faa.phr
mouse.1.protein.faa.psq
mouse.1.protein.faa.pin
```

## 3\. get `query` sequence

We will search and download protein sequence for the `brca1` gene as the `query`; the `query` file may contain multiple sequences.

  - go to the NCBI protein database ： <https://www.ncbi.nlm.nih.gov/protein/>

  - enter in the search box:  `human[organism] brca1`

  - choose the first entry, e.g.: <https://www.ncbi.nlm.nih.gov/protein/AAC37594.1>

  - download its sequence in  `fasta` format, and save to file `human_brca1.faa` in the same folder with  `mouse.1.protein.faa`;
     。

  - double check the file contents by using `more human_brca1.faa`.

## 4\. run `blast` and check the output

use the following command to  `blast` `human brca1` against all mouse protein sequences:

``` sh
blastp -query human_brca1.faa \
    -db mouse.1.protein.faa \
    -out hm_brca1_blp_mm_GRCm38.out.txt \
    -evalue 1e-5
```

the parameters are：

``` sh
-query  : input file
-db     : database file, it should be properly indexed
-out    : output format
-evalue : e-value cutoff
```

use `blastp -help` to check other options/parameters;

please note that `blast` can save the search results in **19种** formats, users can use `-outfmt` to specify the output format. Formats `0, 6, 7` are commonly used. By default its output format is `0`.

Use output format **7** (Tabular with comment lines)：

``` sh
blastp -query human_brca1.faa \
    -db mouse.1.protein.faa \
    -out hm_brca1_blp_mm_GRCm38.outfmt7.txt \
    -outfmt 7 \
    -evalue 1e-5
```

please compare the outputs of the two runs:
``` sh
hm_brca1_blp_mm_GRCm38.outfmt7.txt
hm_brca1_blp_mm_GRCm38.out.txt
```

## 5\. practice

Use `blast` to find homologous sequences of human brca2 protein in all gorila proteins:

  - download the protein sequence of `human brca2` gene,

  - download all protein sequences of **gorila**,

  - do blast and use `evalue <= 1e-5` as cutoff

Key points：

  - where to download sequences for gorila?

  - how to create index files ?

Extended questions：

  - how many tools are in the `blast` tool kit?
      - `blastp`
      - …
  - how to choose the correct tool according to the molecular types (protein or nucleotide) of the sequences in the input and database files?
