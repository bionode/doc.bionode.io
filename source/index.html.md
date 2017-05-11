---
title: Bionode

language_tabs:
  - shell
  - javascript

toc_footers:
  - <a href='https://gitter.im/bionode/bionode'>Chat room</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - ncbidb

search: true
---

# Introduction

Welcome to the Bionode documentation! Here we document only some of the currently more stable modules. If the module you are looking for is not here, please check for its [GitHub repository](https://github.com/search?q=org:bionode+topic:tool) in the list below and read the README.md file.

[**Bionode modules list**](https://github.com/search?q=org:bionode+topic:tool)

Bionode modules can be used as command line tools or JavaScript libraries! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Installation

Make sure you have the latest [Node.JS installed](https://nodejs.org/en/). You can then install each module with npm (see shell examples on the right).

```shell
# Install module as command line tool available in PATH
# using npm -g option
npm install bionode-ncbi -g

# Install locally in the node_modules folder of a project
# for usage as a library. See the JavaScript tab.
npm install bionode-ncbi
```

# bionode-ncbi

## General usage

```shell
bionode-ncbi <command> [arguments] --limit [num] --pretty

## Options
--stdin, -s         Read STDIN  
--limit, -l         Limit number of results  
--throughput, -t    Number of items per API request  
--pretty, -p        Print human readable output instead of NDJSON  
export DEBUG='*'    Debug mode

# Display CLI help
bionode-ncbi --help
```

```javascript
var ncbi = require('bionode-ncbi')

// Callback pattern
ncbi.search('genome', 'human', callback)

// Event pattern
ncbi.search('genome', 'human').on('data', console.log)

// Pipe pattern
var JSONStream = require('JSONStream')

ncbi.search('genome', 'human')
.pipe(JSONStream.stringify())
.pipe(process.stdout)
```

## Search

Takes a database name and a query term. Returns the metadata.

For a list of NCBI database that can be used, see this [documentation's appendix](#bionode-ncbi-database)

### Usage

`search <db> [term]`

Parameter | Default | Description
--------- | ------- | -----------
db | none | [One of these](#bionode-ncbi-database)
term | none | Species, dataset ID, etc

```shell
bionode-ncbi search taxonomy 'solenopsis invicta' --limit 1 --pretty
```

```javascript
ncbi.search('taxonomy', ''solenopsis invicta'').on('data', console.log)
```

```json
{
  "uid": "13686",
  "status": "active",
  "rank": "species",
  "division": "ants",
  "scientificname": "Solenopsis invicta",
  "commonname": "red fire ant",
  "taxid": 13686,
  "akataxid": "",
  "genus": "Solenopsis",
  "species": "invicta",
  "subsp": "",
  "modificationdate": "2015/09/16 00:00",
  "genbankdivision": "Invertebrates"
}
```

```javascript
// Arguments can be passed as an object instead:
ncbi.search({ db: 'sra', term: 'solenopsis' })
.on('data', console.log)
```

```javascript
// Advanced options can be passed using the previous syntax:
var options = {
  db: 'assembly', // database to search
  term: 'human',  // optional term for search
  limit: 500,     // optional limit of NCBI results
  throughput: 100 // optional number of items per request
}
```

```javascript
// The search term can also be passed with write:
var search = ncbi.search('sra').on('data', console.log)
search.write('solenopsis')
```

```javascript
// Or piped, for example, from a file:
var split = require('split')

fs.createReadStream('searchTerms.txt')
.pipe(split())
.pipe(search)
```

## Fetch

Takes a database name and a query term. Returns the data.

### Usage

`fetch <db> [term]`

Parameter | Default | Description
--------- | ------- | -----------
db | none | [One of these](#bionode-ncbi-database)
term | none | Species, dataset ID, etc


```shell
bionode-ncbi fetch nucest p53 -l 1 --pretty
```

```javascript
ncbi.fetch('nucest', 'p53').on('data', console.log)
```

```json
{
  "id": "JZ923713.1 clone 186 Pelteobagrus fulvidraco spleen cDNA library Tachysurus fulvidraco cDNA similar to p53, mRNA sequence",
  "seq": "ACTCCACAACTTCACCCTGCACTTCCAGAAGTCTAGTACGGCCAAATCAGTCACCTGCACGTACTCCCCGGAGCTGAATAAACTCTTCTGTCAGTTAGCTAAGACGTGCCCTGTGCTCATGGCAGTGAGTTTTTCTCCACCACATGGTTCTGTGCTCAGAGCCACTGCTGTGT"
}
```

```javascript
// With advanced parameters for sequence databases (all are optional):
var opts = {
   db: 'nucest',
   term: 'guillardia_theta',
   strand: 1,
   complexity: 4
 }
ncbi.fetch(opts).on('data', console.log)
```

For some databases there are multiple return types. A default one will be chosen automatically, however it is possible to specify this via the rettype option. The NCBI website provides a list of databasese supported by efetch here: [http://www.ncbi.nlm.nih.gov/books/NBK25497/table/chapter2.T._entrez_unique_identifiers_ui/?report=objectonly] (http://www.ncbi.nlm.nih.gov/books/NBK25497/table/chapter2.T._entrez_unique_identifiers_ui/?report=objectonly)

## URLs

Takes either `sra` or `assembly` database name and query term. Returns URLs of datasets.

### Usage

`urls <dlsource> [term]`

```shell
bionode-ncbi urls assembly human -l 1 -p
```

```javascript
ncbi.urls('assembly', 'human').on('data', console.log)
```

```json
{
  "uid": "1075781",
  "assembly_report": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_assembly_report.txt"
  },
  "assembly_stats": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_assembly_stats.txt"
  },
  "cds_from_genomic": {
    "fna": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_cds_from_genomic.fna.gz"
  },
  "feature_table": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_feature_table.txt.gz"
  },
  "genomic": {
    "fna": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_001856865.2_ASM185686v2_genomic.fna.gz",
    "gbff": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_001856865.2_ASM185686v2_genomic.gbff.gz",
    "gff": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_genomic.gff.gz"
  },
  "protein": {
    "faa": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_001856865.2_ASM185686v2_protein.faa.gz",
    "gpff": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_0018568
65.2_ASM185686v2_protein.gpff.gz"
  },
  "rna_from_genomic": {
    "fna": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_00185686
5.2_ASM185686v2_rna_from_genomic.fna.gz"
  },
  "wgsmaster": {
    "gbff": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_0018568
65.2_ASM185686v2_wgsmaster.gbff.gz"
  },
  "README": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/README.txt"
  },
  "annotation_hashes": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/annotation_hashes.txt"
  },
  "assembly_status": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/assembly_status.txt"
  },
  "md5checksums": {
    "txt": "http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/md5checksums.txt"
  }
}
```

```shell
# The following examples requires a json parser
bionode-ncbi urls assembly human -l 1 -p | json genomic.fna
# Returns: http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/856/865/GCA_001856865.2_ASM185686v2/GCA_001856865.2_ASM185686v2_genomic.fna.gz
```

## Download

Takes either sra or assembly db name and query term. Downloads the corresponding SRA or assembly (genomic.fna) file into a folder named after the unique ID (UID).

### Usage

`download <dlsource> [term]`

```shell
bionode-ncbi download assembly 'solenopsis invicta'
```

```javascript
ncbi.download('assembly', 'solenopsis invicta').on('data', console.log)
.on('end', function(path) { console.log('File saved at ' + path) }
```

```json
{"uid":"244018","url":"http://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/188/075/GCF_000188075.1_Si_gnG/GCF_000188075.1_Si_gnG_genomic.fna.gz","path":"244018/GCF_000188075.1_Si_gnG_genomic.fna.gz","status":"downloading","total":557056,"progress":0.4775266121244368,"speed":131072}
```

```shell
bionode-ncbi download assembly 'solenopsis invicta' --pretty
# Downloading GCF_000188075.1_Si_gnG_genomic.fna.gz
# [>                                            ] 0.7% of 116.65 MB (192.51 kB/s)
```

## Link

Returns a unique ID (UID) from a destination database linked to another UID from a source database.

### Usage

`link <srcDB> <destDB> [srcUID]`

```shell
bionode-ncbi link assembly bioproject 244018 --pretty
```

```javascript
ncbi.link('assembly', 'bioproject', 244018).on('data', console.log)
```

```json
{
  "srcDB": "assembly",
  "destDB": "bioproject",
  "srcUID": "244018",
  "destUIDs": [
    "268798",
    "49629"
  ]
}
```

## Expand

Takes a property (e.g. biosample) and an optional destination property
(e.g. sample) and looks for a field named property+id (e.g. biosampleid)
in the Streamed object. Then it will do a `ncbi.search` for that id and save the
result under Streamed object.property.

### Usage

`expand <property> [destProperty]`

```shell
bionode-ncbi search genome 'solenopsis invicta' -l 1 | \
bionode-ncbi expand tax -s --pretty
```

```javascript
ncbi.search('genome', 'solenopsis invicta').pipe(ncbi.expand('tax'))
```

```json
{
  "uid": "2938",
  "organism_name": "Solenopsis invicta",
  "organism_kingdom": "Eukaryota",
  "organism_group": "",
  "organism_subgroup": "Insects",
  "defline": "Solenopsis invicta overview",
  "projectid": 49663,
  "project_accession": "PRJNA49663",
  "status": "Draft",
  "number_of_chromosomes": "0",
  "number_of_plasmids": "0",
  "number_of_organelles": "1",
  "assembly_name": "Si_gnG",
  "assembly_accession": "GCA_000188075.1",
  "assemblyid": 244018,
  "create_date": "2011/02/03 00:00",
  "options": "",
  "weight": "",
  "chromosome_assemblies": "0",
  "scaffold_assemblies": "1",
  "sra_genomes": "0",
  "taxid": 13686,
  "tax": {
    "uid": "13686",
    "status": "active",
    "rank": "species",
    "division": "ants",
    "scientificname": "Solenopsis invicta",
    "commonname": "red fire ant",
    "taxid": 13686,
    "akataxid": "",
    "genus": "Solenopsis",
    "species": "invicta",
    "subsp": "",
    "modificationdate": "2015/09/16 00:00",
    "genbankdivision": "Invertebrates"
  }
}
```

## Plink

Similar to Link but takes the `srcUID` from a property of the Streamed object
and attaches the result to a property with the name of the destination DB.

### Usage

`bionode-ncbi plink <property> <destDB>`

```shell
bionode-ncbi search genome 'solenopsis invicta' -l 1 | \
bionode-ncbi expand tax -s | \
bionode-ncbi plink tax sra -s --pretty
```

```javascript
ncbi.search('genome', 'solenopsis invicta')
.pipe(ncbi.expand('tax'))
.pipe(ncbi.plink('tax', 'sra')
```

```json
{ "uid":"2938",
  "organism_name":"Solenopsis invicta",
  "organism_kingdom":"Eukaryota",
  "organism_group":"",
  "organism_subgroup":"Insects",
  "defline":"Solenopsis invicta overview",
  "projectid":49663,
  "project_accession":"PRJNA49663",
  "status":"Draft",
  "number_of_chromosomes":"0",
  "number_of_plasmids":"0",
  "number_of_organelles":"1",
  "assembly_name":"Si_gnG",
  "assembly_accession":"GCA_000188075.1",
  "assemblyid":244018,
  "create_date":"2011/02/03 00:00",
  "options":"",
  "weight":"",
  "chromosome_assemblies":"0",
  "scaffold_assemblies":"1",
  "sra_genomes":"0",
  "taxid":13686,
  "tax":{"uid":"13686",
  "status":"active",
  "rank":"species",
  "division":"ants",
  "scientificname":"Solenopsis invicta",
  "commonname":"red fire ant",
  "taxid":13686,
  "akataxid":"",
  "genus":"Solenopsis",
  "species":"invicta",
  "subsp":"",
  "modificationdate":"2015/09/16 00:00",
  "genbankdivision":"Invertebrates"},
  "sraid":["2209130","2209129","2209128","2209127","2209126","2209125","2209124","2209123","2209122","2209121","2209120","2209119","1094137","1094136","1094135","280243","280116","280115","280114","280113","280112","280111","280110","280109","280108","280107","280099","280098","280097","279869","279868","279867","279866","279865","279864","279863","279040","278922","278818","278816","278808","278807","278806","278805","278802","225471","225470","225469","225468","25256","25255","25254","25253","25252","25251","24418","23953","23952","23951","23920","23919","23918","23917","23914","23912","23468","23459","23457"]}
```

# bionode-fasta

Streamable FASTA parser.

## Usage

```shell
# bionode-fasta [options] [input file] [output file]
bionode-fasta input.fasta.gz output.json
# You can also use fasta files compressed with gzip
# If no output is provided, the result will be printed to stdout
# Options: -p, --path: Includes the path of the original file as a property of the output objects
```

``` javascript
// Returns a Writable Stream that parses a FASTA content Buffer
// into a JSON Buffer

var fasta = require('bionode-fasta')

fs.createReadStream('./input.fasta')
.pipe(fasta())
.pipe(process.stdout)
```

```javascript
// Can also parse content from filenames Strings
// streamed to it

fs.createReadStream('./fasta-list.txt')
.pipe(split())
.pipe(fasta({filenameMode: true}))
.pipe(process.stdout)
```

```json
{ "id": "contig1", "seq": "AGTCATGACTGACGTACGCATG" }
{ "id": "contig2", "seq": "ATGTACGTACTGCATGC" }
```

```shell
bionode-fasta input.fasta.gz output.json --path
```

```javascript
// When filenames are Streamed like in the previous example,
// or passed directly to the parser Stream, they can be added
// to the output Objects
fasta({includePath: true}, './input.fasta')
.pipe(process.stdout)
```

```json
{ "id": "contig1",
  "seq": "AGTCATGACTGACGTACGCATG",
  "path": "./input.fasta" }
```

```javascript
// The output from the parser can also be available
// as Objects instead of Buffers

fasta({objectMode: true}, './input.fasta')
.on('data', console.log)
```

```javascript
// Shortcut version of the previous example
fasta.obj('./input.fasta').on('data', console.log)
```

```javascript
// Callback style can also be used, however they might
// not be the best for large files
fasta.obj('./input.fasta', function(data) {
  console.log(data)
})
```

# bionode-seq

Module for DNA, RNA and protein sequences manipulation

## Usage

This method currently only works as a JavaScript library and doesn't provide a CLI interface (see [issue #5](https://github.com/bionode/bionode-seq/issues/5)).

## checkType

Check sequence type

Takes a sequence string and checks if it’s DNA, RNA or protein. Follows IUPAC notation which allows ambiguous sequence notation. In this case the sequence is labelled as ambiguous nucleotide rather than amino acid sequence.

```javascript
seq.checkType("ATGACCCTGAGAAGAGCACCG");
// Returns: "dna"
seq.checkType("AUGACCCUGAAGGUGAAUGAA");
// Returns: "rna"
seq.checkType("MAYKSGKRPTFFEVFKAHCSDS");
// Returns: "protein"
seq.checkType("AMTGACCCTGAGAAGAGCACCG");
// Returns: "ambiguousDna"
seq.checkType("AMUGACCCUGAAGGUGAAUGAA");
// Returns: "ambiguousRna"
```

Takes a sequence type argument and returns a function to complement bases.

## reverse

Reverse sequence

Takes sequence string and returns the reverse sequence.

```javascript
seq.reverse("ATGACCCTGAAGGTGAA");
// "AAGTGGAAGTCCCAGTA"
```

## complement

(reverse) complement sequence

Takes a sequence string and optional boolean for reverse, and returns its complement.

```javascript
seq.complement("ATGACCCTGAAGGTGAA");
// "TACTGGGACTTCCACTT"
seq.complement("ATGACCCTGAAGGTGAA", true);
// "TTCACCTTCAGGGTCAT"
//Alias
seq.reverseComplement("ATGACCCTGAAGGTGAA");
// "TTCACCTTCAGGGTCAT"
```

Takes a sequence string and returns the reverse complement (syntax sugar).

## getTranscribedBase

Transcribe base

Takes a base character and returns the transcript base.

```javascript
seq.getTranscribedBase("A");
// "U"
seq.getTranscribedBase("T");
// "A"
seq.getTranscribedBase("t");
// "a"
seq.getTranscribedBase("C");
// "G"
```

## getTranslatedAA

Get codon amino acid

Takes an RNA codon and returns the translated amino acid.

```javascript
seq.getTranslatedAA("AUG");
// "M"
seq.getTranslatedAA("GCU");
// "A"
seq.getTranslatedAA("CUU");
// "L"
```

## removeIntrons

Remove introns

Take a sequence and an array of exonsRanges and removes them.

```javascript
seq.removeIntrons("ATGACCCTGAAGGTGAATGACAG", [[1, 8]]);
// "TGACCCT"
seq.removeIntrons("ATGACCCTGAAGGTGAATGACAG", [[2, 9], [12, 20]]);
// "GACCCTGGTGAATGA"
```

## transcribe

Transcribe sequence

Takes a sequence string and returns the transcribed sequence (dna <-> rna). If an array of exons is given, the introns will be removed from the sequence.

```javascript
seq.transcribe("ATGACCCTGAAGGTGAA");
// "AUGACCCUGAAGGUGAA"
seq.transcribe("AUGACCCUGAAGGUGAA"); //reverse
// "ATGACCCTGAAGGTGAA"
```

## translate

Translate sequence

Takes a DNA or RNA sequence and translates it to protein If an array of exons is given, the introns will be removed from the sequence.

```javascript
seq.translate("ATGACCCTGAAGGTGAATGACAGGAAGCCCAAC"); //dna
// "MTLKVNDRKPN"
seq.translate("AUGACCCUGAAGGUGAAUGACAGGAAGCCCAAC"); //rna
// "MTLKVNDRKPN"
seq.translate("ATGACCCTGAAGGTGAATGACAGGAAGCC", [[3, 21]]);
// "LKVND"
```

## reverseExons

Reverse exons

Takes an array of exons and the length of the reference and returns inverted coordinates.

```javascript
seq.reverseExons([[2,8]], 20);
// [ [ 12, 18 ] ]
seq.reverseExons([[10,45], [65,105]], 180);
// [ [ 135, 170 ], [ 75, 115 ] ]
```

## checkCanonicalTranslationStartSite

Find non-canonical translation start site

Takes a sequence and returns boolean for canonical translation start site.

```javascript
seq.checkCanonicalTranslationStartSite("ATGACCCTGAAGGT");
// true
seq.checkCanonicalTranslationStartSite("AATGACCCTGAAGGT");
// false
```

## getReadingFrames

Get reading frames

Takes a sequence and returns an array with the six possible Reading Frames (+1, +2, +3, -1, -2, -3).

```javascript
seq.getReadingFrames("ATGACCCTGAAGGTGAATGACAGGAAGCCCAAC");
// [ 'ATGACCCTGAAGGTGAATGACAGGAAGCCCAAC',
//   'TGACCCTGAAGGTGAATGACAGGAAGCCCAAC',
//   'GACCCTGAAGGTGAATGACAGGAAGCCCAAC',
//   'GTTGGGCTTCCTGTCATTCACCTTCAGGGTCAT',
//   'TTGGGCTTCCTGTCATTCACCTTCAGGGTCAT',
//   'TGGGCTTCCTGTCATTCACCTTCAGGGTCAT' ]
```

## getOpenReadingFrames

Get open reading frames

Takes a Reading Frame sequence and returns an array of Open Reading Frames.

```javascript
seq.getOpenReadingFrames("ATGACCCTGAAGGTGAATGACAGGAAGCCCAAC");
// [ 'ATGACCCTGAAGGTGAATGACAGGAAGCCCAAC' ]
seq.getOpenReadingFrames("AUGACCCUGAAGGUGAAUGACAGGAAGCCCAAC");
// [ 'AUGACCCUGAAGGUGAAUGACAGGAAGCCCAAC' ]
seq.getOpenReadingFrames("ATGAGAAGCCCAACATGAGGACTGA");
// [ 'ATGAGAAGCCCAACATGA', 'GGACTGA' ]
```

## getAllOpenReadingFrames

Get all open reading frames

Takes a sequence and returns all Open Reading Frames in the six Reading Frames.

```javascript
seq.getAllOpenReadingFrames("ATGACCCTGAAGGTGAATGACA");
// [ [ 'ATGACCCTGAAGGTGAATGACA' ],
//  [ 'TGA', 'CCCTGA', 'AGGTGA', 'ATGACA' ],
//  [ 'GACCCTGAAGGTGAATGA', 'CA' ],
//  [ 'TGTCATTCACCTTCAGGGTCAT' ],
//  [ 'GTCATTCACCTTCAGGGTCAT' ],
//  [ 'TCATTCACCTTCAGGGTCAT' ] ]
```

## findLongestOpenReadingFrame

Find longest open reading frame

Takes a sequence and returns the longest ORF from all six reading frames and corresponding frame symbol (+1, +2, +3, -1, -2, -3). If a frame symbol is specified, only look for longest ORF on that frame. When sorting ORFs, if there’s a tie, choose the one that starts with start codon Methionine. If there’s still a tie, return one randomly.

```javascript
seq.findLongestOpenReadingFrame("ATGACCCTGAAGGTGAATGACA");
// [ 'ATGACCCTGAAGGTGAATGACA', '+1' ]
seq.findLongestOpenReadingFrame("ATGACCCTGAAGGTGAATGACA", "-1");
// "TGTCATTCACCTTCAGGGTCAT"
```
