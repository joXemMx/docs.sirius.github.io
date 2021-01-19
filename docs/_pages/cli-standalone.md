---
permalink: /cli-standalone/
title: "Standalone CLI tools"
---
Standalone tools provide additional SIRIUS related tasks that do not fit into the SIRIUS identification workflow.
These can e.g. be configuration tasks, file conversion tasks or features that might be helpful for downstream analysis.
These tool cannot be part of a toolchain and have to be executed in a separate command. Each of these tools has its own
help message (`sirius <TOOLNAME> -h`).


## Custom database tool
The `custom-db` tool allows you to import custom structure databases from a `csv/tsv` (tab separated) file with 
structures given in `SMILES` format. Optionally a database `id` and a `name` can be given. 

```
CN1CCCC1C2=C[N+](=CC=C2)[O-]	id-01	Caffein
CN1C=NC2=C1C(=O)N(C(=O)N2C)C	id-03	Nicotin
```

You can import multiple files with compounds as SMILES into one DB. If a given structure can be found in
SIRIUS' internal structure DB then fingerprint is downloaded from there, otherwise it is computed locally on your computer 
which might take some time for many structures.

Note, that we usually use PubChem standardized SMILES to represent the structures for our machine learning methods. 
PubChem standardization is not yet part of this import process. For best possible results we recommend standardizing
your SMILES using the PubChem standardization before importing them, but this step is **not** mandatory.


## Similarity tool
The `similarity` tool allows you to compute different similarity measures between compounds.
It takes a SIRIUS project-space (or any input format SIRIUS can convert into a project such as `ms`, `mgf` or `cef`) 
as input (`sirius -i <INPUT>`) and calculates all against all similarity matrices for the compounds
in the project-space and stores them in the given output directory given by `-d`.

```
sirius -i <project-space> similarity --cosine --ftalign --ftblast <SPECTRA_LIB> --tanimoto -d <OUTPUT>
```

### Cosine Similarity   (`--cosine`)
Computes the cosine similarity of the merged MS/MS between all compounds.
Just the spectra are needed no additional computation have to be performed beforehand.

### Fragmentation Tree alignment Similarity  (`--ftalign`)
Computes the tree alignment score of the top ranked fragmentation tree between all compounds.
To perform this computation the input project-space needs to contain the fragmentation trees from the `formula`/`sirius`
subtool. So the `formula` subtool has to be executed beforehand. The Alignment method is described in
["*Identifying the Unknowns by Aligning Fragmentation Trees*"](https://doi.org/10.1021/ac300304u)

### FT-Blast (`--ftblast`)
Computes fragmentation tree alignments between all compounds in the dataset, incorporating the given fragmentation
tree library as described in ["*Identifying the Unknowns by Aligning Fragmentation Trees*"](https://doi.org/10.1021/ac300304u).
The input project-space needs to contain fragmentation trees computed with the `formula`/`sirius` subtool. 
So the `formula` subtool has to be executed beforehand. The given library (`--ft-blast=<LIB_PATH>`) can either be another
SIRIUS project-space containing fragmentation trees, or a directory containing fragmentation trees in `json` format.

### Tanimoto Similarity (`--tanimoto`)
Computes the tanimoto similarity of the top ranked predicted fingerprints between all compounds.
The input project-space needs to contain the predicted fingerprints from the `structure`/`fingerid`
subtool. So the `formula` and the `structure` subtool have to executed beforehand.
Note that the fingerprints compared are probabilistic. The tanimoto computation for two probabilistic fingerprints 
$F$ and $F'$ of length $n$ is computed as follows:

$$\frac{ \sum_{i=1}^{n} F_i \cdot F'_i } { \sum_{i=1}^{n} 1 - (1 - F_i) \cdot (1 - F'_i) }$$


## Mass Decomposition tool
The `decomp` tool provides the SIRIUS internal ["*Efficient mass decomposition*"](https://doi.org/10.1145/1066677.1066715) 
algorithm as standalone tool to decompose masses with given deviation, ionization, chemical alphabet and chemical filter.

## MGF export tool
The `mgf-export` tool exports the spectra of a given input project-space as `.mgf` for use with other tools like [GNPS](https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp).
The `--quant-table` option allows to export an additional feature quantification table (`csv`),
e.g. to export a SIRIUS project-space for [GNPS Feature Based Molecular Networking](https://ccms-ucsd.github.io/GNPSDocumentation/featurebasedmolecularnetworking/):
```
sirius --input <project-space> --merge-ms2 --quant-table <tale.csv> --output <spectra.mgf>
```
Note, quantification information are only available if the source of the project-space was in `mzml`(`mzxml`).

## Fragmentation tree export tool
The `ftree-export` tool exports the fragmentation trees of a given project-space (`sirius -i <INPUT>`) in
various formats (`json`, `dot`) to a given output directory (`--output <DIR>`).

## Project-space tool
The `project-space` tool Modifies a given project-space (e.g. merging, splitting, filtering, version conversion). 
Read project(s) with `--input`, apply modification and write the result via `--output`. If either only `--input` or 
`--output` is given the modifications will be made in-place.