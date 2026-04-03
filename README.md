# This is for EuchroGene Members.

NLR-Finder (EuchroGene NLR-Finder)
Automated identification and annotation of NB-LRR (NLR) disease resistance genes from plant proteomes. NLR-Finder accepts a predicted protein FASTA file (and optionally a GFF3 annotation) and performs HMMER3-based domain profiling against a curated set of NLR-relevant Pfam HMMs. Each candidate is classified into functional classes — TNL, CNL, RNL, TN, or NBS-only — following the RefPlantNLR nomenclature (Kourelis et al. 2021). Full gene models are extracted from the GFF3 when provided, and per-class protein FASTA files are generated alongside an interactive HTML report and a publication-ready methods section.

## How It Works:
1. **Input Validation:** Protein FASTA and optional GFF3 files are validated and staged for containerized execution.
2. **HMMER3 Domain Search:** All proteins are searched against 8 curated NLR Pfam profiles (NB-ARC, TIR, LRR_1, LRR_4, LRR_8, LRRNT_2, RPW8, Rx_N) using `hmmsearch`.
3. **NLR Classification:** Candidates are classified into TNL / CNL / RNL / TN / NBS-only based on domain architecture following Kourelis et al. (2021).
4. **Gene Model & Sequence Extraction:** Matching gene models are extracted from the GFF3 (gene → mRNA → exon/CDS hierarchy), and NLR protein sequences are written to class-specific FASTA files.
5. **Report Generation:** An interactive HTML analysis report, a machine-readable JSON summary, and a journal-ready methods section are produced automatically.

## Required Inputs:
1. **Protein FASTA** — All predicted proteins from the target genome (e.g., output from Augustus, MAKER, BRAKER, or equivalent).
2. **GFF3 annotation** *(optional)* — Genome annotation file whose gene IDs match the protein FASTA headers. Required to produce `NLR_genes.gff3` with full gene model output.

## Post-Analysis:
- Review `ANALYSIS_REPORT.html` for an interactive summary of NLR class distribution, hit statistics, and the full classification table.
- Use the per-class FASTA files (`by_class/NLR_TNL.fa`, `NLR_CNL.fa`, etc.) for downstream phylogenetic or functional analyses.
- The `METHODS_FOR_PUBLICATION.txt` file contains a complete, ready-to-paste methods paragraph with all tool versions and citations.

---

## Installation

### 0. Install EG_tools &nbsp; *(skip if already installed)*
```
wget https://github.com/euchrogene/EG_tools/raw/refs/heads/main/EG_tools
sudo chmod 777 EG_tools
sudo mv EG_tools /usr/bin
```

### 1. Install NLR-finder
```
sudo EG_tools install -r https://github.com/euchrogene/NLR-finder.git -d NLR-finder -e NLR-finder_v.1.0 -m "NB-LRR disease resistance gene identification and annotation"
```

### 2. Display installed software
```
EG_tools
```

### 3. Show help contents
```
NLR-finder_v.1.0
```

### 4. Uninstall
```
sudo EG_tools uninstall -t NLR-finder_v.1.0 -i managene7/nlr-finder:v.1.0
```

---

## Help Contents:
```
This pipeline is provided by EuchroGene, LLC.
Bug reports: bioinformatics@euchrogene.com

============================================================================
EuchroGene NLR-Finder Pipeline v1.0.0
Docker Image: managene7/nlr-finder:v.1.0
============================================================================

DESCRIPTION:
  Identifies all NB-LRR (NLR) disease resistance genes from a proteome
  using HMMER3 Pfam domain search (NB-ARC, TIR, LRR, RPW8 profiles).

  GFF3 is optional. Two run modes:
    Protein-only mode : -protein only  → NLR FASTA + classification table
    Full mode         : -protein + -gff → + complete GFF3 gene models

USAGE:
  NLR-finder_v.1.0 -protein <proteins.fa> [OPTIONS]

REQUIRED:
  -protein <FILE>      Protein FASTA (all predicted proteins)

OPTIONAL:
  -gff     <FILE>      GFF3 annotation (enables full gene model extraction)
  -output  <DIR>       Output folder name (default: NLR_Finder_Results_<date>)
  -threads <N>         CPU threads (default: 32)
  -max_memory <SIZE>   JVM heap (reserved), e.g. 8g, 16g (default: 16g)
  -evalue  <FLOAT>     HMMER domain e-value cutoff (default: 1e-5)

EXAMPLES:

  # Protein-only mode (no GFF3)
  NLR-finder_v.1.0 -protein genome_proteins.fa

  # Full mode with GFF3
  NLR-finder_v.1.0 -protein genome_proteins.fa -gff genome.gff3

  # Custom settings
  NLR-finder_v.1.0 -protein proteins.fa -gff annotation.gff3 \
                   -output MyNLR_Run -threads 32 -evalue 1e-10

OUTPUT FILES (protein-only mode):

  <o>/
  ├── NLR_proteins.fa              NLR protein sequences — all classes combined (FASTA)
  ├── by_class/
  │   ├── NLR_TNL.fa               TIR-NBS-LRR sequences
  │   ├── NLR_CNL.fa               CC-NBS-LRR sequences
  │   ├── NLR_RNL.fa               RPW8-NBS-LRR sequences
  │   ├── NLR_TN.fa                TIR-NBS sequences (LRR-truncated)
  │   └── NLR_NBS-only.fa          NBS-only / degenerate sequences
  ├── NLR_classification.tsv       Domain classification (TNL/CNL/RNL/TN/NBS-only)
  ├── hmmer_domtblout.txt          Raw HMMER domain table
  ├── pipeline_summary.json        Machine-readable run summary
  ├── ANALYSIS_REPORT.html         Interactive HTML report
  ├── METHODS_FOR_PUBLICATION.txt  Ready-to-paste journal methods section
  └── RUN_SPEC.json                Full run specification

OUTPUT FILES (full mode, with -gff):

  <o>/
  ├── NLR_proteins.fa              (as above)
  ├── by_class/
  │   ├── NLR_TNL.fa               (as above)
  │   ├── NLR_CNL.fa               (as above)
  │   ├── NLR_RNL.fa               (as above)
  │   ├── NLR_TN.fa                (as above)
  │   └── NLR_NBS-only.fa          (as above)
  ├── NLR_genes.gff3               NLR gene models — full hierarchy (GFF3)
  ├── NLR_classification.tsv       (as above)
  ├── hmmer_domtblout.txt          (as above)
  ├── pipeline_summary.json        (as above)
  ├── ANALYSIS_REPORT.html         (as above, includes GFF3 stats)
  ├── METHODS_FOR_PUBLICATION.txt  (as above, includes GFF3 paragraph)
  └── RUN_SPEC.json                (as above)

SUPPORT:
  Bugs / Questions: bioinformatics@euchrogene.com

============================================================================
```

---

## Citation

If you use this pipeline in published research, please cite:

> Eddy SR (2011) Accelerated profile HMM searches. *PLoS Comput Biol* 7(10): e1002195. https://doi.org/10.1371/journal.pcbi.1002195

> Mistry J, Chuguransky S, Williams L, et al. (2021) Pfam: The protein families database in 2021. *Nucleic Acids Res* 49(D1): D412–D419. https://doi.org/10.1093/nar/gkaa913

> Kourelis J, Sakai T, Adachi H, Kamoun S (2021) RefPlantNLR is a comprehensive collection of experimentally validated plant disease resistance proteins from the NLR family. *PLoS Biol* 19(10): e3001124. https://doi.org/10.1371/journal.pbio.3001124

> EuchroGene NLR-Finder v1.0 (2026). EuchroGene, LLC. bioinformatics@euchrogene.com

The `METHODS_FOR_PUBLICATION.txt` file generated at the end of each run contains a complete methods paragraph formatted for journal submission, including all tool versions and references.
