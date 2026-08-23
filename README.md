# NLR-Finder

*EuchroGene NLR-Finder v2.0, for EuchroGene members.*

NLR-Finder reads a predicted proteome and returns the plant NLR (NB-LRR) disease resistance repertoire. Every protein is scanned with [HMMER3](https://doi.org/10.1371/journal.pcbi.1002195) against 14 curated [Pfam](https://doi.org/10.1093/nar/gkaa913) profiles covering the NB-ARC core, the TIR, Rx-type coiled-coil and RPW8 N-terminal domains, and nine leucine-rich repeat families. Retained domains are laid out in the order they occur along the sequence, and that architecture assigns the class following the [RefPlantNLR](https://doi.org/10.1371/journal.pbio.3001124) scheme: TNL, CNL, RNL, NL, TN, CN, RN or N.

A genome annotation is optional. Supply one and NLR-Finder also writes the full gene models and calls the physical NLR clusters those genes fall into. Optional stages assign the closest experimentally characterised NLR with [DIAMOND](https://doi.org/10.1038/s41592-021-01101-x), report candidate [integrated domains](https://doi.org/10.1186/s12915-016-0228-7), and build an NB-ARC phylogeny with [ClipKIT](https://doi.org/10.1371/journal.pbio.3001007) and [FastTree 2](https://doi.org/10.1371/journal.pone.0009490).

---

## Installation

**0. Install EG_tools**

```bash
wget https://github.com/euchrogene/EG_tools/raw/main/EG_tools
chmod 777 EG_tools
sudo mv EG_tools /usr/bin
```

**1. Remove v1.0 if it is installed**

v2.0 installs alongside v1.0 rather than replacing it: the executable, the Docker image and the EG_tools entry all carry the version, so the two never collide. Remove v1.0 anyway. Its `CNL` calls are wrong (see Notes below), and leaving it on the machine invites someone to run it by habit.

```bash
EG_tools                                                          # confirm what is installed
sudo EG_tools uninstall -t NLR-finder_v.1.0 -i managene7/nlr-finder:v.1.0
```

If v1.0 was installed by hand rather than through EG_tools, remove the executable and the image directly:

```bash
sudo rm -f /usr/bin/NLR-finder_v.1.0 /usr/local/bin/NLR-finder_v.1.0
sudo docker rmi managene7/nlr-finder:v.1.0
```

Keep any v1.0 **results** you have already published or submitted. Do not silently overwrite them: rerun with v2.0 into a new folder and compare, because the class distribution will change (see Notes).

**2. Install v2.0**

```bash
sudo EG_tools install \
  -r https://github.com/euchrogene/NLR-finder.git \
  -d NLR-finder \
  -e NLR-finder_v.2.0 \
  -m "NLR disease resistance repertoire annotation from a proteome"
```

**3. Display installed software**

```bash
EG_tools
```

**4. Show help contents**

```bash
NLR-finder_v.2.0
```

**5. Uninstall v2.0**

```bash
sudo EG_tools uninstall -t NLR-finder_v.2.0 -i managene7/nlr-finder:v.2.0
```

Docker image: `managene7/nlr-finder:v.2.0`

---

## Quick Start

Protein-only run, the quickest way to get the repertoire:

```bash
NLR-finder_v.2.0 \
   -protein Vashei_proteins.fa \
   -species "Vaccinium ashei"
```

Full run with the genome annotation, adding gene models and clusters:

```bash
NLR-finder_v.2.0 \
   -protein Vashei_proteins.fa \
   -gff Vashei_genes.gff3 \
   -species "Vaccinium ashei" \
   -assembly "V.ashei v1.0" \
   -out Vashei_NLR \
   -cores 32
```

Publication run, adding integrated domains and the NB-ARC tree:

```bash
NLR-finder_v.2.0 \
   -protein Vashei_proteins.fa \
   -gff Vashei_genes.gff3 \
   -species "Vaccinium ashei" \
   -integrated_domains true \
   -phylo true \
   -cores 32
```

---

## Inputs

| Input | Required | Notes |
|---|---|---|
| Protein FASTA (`-protein`) | Yes | All predicted proteins, one entry per protein. The identifier is the first whitespace-delimited token of the header. |
| Genome annotation (`-gff`) | No | GFF3 with `ID` and `Parent` attributes. Supplying it turns on gene model extraction and cluster analysis. An embedded `##FASTA` block is ignored. |
| Organism (`-species`) | No | Written into the report title and the Methods section. |
| Assembly (`-assembly`) | No | Written into the Methods section. |

Protein identifiers are matched to the annotation directly and, failing that, after stripping a transcript suffix such as `.t1`, `.p1`, `.1`, `-mRNA-1`, `-RA` or a trailing `_1`. Identifiers that still fail to match are counted in the report and listed in `tmp/unmatched_protein_ids.txt` when `-keep_tmp true` is set.

---

## Outputs

```
NLR_Finder_<date>/
├── NLR_proteins.fa              all NLR proteins, class and architecture in the header
├── NLR_nbarc_domains.fa         NB-ARC domain sequences, one per NLR, ready for phylogenetics
├── by_class/
│   ├── NLR_TNL.fa               TIR-NB-ARC-LRR
│   ├── NLR_CNL.fa               CC (Rx_N type)-NB-ARC-LRR
│   ├── NLR_RNL.fa               RPW8-NB-ARC-LRR
│   ├── NLR_NL.fa                NB-ARC-LRR, no identifiable N-terminal domain
│   └── NLR_<TN|CN|RN|N>.fa      LRR-truncated and NB-ARC-only classes
├── NLR_classification.tsv       class, architecture, completeness, coordinates, E-values
├── NLR_domain_architecture.tsv  every retained domain hit with its coordinates
├── NLR_integrated_domains.tsv   candidate integrated domains        [-integrated_domains true]
├── NLR_refplantnlr_hits.tsv     closest characterised NLR per protein    [-add_reference true]
├── NLR_genes.gff3               NLR gene models, full feature hierarchy           [-gff]
├── NLR_clusters.tsv             physical NLR clusters with member genes           [-gff]
├── phylogeny/
│   ├── NLR_nbarc.aln.fasta      hmmalign alignment of the NB-ARC domains   [-phylo true]
│   ├── NLR_nbarc.trimmed.fasta  ClipKIT smart-gap trimmed alignment        [-phylo true]
│   └── NLR_nbarc.tree.nwk       FastTree tree, SH-like local support       [-phylo true]
├── hmmer/                       raw HMMER domain table and search log
├── ANALYSIS_REPORT.html         publication report, Methods section included
├── RUN_SPEC.json                parameters, input checksums, worker checksums
├── tool_versions.json           versions queried from the tools at run time
└── nlr_finder_summary.json      machine-readable run summary
```

`NLR_classification.tsv` is the table every other output is keyed on; use it for downstream work.

---

## Key Options

| Option | Default | Purpose |
|---|---|---|
| `-protein` | required | Protein FASTA of all predicted proteins |
| `-gff` | none | Genome annotation; enables gene models and clusters |
| `-out` | `NLR_Finder_<date>` | Results folder |
| `-species` | none | Organism name for the report and Methods |
| `-assembly` | none | Assembly name or version for the Methods |
| `-cores` | `16` | CPU threads |
| `-evalue` | `1e-5` | Domain i-E-value cutoff for NB-ARC, TIR, RPW8 and Rx_N |
| `-lrr_evalue` | `1e-3` | Domain i-E-value cutoff for the LRR profiles |
| `-min_nbarc_cov` | `0.5` | Least fraction of the NB-ARC profile a hit must cover |
| `-integrated_domains` | `false` | Scan against full Pfam-A and report NLR-ID candidates |
| `-id_evalue` | `1e-5` | i-E-value cutoff for integrated domains |
| `-add_reference` | `true` | Assign the closest RefPlantNLR homolog |
| `-ref_evalue` | `1e-10` | E-value cutoff for the RefPlantNLR search |
| `-cluster_gap` | `200000` | Largest gap in bp still counted as one cluster |
| `-cluster_min` | `2` | Least number of genes in a cluster |
| `-phylo` | `false` | Build an NB-ARC domain phylogeny |
| `-phylo_model` | `lg` | Amino acid substitution model: `lg`, `wag` or `jtt` |
| `-keep_tmp` | `false` | Keep the container scratch folder |

`-output` and `-threads` from v1.0 still work as aliases of `-out` and `-cores`.

---

## Building the image

The published image already contains everything. Build it yourself only if you need a different reference set or the larger Pfam build.

```bash
sudo docker build -t managene7/nlr-finder:v.2.0 .
```

| Build argument | Default | What it does |
|---|---|---|
| `REQUIRE_FULL_PFAM` | `no` | `yes` keeps the full Pfam-A library, which `-integrated_domains true` needs. Adds roughly 1.5 GB. |
| `REQUIRE_REFPLANTNLR` | `yes` | `no` builds without the reference NLR set. Runs then need `-add_reference false`. |
| `REFPLANTNLR_URL` | Zenodo record 3936022, supplemental dataset 1 | Source of the reference FASTA. |
| `REFPLANTNLR_VERSION` | `v.20200528_415` | Label written to `VERSION.txt` and quoted in the report Methods. |
| `REFPLANTNLR_SEQ_COUNT` | `415` | Expected sequence count. `0` skips the check. |
| `REFPLANTNLR_MD5` | md5 of the default file | Expected checksum. Empty skips the check. |

The build verifies the reference set by both sequence count and md5, so a moved URL, a truncated transfer or an HTML error page served in place of the FASTA all fail the build rather than shipping quietly.

**Shipping a different RefPlantNLR release.** The image defaults to `v.20200528_415`, the release archived at Zenodo record 3936022. The RefPlantNLR paper describes the later `v.20210712_481`, distributed as supplemental dataset S1 of the article. To ship that instead, pass all four arguments together:

```bash
sudo docker build -t managene7/nlr-finder:v.2.0 . \
  --build-arg REFPLANTNLR_URL="<url of the release you want>" \
  --build-arg REFPLANTNLR_VERSION="v.20210712_481" \
  --build-arg REFPLANTNLR_SEQ_COUNT=481 \
  --build-arg REFPLANTNLR_MD5=""
```

Whichever release is built in, the version label is read from the running container and appears in the report Methods, so the results always say which reference set produced them.

**If the machine cannot reach Zenodo.** Build without the reference set and run with `-add_reference false`; every other stage is unaffected.

```bash
sudo docker build -t managene7/nlr-finder:v.2.0 . --build-arg REQUIRE_REFPLANTNLR=no
```

---

## Notes

**Results from v1.0 are not comparable.** v1.0 mapped `Rx_N` to the wrong Pfam accession, so no Rx-type coiled coil was ever detected and every NB-ARC + LRR protein without a TIR was labelled `CNL`. Those `CNL` counts were really `CNL + NL` combined, and the true `CN` and `RN` classes were absent. `LRR_8` also carried the wrong accession in the v1.0 output tables. Rerun anything you intend to publish; do not carry v1.0 class counts into a v2.0 figure or table.

**Class assignment is architecture-based.** The N-terminal domain sets the prefix and a detected LRR solenoid downstream of the NB-ARC core adds the trailing L. `CNL` therefore means an Rx-type coiled coil was detected by profile, not that a coiled coil was predicted by secondary structure. Proteins with an NB-ARC core and an LRR but no detectable N-terminal domain are reported as `NL` rather than being forced into `CNL`, which is what v1.0 did.

**LRR profiles get their own threshold.** Individual LRR repeats are short and score weakly, so they are filtered at `-lrr_evalue` rather than `-evalue`, and adjacent hits within 30 residues are merged into single LRR regions. Tightening `-evalue` alone will not remove weak LRR calls; tighten `-lrr_evalue` for that.

**`-min_nbarc_cov` is the main sensitivity dial.** It rejects fragments that carry only a corner of the NB-ARC profile. Raise it on a fragmented annotation, lower it when you are deliberately hunting truncated or pseudogenised NLRs. HMMER often reports one NB-ARC as two alignment fragments when the protein carries an insertion, so fragments within 150 residues of each other are treated as one domain and coverage is measured over their combined profile span. The `nbarc_fragments` column records how many fragments were merged.

**`-integrated_domains true` needs the larger image.** The default build discards full Pfam-A to stay small. Without it the run stops with a message telling you to rebuild with `--build-arg REQUIRE_FULL_PFAM=yes`. Integrated domains are candidate decoy or sensor domains, not confirmed integrations.

**Input proteomes are sanitised before the search.** A trailing `*` is removed and any character outside the amino acid alphabet is replaced by `X` in place, so residue numbering still matches your file. Duplicate identifiers keep their first occurrence. Both counts are printed at the start of the run. Feeding a nucleotide FASTA to `-protein` is caught before the container starts.

**Annotations without `gene` features still work.** Some predictors emit GFF3 whose top-level feature is `mRNA` or `transcript`. When no `gene` row is found, coordinates are taken from the root feature of each match instead, the run log says so, and `gff3_root_feature` in the summary JSON records which type was used.

**Cluster calls depend on assembly contiguity.** `-cluster_gap` is applied within one sequence, so NLRs split across scaffolds in a fragmented assembly will be counted as singletons even when they are genuinely clustered in the genome.

**The tree is approximate.** FastTree branch support values are SH-like local supports, not bootstrap proportions, and the report and Methods say so. For a publication-grade NLR phylogeny, take `NLR_nbarc_domains.fa` into a full maximum-likelihood or Bayesian analysis.

**Nothing here is manually curated.** Classes, architectures and integrated-domain candidates are all automated calls, and the Methods section states that.

---

## Citation

> Eddy SR (2011) Accelerated profile HMM searches. *PLoS Computational Biology* 7: e1002195.

> Mistry J, Chuguransky S, Williams L, et al. (2021) Pfam: the protein families database in 2021. *Nucleic Acids Research* 49: D412-D419.

> Kourelis J, Sakai T, Adachi H, Kamoun S (2021) RefPlantNLR is a comprehensive collection of experimentally validated plant disease resistance proteins from the NLR family. *PLoS Biology* 19: e3001124.

> Buchfink B, Reuter K, Drost HG (2021) Sensitive protein alignments at tree-of-life scale using DIAMOND. *Nature Methods* 18: 366-368. *Used when -add_reference true.*

> Sarris PF, Cevik V, Dagdas G, Jones JDG, Krasileva KV (2016) Comparative analysis of plant immune receptor architectures uncovers host proteins likely targeted by pathogens. *BMC Biology* 14: 8. *Used when -integrated_domains true.*

> Steenwyk JL, Buida TJ III, Li Y, Shen X-X, Rokas A (2020) ClipKIT: a multiple sequence alignment trimming software for accurate phylogenomic inference. *PLoS Biology* 18: e3001007. *Used when -phylo true.*

> Price MN, Dehal PS, Arkin AP (2010) FastTree 2: approximately maximum-likelihood trees for large alignments. *PLoS ONE* 5: e9490. *Used when -phylo true.*

> EuchroGene NLR-Finder Pipeline v2.0 (2026). EuchroGene, LLC.

The Methods section inside `ANALYSIS_REPORT.html` is generated from the actual run settings and the tool versions queried at run time, with only the tools that actually ran cited, ready to copy into a manuscript.

**Support:** bioinformatics@euchrogene.com
