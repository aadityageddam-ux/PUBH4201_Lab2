# Lab 2: Differential expression in dexamethasone-treated airway cells

PUBH 4201 (undergrad) Applied Computing in Health Data Science, Lab 2 (Analysis Notebook).

## What this is

An analysis of [GSE52778](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE52778) (Himes et al. 2014), RNA-seq of 4 human airway smooth muscle cell lines treated with dexamethasone vs. untreated. I loaded the processed FPKM matrix straight from GEO, filtered expressed genes, ran a per-gene t-test with a log2 fold-change and Benjamini-Hochberg FDR correction, and checked whether the results reproduce a handful of known glucocorticoid-response genes from the literature (they mostly do).

Single language (Python) 

## Repo structure

```
notebooks/
  airway_dex_degs.ipynb   <- the notebook, source
  airway_dex_degs.html    <- rendered export (open this to just read results, no need to run anything)
  volcano_plot.png        <- the main figure, saved out by the notebook
pyproject.toml / uv.lock  <- dependencies (managed with uv)
AI_USAGE.md               <- AI assistance documentation
```

## How to re-run it

Requires [uv](https://docs.astral.sh/uv/).

```bash
uv sync
uv run jupyter nbconvert --to notebook --execute --inplace notebooks/airway_dex_degs.ipynb
```

That's the same as doing "Restart & Run All" in the Jupyter UI because it re-downloads the data from GEO each time (no local/cached copy of the data is committed), so it needs an internet connection and takes maybe 15-20 seconds. To also regenerate the HTML export:

```bash
uv run jupyter nbconvert --to html notebooks/airway_dex_degs.ipynb
```

Or just open `notebooks/airway_dex_degs.ipynb` in Jupyter/VS Code and hit Restart & Run All yourself.

## Comments for instructor

No caveats beyond what's already flagged in the notebook itself with the main one being that this is an *unpaired* comparison (Can see in the markdown cell right before the differential expression section), since the per-sample column names in this particular processed GEO file doesn't preserve which treated/untreated replicate came from the same original cell line.
