# NetMedScope

An offline web workbench for network-medicine screening.

NetMedScope does **not** invent a new method: the science — proximity/separation
z-scores, null models, LCC significance — is [NetMedPy](https://pypi.org/project/netmedpy/)
(Aldana et al., *Bioinformatics* 2025). What this project adds is the part that
stops a bench biologist: parsing inputs, cutting an analysable sub-network,
caching distance matrices, running the permutations, explaining the numbers in
plain language, drawing the figures and exporting one shareable report.

> Windows 10/11 · local computation · no coding required

![proximity chart](docs/screenshot-proximity.png)

## Why it exists

The paper's Python API expects you to write code, know what a null model is, and
know that a *negative* z means "closer than chance". NetMedScope turns that into:
three file inputs, one button, one sentence of Chinese, one HTML report you can
mail to your supervisor.

## Quick start

```bat
start.bat
```

Then open <http://127.0.0.1:8300>, click **载入示例并运行** (load the bundled
example) — no data needed. First run creates `.venv` and installs dependencies
(3–5 min); later starts take seconds.

Command line, if you prefer:

```bash
pip install -r requirements.txt
python -m netmedscope.cli --network examples/network.csv \
  --drugs examples/drugs.csv --diseases examples/diseases.csv \
  --metric shortest_path --null uniform --iterations 1000 --out results/run1
```

## What you get

| Output | Where |
|---|---|
| Pair-level scores (proximity / separation / separation-z) | `proximity.csv` (or `<score>.csv`) |
| Matrix view of the same numbers | `<score>_wide.csv` |
| LCC size, z and p per gene set | `lcc.csv` |
| Figures (bar chart, heatmap, LCC chart) | `*.svg` |
| Provenance: options, input hashes, timings, versions | `run.json` |
| One shareable report with charts inlined | `/api/job/<id>/report.html` |

## Parameters, in one screen

- **distance**: shortest path (AMSPL, usual), random walk (robust to incomplete
  networks), biased random walk (degree weighted), communicability (all paths).
- **null model**: degree match (strictest), log binning, strength binning,
  uniform (fastest).
- **z sign**: proximity/separation z = (observed − random)/sd, so **negative means
  closer than chance**; LCC z is the opposite — **positive means more clustered**.
  Verified in `tests/`.
- **iterations**: permutation count; the smallest resolvable p is 1/(N+1), so use
  1000+ for a real analysis.
- **raw separation has no null model** in NetMedPy, so it has no z/p at all — the
  UI says so instead of inventing one.

## Repository layout

```
netmedscope/engine.py      input parsing, sub-network, matrix cache, NetMedPy wrappers
netmedscope/charts.py      dependency-free SVG charts (bar, heatmap)
netmedscope/web.py         Flask UI, background jobs, HTML report
netmedscope/cli.py         command line
netmedscope/templates/     page
examples/                  runnable slice of the human interactome + 2 gene sets
tests/                     20 tests (engine, web, report)
tools/                     verification scripts used to check the numbers
```

## Tests

```bash
python -m unittest discover -s tests -v
```

## Attribution and licence

Method and statistics: NetMedPy (MIT). Please cite
Aldana A. et al. *NetMedPy: A Python package for Large-Scale Network Medicine
Screening*. Bioinformatics 2025;41(9):btaf338, and the underlying papers
(Barabási 2011 doi:10.1038/nrg2918; Menche 2015 doi:10.1126/science.1257601;
Guney 2016 doi:10.1038/ncomms10331).

The code in this repository (workflow, caching, charts, report) is MIT licensed;
see `LICENSE`. NetMedScope is not affiliated with the NetMedPy authors.

## Known limits

- Only `shortest_path` has been exercised on real data so far; the other metrics
  are wired but slower (communicability especially).
- Permutation p-values are coarse below 1000 iterations.
- The web UI has automated tests but has not been through a manual browser pass.
- Custom distance functions and custom null models are CLI-only territory: the
  web form deliberately does not accept code.
