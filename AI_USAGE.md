# AI Usage

**Model used:** Claude Sonnet 5, via Claude Code (Anthropic's CLI agent).

Claude walked me through this lab interactively reading the assignment, explaining what it found, and stopping to ask me to make the actual calls (dataset, analysis approach, scope, whether to fix things it found) rather than just deciding everything itself and handing me a finished repo. Below are the actual prompts from that session followed by two of the debugging interactions I watched happen live.

## Prompts 

1. > read this github link with instructions and rubric for my lab and then help me complete it. Always ask questions if you have any never assume anything. 

2. > For design and project choices always ask me. I want to work on the genomics because its most interesting to me. 

   The model looked at the GSE52778 airway dataset, then asked me via explicit multiple-choice questions, not by just picking, which analysis approach (differential expression vs. PCA vs. both), how much scope to attempt (base requirements vs. the R addendum vs. mixed-language extra credit), and what to name the repo before writing any code.

3. > R is installed but just do python

   Claude had found R 4.6.1 actually installed on my machine (with rmarkdown/knitr already there) partway through, which technically made the R addendum feasible. I told it to skip that anyway and stick to Python only.

4. > Were you able to check the rubric to make sure it has everything done correctly and followed instructions

   Claude re-checked the actual rubric/README text (not just its own memory of it) and did a genuine from-scratch `git clone` + `uv sync` + notebook re-execution in a scratch folder to verify the environment truly rebuilds from nothing, the way the grading script would. That check caught two real things: it re-ran the environment-setup error below in a location that disproved its own earlier explanation of the cause (see Interaction 1), and it found that this file was missing prompt/process documentation the syllabus's `GRADING_AND_POLICIES.md` requires beyond what the lab-specific `rubric.md` checklist asks for.

## Interaction 1: `uv add` failing with a hardlink error

While setting up the Python environment with `uv add pandas matplotlib scipy jupyter nbconvert ipykernel`, the install failed with:

```
error: Failed to install: jsonschema_specifications-2025.9.1-py3-none-any.whl (jsonschema-specifications==2025.9.1)
  Caused by: failed to hardlink file from C:\Users\aadit\OneDrive\...\jsonschema_specifications\tests\test_jsonschema_specifications.py to C:\Users\aadit\AppData\Local\uv\cache\...: The cloud operation cannot be performed on a file with incompatible hardlinks. (os error 396)
```

Before touching anything, Claude explained what this actually meant: `uv` normally installs packages by hardlinking files from its global cache into the project's `.venv` instead of copying them (faster, avoids duplicating the same file on disk), and the error is Windows refusing that hardlink because a "cloud operation" is involved. Claude's first-pass explanation was that this was specifically because my project folder sits inside OneDrive-synced `Desktop`. The fix it applied, based on that read, was setting `UV_LINK_MODE=copy` so `uv` copies files instead of hardlinking them, and that worked.

**Where that first explanation turned out to be incomplete:** later in the session, while re-checking the submission against the rubric (prompt 4 above), Claude re-tested the setup from a completely fresh `git clone` in `%TEMP%` — a folder that is *not* OneDrive-synced — and hit the exact same "cloud operation" hardlink error there too. That's direct evidence the original "it's because the folder is inside OneDrive" explanation was wrong, or at least not the whole story (some filesystem filter appears to be intercepting hardlinks more broadly on this machine than just OneDrive's synced folders — Claude didn't fully pin down the precise mechanism, and said so rather than guessing further). The `UV_LINK_MODE=copy` workaround was re-verified to fix it in that non-OneDrive location too, so the fix is solid even though the original root-cause explanation for *why* it was needed wasn't fully accurate. I'm documenting the correction itself here because catching an AI-generated explanation being wrong, via an independent test, felt more honest to record than quietly editing it to sound right the first time.

That first retry attempt also hit a *second*, separate error (`The process cannot access the file because it is being used by another process. (os error 32)`), which Claude treated as a transient file lock, deleting the half-built `.venv` and rerunning `uv sync` + `uv add` from scratch. I verified both fixes by checking the retry output showed all packages installing with zero errors, then running `python -c "import pandas, matplotlib, scipy; print('ok', ...)"`, which printed version numbers instead of a traceback — and again, independently, by the full fresh-clone rebuild described above.

## Interaction 2: `pd.read_csv` silently reading the whole file as one column

The GEO source file (`GSE52778_All_Sample_FPKM_Matrix.txt.gz`) *looks* tab-separated when you eyeball it in a terminal, so the notebook's first draft used `pd.read_csv(url, sep="\t", compression="gzip")`. That call didn't throw an error, it "succeeded" but produced a dataframe with exactly 1 column (the whole header crammed into one string), which only surfaced two cells later as:

```
KeyError: "None of [Index(['Dex_LL14', 'Dex_LL06', 'Dex_LL02', 'Dex_LL10', 'Untreated_LL09',
       'Untreated_LL05', 'Untreated_LL13', 'Untreated_LL01'],
      dtype='str')] are in the [columns]"
```

Claude first checked what the error meant by printing `list(fpkm.columns)`, which showed the "column names" were actually the entire header line glued together meaning the separator assumption was wrong, not that the column names themselves were misspelled. It then confirmed the fix (`sep=r"\s+"` for whitespace-delimited files) by re-running the same load and printing `fpkm.shape`, which went from `(23273, 1)` to the correct `(23273, 41)` with individually-named columns matching the file's documented header. That fix is now called out with an inline comment in the notebook itself (the cell that loads the data), since it's a genuine gotcha someone re-running this later would otherwise hit too.

## What was mine vs. AI-generated

The code, analysis choices (expression filter threshold, Welch's t-test, BH-FDR, which known genes to check), and all design choices in the notebook and this file were built with claude under my direction. My own contributions were the decisions documented in the prompt log above dataset choice, scope, catching errors, working through bugs, and checking over the rubric and instructions to make sure its done correctly.
