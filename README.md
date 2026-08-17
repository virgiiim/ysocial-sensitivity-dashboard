# Sensitivity dashboard

Static, interactive OFAT (one-factor-at-a-time) sensitivity dashboard for the
YSocial recommender-visibility paper, meant to be linked as a citable,
always-current companion to the main-text sensitivity summary and the
appendix tables.

**This is a temporary standalone public repo.** The dashboard's real home is
`icwsm-recsys-visibility` (private, source of truth for the analysis code and
data), under `docs/sensitivity/`. It's mirrored here only because that repo's
GitHub plan doesn't support Pages on a private repository. Once the paper is
out of any blind-review period, fold this back in: copy `index.html` and
`data/dashboard_data.json` from here into `docs/sensitivity/` there (same
filenames, same relative layout — it's a straight copy, nothing to adapt) and
enable Pages on that repo instead.

- **Stability overview** — largest absolute change in mean Gₚ/Gₐ/Cₚ/Cₐ
  (Gini/coverage, content/creator) versus each block's own reference value,
  per recommender. Shaded by magnitude relative to the largest shift in the
  table (a continuous gradient, not a fixed "material" cutoff — there's no
  established convention for what counts as a material Gini/coverage shift,
  so the dashboard deliberately doesn't assert one; that judgment belongs in
  the paper text).
- **Distribution shift (Jensen–Shannon)** — JS distance (bounded 0–1)
  between each tested value's recommendation-volume distribution and the
  block's reference distribution, comparing the whole shape rather than one
  summary number. Each distribution is rescaled by its own mean before the
  comparison, so a value that only changes how many recommendations are
  handed out in total (larger N, longer τₗ, more feed slots) is not by
  itself counted as a shape change — the same scale-invariance already
  built into Gini, applied here so the two are directly comparable. Because
  it's bounded, shading uses a fixed 0–1 scale and *is* comparable across
  blocks/topologies (the stability table above is not). Each value's own
  mean recommendation volume is in the hover tooltip for transparency.
- **Concentration & coverage** — entity-first grouped bars (Gₚ/Gₐ/Cₚ/Cₐ) by
  sensitivity value, mean ± sample SD.
- **Recommendation-volume distribution (φ)** — log–log PMF of recommendation
  volume conditional on r>0, same statistic as the paper's Figure 1, full
  resolution (no binning): run-level distributions aligned on their
  combined support, zero-filled, averaged with equal run weight.
- **Recommender contrast (ΔG, ΔC)** — the same contrast the paper's results
  text already reports (P/UCF vs RC, FP/LR vs F), across sensitivity values.
- **Popularity → future-exposure association (ρ)** — the paper's
  popularity-reinforcement Spearman correlation, faceted by recommender.
- **Degree-resolved creator exposure** (s̄ₐ, ūₐ by kₐ) — same, faceted by
  recommender.

Recommender colors and order come straight from
`src/ysocial_recsys_reviews/config.py` (`FEED_COLORS` / `FEED_ORDER`), so
they match the paper's matplotlib figures exactly. Notation (Gₚ/Gₐ, Cₚ/Cₐ,
ΔG/ΔC, ρ, kₐ, τₗ/k/D/N) matches `paper/main_revised_30gg.tex`. The one
dashboard-specific symbol is `DG`/`DC` (Latin D, deliberately not `Δ`) in the
stability table: it marks a shift versus an OFAT block's own reference
*value* (e.g. τₗ=24h vs the τₗ=36h reference), which is a different
comparison from the paper's `ΔG`/`ΔC` (a recommender versus its reference
*strategy*, RC or F). Don't conflate the two in the paper text.

## Files

```text
index.html             the page — logic + styling, ~60 KB, no external requests
data/
  dashboard_data.json   generated data bundle, fetched at runtime (~3 MB)
```

`index.html` fetches `data/dashboard_data.json` at load time. Refreshing the
data is therefore a data-only change — the HTML never needs to be touched
just because new runs landed.

## Refreshing after new OFAT runs

Regenerate in the main (private) repo, pointing the build script's output at
this checkout, then commit and push here:

```bash
# from the icwsm-recsys-visibility checkout
conda activate ysocial-analysis
python scripts/ysocial_recsys_reviews/build_sensitivity_dashboard.py \
  --output /path/to/ysocial-sensitivity-dashboard/data/dashboard_data.json

# from this repo's checkout
git add data/dashboard_data.json
git commit -m "Refresh sensitivity dashboard data"
git push
```

GitHub Pages redeploys automatically within a minute or two of the push. The
build script reads `data_sensitivity/*/website_data/` (the same tables the
`*_data_exploration.ipynb` notebooks already export in the main repo), so any
block/topology still short of the planned 4 runs per cell is simply
reflected as-is — the page's "Data status" badge and the caveat banner
already surface that.

## Viewing locally

`index.html` uses `fetch()`, which browsers block against `file://`. Serve
the folder over HTTP instead:

```bash
python -m http.server 8000
# open http://127.0.0.1:8000/
```

## Enabling GitHub Pages (one-time)

Settings → Pages → **Source: Deploy from a branch** → **Branch: `main`**,
**Folder: `/ (root)`** → Save. The site publishes at
`https://virgiiim.github.io/ysocial-sensitivity-dashboard/`.

## Known limitations

- ICF shows "–" everywhere, including the JSD table, because the
  `website_data/` exports this dashboard reads from don't currently include
  an ICF OFAT run of their own. If ICF sensitivity runs are ever added to
  that export, no dashboard change is needed — the data refresh picks them
  up automatically.
