# Site-level variation in wastewater resistomes: reproducible analysis

Code, derived tables and figures for a secondary analysis of public processed data from:

> Singh NK, et al. Metagenomic profiling of antimicrobial resistance in wastewater from metropolitan cities of India. *Nature Communications* (2026). doi:10.1038/s41467-026-70702-x

**Question.** Does pooling samples to one value per city and month hide differences between sewer sites in antimicrobial resistance gene (ARG) drug-class composition, and how well does a two-site survey reproduce the profile of all sites sampled?

**Status.** Research code that accompanies a manuscript in preparation. Not peer reviewed. The analysis plan was frozen internally on 26 October 2026, before the site-level analyses were run; it was not registered externally. Step 6 contains post hoc analyses added after review.

**AI-assistance disclosure.** The analysis code was drafted with the assistance of Claude (Anthropic) and [authors: state how you reviewed and verified it]. All result tables can be regenerated with the notebooks, which check the regenerated tables against the committed ones.

## Quick start

**Option 1: Google Colab (easiest).**

1. Open `notebooks/colab_reproduce.ipynb` in Google Colab (upload it, or open it from GitHub once the repository is public).
2. Choose *Runtime > Run all*. Upload `github_repo.zip` when prompted (not needed if you cloned the repository).
3. The notebook downloads the public data from Zenodo, verifies the archive checksum, runs steps 2 to 6, shows the figures, and ends with a table check (`OK` or `CHECK`). Expected run time: a few minutes.

If the check shows `CHECK`, Colab's package versions may differ from `requirements.txt`: use the optional pinned install in section 2 of the notebook, restart the session and run again. If the Zenodo download fails, download `data.tar.xz` from the record, upload it to `data/raw/` and run the data cell again.

**Option 2: local machine.**

1. Download the original data archive (see "Data sources") and extract `meta.tsv`, `rgi_bowtie.tsv` and `bacterial_read_df.tsv` into `data/raw/`.
2. Install the environment: `pip install -r requirements.txt` (plus `jupyter` to open the notebook).
3. Open `notebooks/reproduce_analysis.ipynb` from the repository root and run all cells. In testing the whole run took under one minute.

You can also run the scripts directly in the order given under "Pipeline".

## Data sources (not redistributed here)

| Input | Where to get it |
|---|---|
| Processed tables (`data.tar.xz`, MD5 `dd9e8bfaaad2aa5b33c81c7fd1a8b5bc`) | Zenodo, doi:10.5281/zenodo.17590646 (confirm the record and its licence before reuse) |
| Original authors' code | GitHub `SowpatiLab/Metro_metagenomics`; Zenodo record 18465564 |
| Sample-to-site key | NCBI SRA run table, BioProject PRJNA1415085 (`location` field); already joined in `tables/master_sample_site_map.csv` |

The original data archive is distinct from this repository: this repository holds only the secondary-analysis code, the site key and the derived outputs. Please cite the original authors and their data record whenever you use it.

## Pipeline

| Step | Script | What it does |
|---|---|---|
| 1 | `scripts/step1_audit.py` | Integrity and metadata audit of the downloaded archive (needs the raw data; not run by the notebooks) |
| 2 | `scripts/step2_sample_level_cpm.py` | Sample-level CPM (mapped reads / total reads x 1e6), identity >= 90%, detection above 10 CPM per sample |
| 3 | `scripts/step3_primary_analysis.py` | Analysis A (PERMANOVA by site within city, permutations restricted within month, plus dispersion test), B (pooled vs equal-site estimates) and C (two-site resampling). Seed 20261024 |
| 4 | `scripts/step4_sensitivity.py` | Sensitivity analyses (5 and 20 CPM, lowest-depth 10% removed, Aitchison distance) and leave-one-site-out |
| 5 | `scripts/step5_figures.py`, `scripts/step5b_composition_figure.py` | Figures 1, 2, 4 and 5; Figure 3 (composition by site) |
| 6 | `scripts/step6_audit_extras.py` | Post hoc analyses: exact pair enumeration with city-month bootstrap (Analysis C), PERMANOVA of site given month, Mumbai dispersion by site and exclusion of the most dispersed site. Seed 20261101 |

`scripts/step2_master_table.R` is an R equivalent of step 2 that was written but not executed.

## Main results (for orientation; see the manuscript for context)

| Item | Result |
|---|---|
| Samples | 447 collected, 19 sites, four cities; 446 analysed (one sample with no detections excluded) |
| A: site effect (adjusted p) | Mumbai R² 0.27 (0.0004; dispersion also differs); Delhi 0.07 (0.0024; lost without Najafgarh Drain); Kolkata 0.05 (0.069; two sites); Chennai 0.01 (0.94) |
| B: pooled vs equal-site | median relative difference 1.8% to 2.9% (sum of absolute class-level differences / pooled total) |
| C: two sites vs all sites | top three classes matched in 54% (Chennai), 59% (Mumbai) and 71% (Delhi) of pairs |

## Repository layout

- `notebooks/` : `colab_reproduce.ipynb` (Colab) and `reproduce_analysis.ipynb` (local)
- `scripts/` : analysis scripts
- `tables/` : committed result tables (steps 3, 4 and 6), supplementary tables S1 to S3, the site key and the sample summary
- `figures/` : Figures 1 to 5 (PNG and PDF)
- `data/raw/` : empty; place the downloaded input files here (ignored by git)
- `results/` : created when you run the pipeline (ignored by git)
- `requirements.txt`, `CHECKSUMS_SHA256.txt` : environment and file checksums

## Main caveats

- Site identity was reconstructed from public archive metadata; BioSample coordinates are missing.
- One sample per site-month; the design is unbalanced and Kolkata has only two sites.
- The 10 CPM cut-off is applied per sample here but to pooled city-month values in the original study.
- Mumbai: the dispersion test is significant, and one site (Pratiksha Nagar Wadala) accounts for about half of the effect. Delhi: the result depends on one site (Najafgarh Drain).
- Analysis C compares two sites with the profile of all sampled sites, which includes them; it is not the true citywide resistome. Resamples are not independent.
- Only computational aggregation was evaluated; no physically pooled wastewater was sequenced.
- The 95% identity sensitivity setting is uninformative because all retained hits have 100% identity.
- DNA-based read annotations: no claims about expression, viability, transfer or clinical risk.

## Licence and citation

