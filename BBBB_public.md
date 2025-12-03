## 1. Project Infrastructure

### 1.1 Canonical Sources
- **Canonical BBBB:** `BBBB.md` (private repo)
- **Machine-readable Mirror:** `BBBB_public.md` (this file)
- **Working chapters:** in `/chapters/<chapter_name>/`
- **Teaching projects:** in `/teaching/`

### 1.2 Sync Direction
Private repo → Mirror repo → ChatGPT

### 1.3 Repo Structure

```text
Dissertation_BBBB_Mirror/
├── BBBB_public.md
├── chapters/
│   ├── chapter1_piciformes_brain/
│   │   ├── # Chapter 1 – Piciformes Brain: Overall Summary

## 1. Scope of the Chapter
- Comparative analysis of brain size evolution in Piciformes (~193 species).
- Focus on both **absolute brain size** and **phylogenetically size-corrected relative brain size (Brain_rel_PGLS)**.
- Integrates morphology, diet, dispersal/wing morphology, and phylogeny.

## 2. Session Summary – Project Hygiene + Canonical Pipeline (Today)

**What we did**
- Cleaned and refreshed the Piciformes brain project files:
  - Removed obsolete scripts, RData blobs, and drafts.
  - Ensured the repo reflects only the current, trusted pipeline.
- Established a clear policy for when to use different GPT modes (Pro, Deep Research, Extended, Regular) and recorded it in the project README.
- Canonicalized the Methods:
  - Study system and sampling (193 Piciformes; brain, mass, wing traits, diet, phylogeny).
  - Trait assembly (brain volume, AVONET traits, BirdWingData, diet).
  - Phylogeny (Jetz/BirdTree Hackett MCC tree; pruned, ultrametric).
  - PGLS pipelines for absolute and relative brain.
  - Model grids and AICc selection.
  - Diagnostics, GLS heteroscedastic models, influence analyses.
  - Leave-one-out (LOO) PGLS and BM/OU/EB process models.
- Drafted qualitative Results text aligned with the Methods (no numeric values yet).

**Key decisions**
- Absolute and relative brain size are treated as parallel but **not equal** analytical scales:
  - Absolute brain: stronger, more stable patterns driven by mass + wing + diet.
  - Relative brain (Brain_rel_PGLS): weaker, noisier, and more fragile.
- Causal inference policy:
  - Emphasis on association/patterns, **not** strong causal claims.
  - Interpretations must respect diagnostics, cross-validation, and process-model results.
- Presentation policy:
  - Tables are the **authoritative** source of numeric results.
  - Inline numbers are allowed but secondary to tables.

**Outstanding items**
- Integrate explicit causal-inference caution into Results and Discussion sections.
- Insert numeric values into tables and text once exported from R.

│   │   ├── # Chapter 1 – Piciformes Brain: Canonical Methods & Pipeline

## 1. Study System and Sampling
- Clade: Piciformes (woodpeckers, toucans, etc.).
- Sample: 193 species with complete data for:
  - Brain volume
  - Body mass
  - Wing/limb traits
  - Diet
  - Phylogeny (MCC tree)

## 2. Trait Data: Sources and Structure

### 2.1 Brain Volume
- Species-level brain volumes compiled from multiple datasets:
  - Garcia-Peña (male, female, average columns)
  - Multiple Iwaniuk datasets (Arnold, Nelson, Dean-Nelson, etc.)
  - Sayol
  - Corfield, Ksepka, Ferreira, Fedorova, Cardenas-Posada, Oyston, Fristoe
  - New CT-based measurements from LSU Museum specimens (Hodges)
- Within-source averaging followed by across-source averaging:
  - Canonical column: `Average_BrainVolume_mL`.
- Planned metadata file:
  - One row per trait column (especially brain volume),
  - Columns: `ColumnName`, `TraitType`, `SourceDataset`, `FullCitation`.

### 2.2 Body Mass
- Mass source: AVONET `BodyMass-Value`.
- Canonical column in analyses: `Mass_canonical`.
- Used for:
  - Brain–mass allometry.
  - Size correction for Brain_rel_PGLS.
  - As a predictor in absolute brain models.

### 2.3 Wing Morphology
- From AVONET:
  - Wing length, Kipp’s distance, hand-wing index, tarsus length, tail length.
- From BirdWingData_tidy_ver2.1:
  - Hand-wing area (HWA).
- At present, only brain volume incorporates new LSU measurements; wing traits are entirely from external datasets.

### 2.4 Diet
- Source: EltonTraits.
- Derived factor: `Diet_5Cat` from Elton guilds.
- Two diet categories with `n = 1` were dropped:
  - Reason: cannot estimate separate PGLS coefficients for singleton factor levels.
  - No further collapsing; the remaining five levels are a subset of EltonTraits categories.

## 3. Phylogeny

- Base tree: Jetz et al. BirdTree, Hackett backbone posterior.
- Use a Maximum Clade Credibility (MCC) tree:
  - `Hackett197MCCTree.tre` (or equivalent).
- Steps:
  - Match tree tips to the 193-species trait dataset.
  - Prune missing species.
  - Coerce to ultrametric as needed (e.g., `force.ultrametric("extend")`).
- This single MCC tree underlies all:
  - PGLS models,
  - GLS heteroscedastic models,
  - BM/OU/EB continuous trait models.

## 4. Trait Preprocessing and Size Correction

- Allometry:
  - Model: log(brain volume) ~ log(body mass) using PGLS with λ estimated by ML.
- Relative brain:
  - Extract PGLS residuals → `Brain_rel_PGLS`.
  - This is a **phylogenetically size-corrected** measure.
  - Emphasis: PGLS-based residualization, not OLS.

## 5. PGLS Framework and Model Classes

### 5.1 Absolute Brain Models
- Response: `Average_BrainVolume_mL` (or log-transformed).
- Predictors:
  - `Mass_canonical` (body mass)
  - Wing metrics (e.g., wing length)
  - `Diet_5Cat`
- λ always estimated by ML.

### 5.2 Relative Brain Models
- Response: `Brain_rel_PGLS`.
- Predictors:
  - Dispersal / wing proxies: HWI, HWA, Kipp’s distance, tail length, tarsus length, etc.
  - `Diet_5Cat`.
- λ estimated by ML.
- Interpretation: weaker, noisier models treated with additional caution.

## 6. Model Grids and AICc-Based Selection

- Candidate predictor sets defined for each model class.
- Helper function:
  - Fits all non-empty combinations of predictors for:
    - Absolute brain models.
    - Relative brain models.
- For each model:
  - Compute AICc.
  - Construct ΔAICc-based “confidence set” (ΔAICc ≤ 2).
- Inference:
  - Identify recurring predictors and assess model stability.
  - Avoid over-interpretation of any single best model.

## 7. Diagnostics and Assumption Checks

- Residual vs fitted plots.
- Residual vs predictor plots.
- Cook’s distance and leverage:
  - Computed from a non-phylogenetic analogue for diagnostics.
- Variance inflation factors (VIF):
  - Computed for non-phylogenetic model to diagnose multicollinearity.

## 8. Heteroscedastic GLS

- Framework: `nlme::gls` with Pagel’s λ correlation.
- Variance structures considered:
  - Homoscedastic.
  - `varIdent` (diet-specific variance).
  - `varPower` (variance as a function of brain, mass, or wing length).
  - `varExp` (variance as a function of fitted values).
- Model comparison via AICc:
  - Heteroscedastic models kept only if they improve fit without changing qualitative conclusions.

## 9. Influence and Robustness Checks

- Identify high Cook’s-D species.
- Refit key absolute brain PGLS models excluding influential species.
- Compare:
  - Coefficients and their signs.
  - λ estimates.
  - R² and AICc.
- Aim: ensure main conclusions are not driven by a handful of extreme taxa.

## 10. Leave-One-Out Cross-Validation (LOO-PGLS)

- For each focal model:
  - Refit the model `n` times, leaving out one species each time.
  - Re-estimate λ in each iteration.
- Compute:
  - Cross-validated R² (`R²_cv`).
  - RMSE.
- Purpose:
  - Distinguish genuine predictive signal from in-sample overfitting.
  - Particularly critical for relative brain models.

## 11. Evolutionary Process Models (BM, OU, EB)

- Fit BM, OU, and EB to:
  - `Average_BrainVolume_mL` and key wing traits.
- Implementation: (e.g., `geiger::fitContinuous` – verify exact package/options in the code).
- Compare models via AICc and AIC weights.
- Current results:
  - BM / weak OU adequate.
  - EB not supported.

## 12. Software and Reproducibility

- Analyses orchestrated in:
  - `PGLSScript_HWACorrection__Draft_120225.R` (or current canonical script).
- Input files:
  - Brain trait table (with all source columns).
  - Morphology + ecology file (AVONET traits).
  - BirdWingData-derived HWA.
  - EltonTraits-derived diet file.
  - MCC BirdTree phylogeny.
- Plan:
  - Provide code, data, and a metadata file linking each trait column to its citation.
  - Ensure Methods text and metadata file remain synchronized.

│   │   ├── # Chapter 1 – Piciformes Brain: Results State

## 1. Current Results Summary (Qualitative)

- Brain–body allometry:
  - Relative brain size defined via PGLS residuals (Brain_rel_PGLS) from log(brain) ~ log(mass).
- Absolute brain models:
  - Brain volume is strongly structured by mass, wing morphology, and diet.
  - Model grids and diagnostics indicate stable associations.
  - LOO CV suggests reasonable generalization.
- Relative brain models:
  - Brain_rel_PGLS models show weaker, noisier, and less stable patterns.
  - Effects often fail to generalize under LOO CV.
- Heteroscedastic GLS:
  - Alternative variance structures modestly improve model fit but do not change interpretation.
- Influence analyses:
  - Removing high Cook’s-D species does not flip signs or destroy main conclusions.
- Process models:
  - BM/weak OU adequate for brain and wing traits.
  - EB not supported.

## 2. Numeric Status

- Tables and inline values are **not yet populated**.
- Still needed:
  - Coefficients, SEs, p-values for key models.
  - R² and R²_cv.
  - λ estimates.
  - AICc and ΔAICc for key models and process models.
  - RMSE from LOO CV.

## 3. Planned Presentation

- Tables:
  - Primary repository for all numeric results.
- Text:
  - Interpretive and cautious.
  - Emphasizes:
    - Stronger case for absolute brain models,
    - Weak and non-generalizing relative brain models,
    - Non-causal framing.

│   │   └── # Chapter 1 – Piciformes Brain: Decisions & Open Questions

## 1. Canonical Decisions

- **Scale emphasis**
  - Absolute brain size is primary; relative brain size is secondary and interpreted cautiously.
- **Relative brain definition**
  - Brain_rel_PGLS = residuals from PGLS log(brain) ~ log(mass) with λ estimated by ML.
- **Phylogeny**
  - Single MCC Jetz/BirdTree Hackett phylogeny used for all analyses unless explicitly overridden.
- **Diet handling**
  - Diet data from EltonTraits → Diet_5Cat.
  - Categories with n = 1 removed; no further collapsing.
- **Mass definition**
  - Mass_canonical = AVONET BodyMass-Value.
- **Causal inference policy**
  - Results are interpreted as associations.
  - Causal narratives are explicitly constrained by diagnostics, LOO CV, and process models.
- **Presentation**
  - Tables are the authoritative source of numeric values.
  - Inline numbers are allowed but not the primary record.

## 2. Open Questions

- How prominently to feature BM/OU/EB results (central vs contextual).
- Exact wording for causal-inference caveats in Results and Discussion.
- Final placement and format of the metadata file (appendix vs supplementary vs both).
- Which numeric diagnostics to report in-text vs table-only.

## 3. Next Actions

- Build and export model summaries from R (absolute, relative, GLS, LOO, BM/OU/EB).
- Construct markdown-ready tables for Chapter 1.
- Revise Results text to:
  - Explicitly foreground associational framing.
  - Highlight failure of relative brain models to generalize.
  - Show how diagnostics/LOO constrain causal stories.

│   └── <future chapter dirs>/
├── teaching/
│   ├── BIOL_4161_TA_Standardization/
│   │   ├── summary.md
│   │   ├── neural_canon.md
│   │   ├── tasks_schema.md
│   │   └── decisions_open.md
└── tasks/
    ├── inbox.md
    ├── this_week.md
    └── log_2025.md

```


## 2. Global Dissertation Aims
(placeholder)

---

## 3. Analytical Pipelines
3.1 Chapter 1 – Piciformes Brain  
→ Points to: `chapters/chapter1_piciformes_brain/methods_pipeline.md`

---

## 4. Chapter Summaries
4.1 Chapter 1 – Piciformes Brain  
→ Points to:  
- `summary.md`  
- `results_state.md`  
- `decisions_and_open_questions.md`

---

## 5. Decision Log
High-level durable decisions across chapters.

---

## 6. Outstanding Questions
Active questions requiring resolution.

---

## 7. Task Routing
Tasks stored in:  
- `/tasks/inbox.md`  
- `/tasks/this_week.md`  
- `/tasks/log_2025.md`

---

## 8. Teaching Projects
8.1 BIOL 4161 – TA Standardization  
→ Points to:  
- `summary.md`  
- `neural_canon.md`  
- `decisions_open.md`
