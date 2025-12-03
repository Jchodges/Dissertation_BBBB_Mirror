# BIG BAD BOOK OF BUSINESS (BBBB – Canonical)

This document is the canonical spine of my dissertation work.

It records:
- Projects and chapters
- Analytical pipelines
- Decisions and assumptions
- Open questions
- Task pointers (linked to a tasks system, not the full to-do list)

The GitHub mirror (`Dissertation_BBBB_Mirror`) holds a **sanitized, machine-readable subset** of this content.

---

## 0. PROJECT INFRASTRUCTURE

### 0.1 Canonical Sources

- **Canonical BBBB (this file):** `BBBB/BBBB.md` (private repo)
- **Public mirror:** `Dissertation_BBBB_Mirror` on GitHub
  - `BBBB_public.md` (index)
  - `/chapters/` for chapter-specific summaries, pipelines, decisions
  - `/teaching/` for teaching/TA projects
  - `/tasks/` for inbox, this_week, log

### 0.2 Sync Direction

1. Edit and maintain **canonical BBBB** here.
2. Periodically copy relevant (non-sensitive) chunks → `BBBB_public.md` and per-project mirror files.
3. ChatGPT reads *only* from the mirror, but content and decisions originate here.

### 0.3 Control Phrases

- **END CURRENT SESSION** → log what we did, decisions, open questions, next 1–3 actions.
- **RESTART SCRIPT** → reconstruct context from BBBB + mirror, list open branches, propose top 3 priorities.

---

## 1. GLOBAL DISSERTATION AIMS

*(Fill this in later when you want a clean statement of the big questions.)*

- High-level aim:
- Focal clade(s):
- Core comparative questions:
- Target chapters:

---

## 2. PROJECT LIST

### 2.1 Dissertation Chapters (Core Research)

1. **Chapter 1 – Piciformes Brain Evolution**
   - Comparative analysis of absolute vs relative brain size in ~193 Piciformes.
   - Links brain size to body mass, wing/dispersal traits, diet, and phylogeny.
   - Heavy use of PGLS, model grids, diagnostics, GLS, influence analyses, LOO CV, and BM/OU/EB process models.

2. **Chapter 2 – [Placeholder]**
   - To be defined.

3. **Chapter 3 – [Placeholder]**
   - To be defined.

### 2.2 Teaching / Side Projects

1. **BIOL 4161 – TA Standardization & Cardiac Treatment Database**
   - System-specific TA docs for:
     - Neural physiology
     - Skeletomuscular physiology
     - Cardiac physiology
   - Long-term: database of cardiac treatments tied to the final project.

---

## 3. ANALYTICAL PIPELINES (BY CHAPTER)

### 3.1 Chapter 1 – Piciformes Brain: Canonical Pipeline

**Study system and sampling**

- Clade: Piciformes.
- N ≈ 193 species with complete: brain volume, body mass, wing traits, diet, phylogeny.

**Trait data**

- Brain volume:
  - Multiple named datasets (Garcia-Peña, Iwaniuk variants, Sayol, Corfield, Ksepka, Ferreira, Fedorova, Cardenas-Posada, Oyston, Fristoe).
  - New CT-based measurements from LSU specimens (Hodges).
  - Within-source averaging, then across sources → `Average_BrainVolume_mL`.
  - Planned metadata file mapping each brain column to full citation.

- Mass:
  - AVONET `BodyMass-Value` → `Mass_canonical`.

- Wing traits:
  - From AVONET: wing length, Kipp’s distance, HWI, tarsus length, tail length.
  - From BirdWingData: HWA.

- Diet:
  - From EltonTraits.
  - Derived factor: `Diet_5Cat`.
  - Categories with n = 1 dropped (cannot estimate factor level); no additional merging.

**Phylogeny**

- MCC Jetz/BirdTree Hackett-based phylogeny (e.g. `Hackett197MCCTree.tre`).
- Tips matched to 193 species, tree pruned and made ultrametric if needed.
- **Single MCC tree** used for:
  - PGLS
  - GLS heteroscedastic models
  - BM/OU/EB process models

**Relative brain definition**

- Fit PGLS: log(brain) ~ log(mass) with λ estimated by ML.
- Extract PGLS residuals → `Brain_rel_PGLS`.
- This is the canonical “relative brain size” metric (phylogenetically size-corrected).

**Model classes**

- Absolute brain models:
  - Response: `Average_BrainVolume_mL` (or log).
  - Predictors: `Mass_canonical` + wing metrics + `Diet_5Cat`.
  - λ estimated by ML.

- Relative brain models:
  - Response: `Brain_rel_PGLS`.
  - Predictors: dispersal/wing traits (HWI, HWA, etc.) + `Diet_5Cat`.
  - λ estimated by ML.

**Model grids & selection**

- Systematic PGLS grids:
  - Fit all non-empty combinations of predictor sets for absolute and relative models.
  - AICc for each model; ΔAICc ≤ 2 defines confidence set.
  - Focus on recurring predictors and model stability.

**Diagnostics & robustness**

- Residual vs fitted and residual vs predictor plots.
- Cook’s D and leverage (from non-phylogenetic analogue) for influence.
- VIF (non-phylogenetic) for collinearity.
- Heteroscedastic GLS via `nlme::gls`:
  - Homoscedastic, `varIdent`, `varPower`, `varExp`.
  - Model comparison via AICc; heteroscedastic structures retained only if they improve fit without changing interpretations.

**Influence checks**

- Identify high-Cook’s-D species.
- Refit main absolute brain models excluding them.
- Compare coefficients, λ, R², AICc → check that conclusions are not driven by a few taxa.

**LOO cross-validation**

- For focal models:
  - Leave-one-out PGLS with λ re-estimated each time.
  - Calculate R²_cv and RMSE.
- Used to:
  - Quantify generalization,
  - Penalize overfitting,
  - Highlight the weakness of relative brain models.

**Process models (BM/OU/EB)**

- Fit BM, OU, EB to brain and key wing traits (e.g., `geiger::fitContinuous` or equivalent; confirm in code).
- Compare via AICc and weights.
- Current qualitative result: BM / weak OU sufficient; EB not supported.

**Reproducibility**

- Central R script (e.g. `PGLSScript_HWACorrection__Draft_120225.R`) orchestrates analyses.
- Data and tree stored as versioned files.
- Plan: metadata file tying each trait column to source and citation; code + data to be shared as appendix/supplement.

---

## 4. CHAPTER STATES

### 4.1 Chapter 1 – Piciformes Brain: Current State

**What is done**

- Project hygiene:
  - Legacy scripts, RData blobs, and drafts cleaned out; project reflects only the trusted pipeline.
- Methods:
  - Full Methods section drafted, aligned with actual R workflow (see pipeline above).
- Results:
  - Draft qualitative Results:
    - Absolute brain models: strong, stable associations (mass + wing + diet).
    - Relative brain models: weaker, noisier, less predictive; LOO CV shows poor generalization.
    - GLS heteroscedastic models: minor fit improvements, no change in interpretation.
    - Influence analysis: removing high-Cook’s-D species does not overturn results.
    - Process models: BM/weak OU, no evidence for EB.

**What is NOT done**

- No numeric Results yet:
  - Coefficients, SEs, R², R²_cv, λ, AICc, ΔAICc, RMSE, BM/OU/EB stats still need exporting and table-building.
- Causal framing not fully revised:
  - Current text still leans toward “strong association” prose; must be rewritten to foreground limits on causal inference.

**Causal language policy for this chapter**

- Treat PGLS results and process-model fits as **associations**, not mechanistic proof.
- Explicitly warn against “ham-fisted” causal narratives without:
  - Diagnostics,
  - Cross-validation,
  - Process-consistent behavior.
- Commit to:
  - Highlighting where models fail (especially relative brain),
  - Connecting that failure to broader debates about brain size evolution.

---

## 5. TEACHING / TA PROJECTS

### 5.1 BIOL 4161 – TA Standardization & Cardiac Treatment Database

**Scope**

- Course: BIOL 4161 Vertebrate Physiology Lab.
- Output:
  - TA standardization docs (Neural, Skeletal, Cardiac).
  - Long-term: cardiac treatment database linked to final project.

**Materials ingested**

- Physio.zip (course dump):
  - Global docs, lab manuals, PowerLab notes.
  - System-specific rubrics and final project rubric.
  - Cardiac station PDFs, hypothesis sheets, final project description.
  - Example graded lab reports.
- Six additional neuro/physiology PDFs (dissertation-adjacent knowledge base).

**Canonical neural physiology framework**

- Neurons and excitability (resting potential, depolarization/hyperpolarization).
- Action potentials and conduction (all-or-none, saltatory conduction).
- Synapses (presynaptic Ca²⁺, vesicle fusion, EPSPs/IPSPs, ionotropic vs metabotropic).
- Neurotransmitters and termination (ACh, catecholamines, NO, endocannabinoids).
- Reflex arcs (segmental vs intersegmental).
- Autonomic patterns (preganglionic ACh, parasympathetic ACh, sympathetic NE with exceptions).

**Decisions**

- TA-facing docs will:
  - Use a standardized neural physiology vocabulary derived from course texts + PDFs.
  - Be structured with a cross-system template (to be finalized).
- Cardiac treatment database will:
  - Be systematically tied to rubrics and cardiac station prompts.

**Open questions**

- First concrete neural deliverable:
  - TA glossary vs lab-specific neural guide.
- Cross-system TA template sections.
- Cardiac database schema:
  - Fields such as treatment, mechanism, HR effect, amplitude effect, species/context, station mapping, etc.

---

## 6. DECISION LOG (HIGH-LEVEL)

*(Short bullets; details live in chapter/project files and mirror.)*

- **Canonical BBBB format**: Markdown (`BBBB.md` in private repo), not Google Doc.
- **Mirror**: `Dissertation_BBBB_Mirror` with `BBBB_public.md` and structured subfolders.
- **Chapter 1**:
  - Relative brain = PGLS residuals; absolute vs relative treated as parallel but unequal emphases.
  - Diet = EltonTraits → Diet_5Cat; drop only n = 1 categories.
  - Mass = AVONET BodyMass-Value (`Mass_canonical`).
  - Single MCC BirdTree phylogeny used across analyses.
  - Tables are numeric authority; text is interpretive and cautious.
- **BIOL 4161 project**:
  - Treat TA standardization + cardiac database as one structured project.
  - Use canonical neurophysiology framework for all TA docs.

---

## 7. OUTSTANDING QUESTIONS

- Chapter 1:
  - Exact wording of causal-inference caveats.
  - How prominently to feature BM/OU/EB results.
  - Final location/format of metadata file (appendix vs supplement vs both).

- BIOL 4161:
  - Glossary-first vs neural-lab-guide-first.
  - Final design of TA template.
  - Final cardiac treatment database schema.

---

## 8. TASK POINTERS

Tasks themselves will live in:
- Private repo task files, and/or
- Mirror `/tasks/` (`inbox.md`, `this_week.md`, `log_20XX.md`).

At any time:
- Promote key tasks from here into `/tasks/this_week.md` in the mirror.
