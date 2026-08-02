# Measuring the Error Budget of Low-Cost Single-Antenna Direction Finding

**Final Project Proposal**

| | |
|---|---|
| **Team** | Two students |
| **Duration** | Six months |
| **Scope** | Bearings only, single node, outdoors |
| **Band** | 700–800 MHz |

> Values marked **[TBD]** are fixed after Experiment E1. Several thresholds depend on the measured antenna pattern and cannot honestly be stated before that measurement exists.

---

## Table of Contents

1. [Definitions of Terms](#1-definitions-of-terms)
2. [Research Statement](#2-research-statement)
3. [Problem Statement](#3-problem-statement)
4. [Research Questions and Hypothesis](#4-research-questions-and-hypothesis)
5. [Objectives](#5-objectives)
6. [What Is Being Built](#6-what-is-being-built)
7. [Experiments](#7-experiments)
8. [The Machine Learning Component](#8-the-machine-learning-component)
9. [Timeline and Division of Labour](#9-timeline-and-division-of-labour)
10. [Success Criteria](#10-success-criteria)
11. [Scope Boundaries](#11-scope-boundaries)
12. [Candidate Titles](#12-candidate-titles)
13. [Items to Settle Before Commencement](#13-items-to-settle-before-commencement)

---

## 1. Definitions of Terms

Terms are defined here on first use and used consistently throughout the document.

### 1.1 System and hardware terms

**Direction Finding (DF)**
Determining the direction from which a radio signal arrives, relative to a known reference direction. Produces a bearing, not a position.

**Direction of Arrival (DoA)**
The angle at which a signal arrives at the receiver. Used interchangeably with "bearing" in this document. Distinguished from *localization*, which requires two or more bearings from separated positions.

**Software Defined Radio (SDR)**
A radio in which tuning, filtering, and demodulation are performed in software rather than in fixed analogue circuitry. The specific device used here is the ADALM-Pluto, built around the Analog Devices AD9363 transceiver.

**Automatic Gain Control (AGC)**
A receiver function that automatically adjusts amplification to keep the output at a consistent level. Beneficial in communications, harmful here: the measurement in this project *is* the variation of received power with angle, and AGC actively suppresses that variation. AGC must be disabled and the gain fixed manually.

**Encoder**
A sensor reporting the true angular position of a rotating shaft. An *absolute* encoder reports position directly at power-on; an *incremental* encoder only reports changes and must be homed. This project requires absolute position, mounted after the gearbox so that it reports the antenna's true angle including all mechanical slack — not the motor's commanded angle.

**Phase centre**
The effective point from which an antenna appears to radiate or receive. If the phase centre is offset from the rotation axis, rotating the antenna changes its distance to the transmitter, producing an amplitude variation indistinguishable from a genuine feature of the antenna pattern.

**Far field**
The distance beyond which the wavefront arriving at an antenna is effectively planar, given approximately by 2D²/λ where D is the largest aperture dimension and λ the wavelength. Measurements taken inside the far-field boundary do not satisfy the assumptions of the signal model.

### 1.2 Signal model terms

**Antenna pattern**
The variation of an antenna's sensitivity with direction. The *magnitude pattern* is denoted G; the *power pattern* is |G|². The theory underlying this project operates on the power pattern.

**Sampling kernel**
The role played by the antenna pattern in this method. The measurement is a convolution of the true angular power distribution with the antenna's power pattern; the pattern therefore acts as the kernel through which angular space is sampled, analogous to a sampling function in time-domain signal processing.

**Angular bandwidth (M)**
The antenna power pattern can be expressed as a finite sum of complex exponentials, a(θ) = Σ aₘ·e^(jmθ), for m from −M to +M. The quantity 2M+1 is the pattern's angular bandwidth. Larger M means a pattern with more angular structure — more lobes, ripple, and asymmetry.

**Angular sampling condition**
Valid deconvolution requires the number of angular samples L to satisfy **L ≥ 2M+1**. Taking fewer samples causes aliasing, and the recovered result is invalid regardless of which estimation algorithm is applied. Determining M for the antenna in use is therefore a prerequisite to interpreting any result.

**Dictionary**
The matrix of candidate antenna-pattern responses, one column per candidate bearing, against which the measurement is fitted. Constructed from the antenna power pattern. A dictionary built from an inaccurate pattern produces *dictionary error*, which manifests as bias and as spurious peaks that resemble genuine emitters.

**Deconvolution**
Recovering the true angular distribution from the measured convolved profile — the mathematical operation of reversing the antenna pattern's blurring effect.

### 1.3 Estimation and statistics terms

**Annihilating Filter (AF)**
A parametric estimator that constructs a polynomial whose roots encode the source directions. Derived from finite-rate-of-innovation sampling theory. Has no theoretical resolution limit in the noiseless case; performance in noise depends critically on a denoising stage.

**Cadzow denoising**
An iterative procedure that alternately truncates a structured matrix to rank K by singular value decomposition and restores its Toeplitz structure by averaging along anti-diagonals. In the source literature this stage is what allows the annihilating filter to approach the Cramér–Rao bound; without it, the same estimator performs one to two orders of magnitude worse.

**FISTA**
Fast Iterative Shrinkage-Thresholding Algorithm. Solves a sparse recovery problem of the form: minimise ½‖Ax − V‖² + λ‖x‖₁. The parameter λ trades sparsity against data fidelity.

**Non-Negative Least Squares (NNLS)**
Least-squares fitting with the constraint that all coefficients are non-negative. Relevant here because power coefficients are physically non-negative and dictionary columns are non-negative. In this regime, non-negativity alone induces sparsity, which raises the question of whether the L1 penalty in FISTA contributes anything additional.

**Regularization parameter (λ)**
The weight applied to the sparsity penalty in FISTA. Conventionally hand-tuned per dataset, which prevents unattended operation. The *discrepancy principle* sets λ so that the residual matches the independently measured noise level, making the choice principled rather than manual.

**Model order (K)**
The assumed number of sources present. Selected automatically here using the **Minimum Description Length (MDL)** criterion applied to the singular value spectrum, rather than being fixed by hand.

**Blind peak selection**
A decision rule that selects which candidate peak is reported as the bearing, using only information available at run time. Required because an estimator that produces several candidates and reports whichever is nearest the known truth has no measurable accuracy.

**Bias and variance**
Two distinct components of error. **Bias** is systematic — the average offset of estimates from truth — and typically indicates modelling or calibration error. **Variance** is random scatter and typically indicates noise. Total error combines both (MSE = bias² + variance). Reporting only aggregate RMSE conceals which is dominant, and the two require opposite engineering responses. **This project reports them separately throughout.**

**Cramér–Rao Bound (CRB)**
The theoretical minimum variance achievable by any unbiased estimator for a given measurement problem. Serves as a diagnostic: an estimator whose variance greatly exceeds the CRB is suboptimal and can be improved algorithmically; one at the CRB with significant residual bias cannot, and the remaining error must be attributed to modelling or calibration.

**Repeatability (σ_r) and reproducibility (σ_R)**
Repeatability is variation under identical conditions — same rig, site, operator, session. Reproducibility is variation across different days, operators, sites, and recalibrations. σ_R is invariably larger and is the practically relevant figure; it is also almost never reported in this literature.

### 1.4 Data and analysis terms

**SigMF**
Signal Metadata Format. An open standard pairing a binary IQ recording with a JSON metadata file, supporting namespaced extensions. Used here rather than a bespoke schema.

**IQ data**
In-phase and Quadrature samples — the complex-valued representation of a received signal at baseband.

**Quality Control (QC) gate**
An automated pass/fail check applied to a recording before it enters analysis, returning a reason code on failure. Distinguished from *calibration*: QC asks whether a measurement is admissible; calibration asks whether it corresponds correctly to physical reality.

**Factorial experiment**
A design in which two or more factors are varied independently across all combinations, allowing the effect of each (the *main effect*) and their interaction to be estimated separately.

**Permutation importance**
A method for ranking feature relevance by measuring the degradation in model performance when a single feature's values are randomly shuffled. Correlational, not causal.

---

## 2. Research Statement

### 2.1 In plain terms

A directional antenna receives more strongly in some directions than others. Rotating it through a full circle and recording signal strength at each angle produces a curve of strength versus angle.

That curve is not the answer. It is a **blurred** version of the answer, and the blur is the shape of the antenna's own beam.

The analogy holds well: photograph a point of light through an out-of-focus lens and the result is a smear, not a point — and the smear's shape is a property of the *lens*, not of the light. The antenna beam is the lens. The measured curve is the smear.

Every direction-finding algorithm in this family — annihilating filter, FISTA, non-negative least squares — performs the same underlying job. It **un-blurs**. Given the lens shape, it mathematically sharpens the smear back toward a point and reads off the angle.

The failure mode follows immediately. **If the assumed lens is wrong, sharpening does not merely blur the answer — it manufactures detail.** Extra peaks appear that are indistinguishable from real emitters but are artefacts of the mismatch between the assumed lens and the real one.

The predecessor work at this institution observed exactly such extra peaks and attributed them to reflections from the room's walls and a metallic chair. That is a reasonable hypothesis. But the antenna pattern used in that work was computed in MATLAB from caliper measurements of the antenna's dimensions — never measured on the assembled system. Wrong-lens artefacts and genuine reflections produce the same signature. No experiment in that work distinguishes them, and none in the literature it builds on does either.

**This project builds the minimum apparatus needed to tell them apart, and then tells them apart.**

### 2.2 In one sentence

> We are measuring whether the bearing error in low-cost SDR direction finding originates in the estimation mathematics or in the measurement chain that feeds it — because the two have never been separated, and until they are, no further algorithm work on this platform can be interpreted.

---

## 3. Problem Statement

### 3.1 Background

Single-antenna power-measurement direction finding is a mature technique.

| Work | Year | Contribution |
|---|---|---|
| Lie, Blu and See | 2010 | Signal model, spectral transformation, annihilating-filter solution, Cadzow denoising, Cramér–Rao bound |
| Kumar et al. | 2018 | Extension to multipath via restoration–separation–estimation |
| Akhdar et al. | 2008, 2009 | Deconvolution formulation and CLEAN-based denoising |

The mathematics is settled and is not in dispute.

Every one of these works validates under **laboratory conditions**: anechoic chambers, vector network analysers, turntable-measured antenna patterns, laboratory-grade receivers. Each reports accuracy as aggregate RMSE, against simulation or a small number of controlled runs.

None reports bias separately from variance. None decomposes error across the acquisition chain. And none asks what happens when the same technique is built from commodity hardware, where the antenna pattern is imperfectly known and the receiver costs fifteen thousand rupees.

### 3.2 The immediate instance

The predecessor implementation (Sharma and Suryavanshi, *"Using Small SDRs to Rapidly Validate Complex Pipelines: A Case Study in Radio Direction Finding"*) demonstrated three estimators running end-to-end on real RF data captured with a pair of ADALM-Pluto SDRs and a hand-rotated directional antenna. It established feasibility, which was its stated purpose.

On real data, against a true bearing of **310°**:

| Estimator | Reported peak | Error |
|---|---|---|
| Annihilating Filter | 293.6° | 16.4° |
| Beamforming | 329.0° | 19.0° |
| FISTA | 322.0° | 12.0° |

A spread of 35° across the three pipelines.

**That result cannot be interpreted**, because at least seven candidate causes were simultaneously present and mutually confounded.

#### Cause 1 — Dictionary error

The antenna pattern used as the estimator's dictionary was simulated, not measured. Its deviation from the installed physical pattern is unknown.

#### Cause 2 — Sampling adequacy unknown

Eighteen angular samples were taken. Valid deconvolution requires L ≥ 2M+1. **M was never measured.** The reference implementation in the source literature required M = 8 for a modest GSM antenna, implying a minimum of seventeen samples for that antenna alone. Whether eighteen samples were adequate here is unknown, in a direction that cannot be determined without measuring M.

#### Cause 3 — Angle error

Rotation was by hand, so assumed angles and true angles differ by an unrecorded amount.

#### Cause 4 — Acquisition state unlogged

Receiver gain state, AGC configuration and noise floor were not recorded. Absolute power is therefore not recoverable from the stored data. **If AGC was active, the power-versus-angle profile — which is the entire measurement — was flattened by the receiver before it reached the algorithm.**

#### Cause 5 — Geometry

Transmitter and receiver were one metre apart indoors, marginally within the far-field boundary, in a room where reflected paths arrive with almost no excess path loss relative to the direct path. Reflections were not a perturbation on the measurement; they were comparable to it.

#### Cause 6 — Pipeline deviations from the cited theory

Three deviations, each independently consequential.

**(a) Wrong domain.** The measurement is taken in RMS voltage and deconvolved against the magnitude pattern G. The published theory operates in received power against the power pattern |G|². Linearity in that theory arises from uncorrelated sources superposing *in power*; the square root breaks it for any multi-source or direct-plus-reflection case.

A second consequence: noise contributes a deterministic additive bias of **2σ²** to each power measurement. In the power domain this is measurable and subtractable. Under the square root it becomes an unremovable, nonlinear, angle-dependent bias — largest at the pattern nulls, which is precisely where the informative high-angular-frequency content lies.

**(b) Cadzow denoising omitted.** This is the stage the source paper credits for its estimator reaching the CRB; without it, the same estimator sits one to two orders of magnitude above the bound across the entire SNR range. The predecessor's conclusion that FISTA is the strongest estimator was therefore reached by comparing a tuned FISTA against a deliberately incomplete annihilating filter.

**(c) Crude deconvolution.** An epsilon-regularised Fourier ratio is used in place of a least-squares solve. The two coincide only when the sample count exactly equals the pattern bandwidth and sampling is uniform. The ratio form discards the noise-averaging benefit of angular oversampling, cannot accommodate the non-uniform sampling a real encoder produces, and regularises by an arbitrary constant rather than from measured noise variance.

#### Cause 7 — Ground-truth-informed reporting

Three peaks were produced per estimator, and the peak nearest the known truth was reported. Without a blind decision rule, **the system has no measurable accuracy** — it produces a candidate set that happens to contain the answer.

### 3.3 The consequence

Because all seven vary together and none is measured, no further algorithm work on this platform can be interpreted.

If a new estimator beats FISTA by two degrees on this data, that difference is indistinguishable from a dictionary artefact, an aliasing artefact, or a gain-state artefact. The natural next step — try better algorithms — is unavailable until the measurement chain is characterised.

### 3.4 Statement

> For low-cost single-antenna direction finding, the contributions of antenna-pattern error, angular sampling adequacy, acquisition-chain discipline and estimator choice to total bearing error have never been separately measured. This project builds the minimum apparatus required to measure them, and measures them.

---

## 4. Research Questions and Hypothesis

### 4.1 Primary research question

> **In a low-cost SDR direction-finding system, how much bearing error is attributable to acquisition and calibration quality, and how much to the choice of estimation algorithm?**

**Answered by:** a two-factor experiment (E3) reporting RMSE and bias separately for each cell, with the two main effects compared directly, in degrees, with confidence intervals.

### 4.2 Supporting question 1

> **What is the angular bandwidth of the receive antenna in its installed configuration, and how many angular samples are required before the estimator becomes noise-limited rather than aliasing-limited?**

**Answered by:** fitting the measured power pattern to a finite sum of complex exponentials to obtain M (E1), then decimating a single high-resolution sweep to synthesise several sample counts and plotting error against the ratio L/(2M+1) (E2).

### 4.3 Supporting question 2

> **Does restoring the pipeline to the published theory — power-domain measurement, 2σ² noise-bias subtraction, Cadzow denoising, least-squares deconvolution — change which estimator performs best?**

**Answered by:** running corrected and uncorrected pipelines on identical data (E3). This is a direct and inexpensive test of a stated finding in the predecessor work; the prior expectation is that it overturns it.

### 4.4 Hypothesis

> Acquisition and calibration quality dominate estimator choice in determining bearing error, by a margin of several degrees.

**Falsification criterion:** rejected if the algorithm main effect in E3 exceeds the data-quality main effect by more than the pooled standard error.

The design is constructed so that **both outcomes are informative**. If the hypothesis is rejected, the finding is that estimator development on this platform is well founded after all — equally worth reporting, and equally unreported at present.

---

## 5. Objectives

### O1 — Corrected reference pipeline
Restore power-domain measurement with 2σ² noise-bias subtraction and the |G|² dictionary. Replace the Fourier-ratio deconvolution with regularised least squares. Implement Cadzow denoising. Implement automatic model order by MDL. Implement blind peak selection under a rule fixed in writing before any data is examined. Retain the predecessor pipeline unmodified as the baseline arm.

**Deliverable:** pipeline verified against the CRB in simulation; predecessor pipeline preserved for comparison.

### O2 — Motorised acquisition rig
Azimuth rotator with absolute angle knowledge, carrying the receive antenna with its phase centre on the rotation axis, with cable management that does not twist.

**Deliverable:** a rig whose reported angle is trustworthy to a measured tolerance.

### O3 — Acquisition software with complete metadata
Rotator control, settle-and-capture sequencing, fixed manual gain with AGC disabled, paired transmitter-off noise captures at identical gain bracketing every sweep, SigMF output with a direction-finding metadata extension.

**Deliverable:** unattended 360° sweeps with complete provenance on every capture.

### O4 — Measured installed antenna pattern
Two-degree resolution across 700–800 MHz, measured with mast, mount, rotator and cabling present, fitted to obtain M.

**Deliverable:** measured pattern; fitted M with selection curve; normalised residual against the predecessor's simulated pattern.

### O5 — Quality control
Six automated checks with pass/fail gates and reason codes, applied before data enters analysis.

**Deliverable:** QC framework with a measured detection rate on seeded faults and a documented false-rejection rate.

### O6 — Error decomposition
Execute the factorial experiment and report main effects.

**Deliverable:** the primary result — acquisition-quality effect and algorithm effect, both in degrees, with confidence intervals.

### O7 — Sampling adequacy characterisation
Determine the empirical transition from aliasing-limited to noise-limited operation.

**Deliverable:** error as a function of L/(2M+1).

### O8 — Repeatability and reproducibility
Both measured and reported as a pair.

**Deliverable:** σ_r within session; σ_R across days, operators and recalibrations.

### O9 — Supporting ML analysis
Tree-based model over logged acquisition metadata to rank which quality metrics predict bearing error, plus a per-sweep confidence predictor.

**Deliverable:** cross-validated feature importances corroborating or contradicting E3; a calibrated reliability score per bearing.

### O10 — Release
SigMF dataset with measured patterns, calibration records, QC reason codes and surveyed truth; reference implementation; documentation sufficient for independent regeneration of figures.

**Deliverable:** versioned, hashed, citable dataset and code.

---

## 6. What Is Being Built

**Four artefacts. Deliberately no more.**

### 6.1 The rig

Motorised azimuth rotator with an absolute encoder, or a stepper with homing switch and independently verified step accuracy. Tripod or mast. Receive antenna mounted with its phase centre on the rotation axis — an offset modulates range during rotation and produces an amplitude taper indistinguishable from genuine pattern structure. Cable routing that does not twist across a full revolution.

This does not need to be elegant. It needs to be **repeatable** and to report its **true** angle rather than its commanded angle.

> **Antenna note.** We do not commit to a high-gain, smooth-beam antenna by default. This method uses the antenna pattern as a *sampling kernel*, and estimation accuracy improves with the pattern's angular bandwidth M — the source literature demonstrates RMSE improving monotonically with M. A clean pencil beam is a low-M kernel and therefore a poor sampling kernel, despite being a good communications antenna. Structured sidelobes, ripple and asymmetry are assets here. Where two or three candidate antennas are available, we measure M for each in E1 and select on measured M rather than on datasheet gain. Selecting an antenna on angular bandwidth rather than gain inverts standard practice and is itself worth reporting.

> **Band note.** Work is at 700–800 MHz outdoors rather than 3 GHz indoors. The predecessor moved to 3 GHz solely to escape near-field effects at one metre separation. Outdoors at ten to thirty metres that constraint vanishes: at 800 MHz a 30 cm aperture is far-field beyond roughly half a metre, and 5 mW over thirty metres is a comfortable link. This removes a workaround, moves the work into the actual target band, and costs nothing.

### 6.2 Acquisition software

Python. Commands the rotator, waits for mechanical settling, captures IQ at fixed manual gain with AGC disabled, records a transmitter-off noise capture at identical gain immediately before and after each sweep, writes SigMF containers with a `df:` metadata extension.

**Metadata per capture, at minimum:**

- Commanded angle
- Encoder angle, recorded at the midpoint of the IQ capture rather than at the command
- Timestamp
- LO frequency and sample rate
- Gain index and gain in dB
- AGC state
- Front-end temperature where available
- Antenna identifier and polarisation
- Calibration table version
- Operator and site identifier
- Surveyed transmitter bearing where known

Roughly 400–800 lines. SigMF is used rather than a bespoke schema.

### 6.3 Corrected processing pipeline

Python or MATLAB. One module per estimator behind a common function signature — that is the entirety of the "modularity" this project requires. Anything more elaborate is scope that will not pay for itself in six months.

**Contents:**

- Power-domain measurement with 2σ² subtraction
- Regularised least-squares deconvolution
- Cadzow denoising by rank-K SVD truncation with anti-diagonal averaging, twenty iterations
- Annihilating filter with root extraction
- Non-negative least squares
- FISTA with λ set from measured noise variance via the discrepancy principle
- MDL model order selection
- Blind peak selection
- Off-grid refinement by local Newton iteration

The predecessor pipeline is retained **unmodified** as the degraded baseline.

### 6.4 QC script

Six checks, not ten. Each returns pass or fail with a reason code; any failure excludes the sweep from analysis, and the exclusion is logged.

| # | Check | Threshold |
|---|---|---|
| 1 | Zero clipped samples; fill factor within range | −6 to −12 dBFS peak |
| 2 | Gain index constant across the sweep | Exact |
| 3 | Per-angle SNR, occupied bins vs guard bins | > [TBD] dB |
| 4 | Forward-versus-reverse sweep agreement | < [TBD] dB |
| 5 | Encoder-versus-commanded residual | < [TBD]° |
| 6 | Noise-capture pair present, matched gain, consistent | < [TBD] dB |

> **On check 4.** Sweep 0°→350°, then 350°→0°. Disagreement between the two is mechanical, thermal or cable-flex in origin and **never** physical — the direction of rotation is not a property of the signal. This is the cheapest high-yield diagnostic available.

---

## 7. Experiments

Four core, two stretch, one supporting analysis. All outdoors at 700–800 MHz against a surveyed transmitter bearing.

### 7.1 E1 — Installed antenna pattern measurement

**Purpose:** obtain the true sampling kernel and quantify the predecessor's dictionary error.

**Method:** open field or rooftop. Transmitter at a surveyed bearing at fixed range, ten to thirty metres. Receive antenna rotated through 360° in two-degree steps by our own rig. Repeated at five frequencies across 700–800 MHz. Both polarisations if time permits, one if not.

This yields a *relative installed* pattern measured with mast, mount and cabling present — which is the pattern the estimator actually needs. **No anechoic chamber required.** If chamber access exists, we use it for one absolute reference measurement and compare, but the project does not depend on it.

**Outputs:**
- Measured |G|²
- Fitted M with the residual-versus-order selection curve
- Normalised residual between measured and the predecessor's simulated pattern, in dB
- Comparison of M across candidate antennas

> **Why this comes first:** every subsequent experiment depends on it. If the fitted M implies that eighteen samples were aliasing, that alone is a reportable finding about the predecessor work, obtained by week ten.

### 7.2 E2 — Angular sampling sweep

**Purpose:** answer supporting question 1.

**Method:** record one sweep at two-degree resolution. Decimate offline to synthesise L = 18, 36, 72, 180. Repeat at three SNR levels by stepping transmit attenuation.

One acquisition yields the entire curve, which eliminates between-acquisition variance as a confound and gives the highest information return per hour of field time of any experiment here.

**Outputs:** RMSE and bias against L/(2M+1); the empirical aliasing-to-noise-limited transition point.

### 7.3 E3 — The factorial decomposition

**Purpose:** the primary result. Answers the primary research question and tests the hypothesis.

**Factor 1 — acquisition quality (2 levels)**

| Level | Configuration |
|---|---|
| **Degraded** | AGC enabled, simulated dictionary, commanded angles, no noise reference, no QC gating |
| **Disciplined** | Fixed gain, measured dictionary, encoder angles, noise-referenced, QC-gated |

> The degraded arm must reproduce the predecessor's *actual* configuration faithfully, derived from the week-one audit — not a strawman constructed to lose.

**Factor 2 — estimator (4 levels)**

1. Annihilating filter without Cadzow (the predecessor's version)
2. Annihilating filter with Cadzow
3. Non-negative least squares
4. FISTA with discrepancy-principle λ

**Design:** eight cells. Minimum twenty independent sweeps per cell; more if time permits. All cells run with blind peak selection and automatic model order — no estimator sees ground truth at any point in its decision chain.

**Outputs per cell:** RMSE, bias reported separately, runtime, peak memory.
**Analysis:** both main effects with confidence intervals, plus the interaction term.

> **Design note on the NNLS arm.** Power coefficients are strictly non-negative and dictionary columns are non-negative. In that regime, non-negativity alone induces sparsity, and an L1 penalty degenerates toward a weak linear penalty on the coefficient sum. The prior expectation is that NNLS is competitive with tuned FISTA while having no hyperparameter at all — a useful negative result about the necessity of the sparse-recovery machinery.

### 7.4 E4 — Repeatability and reproducibility

**Purpose:** quantify what "reliable" means for this system, in a way absent from the literature.

**Method:**
- **Repeatability:** N sweeps, same rig, same site, same session, no intervention. Yields σ_r.
- **Reproducibility:** sweeps across at least three different days, at least two operators, with full recalibration between sessions, at more than one site. Yields σ_R.

Both are reported. **The gap between them is the practically relevant figure**, and it is the figure nobody in this literature publishes.

### 7.5 Stretch A — Controlled reflector

**Purpose:** settle the dictionary-versus-multipath ambiguity that every paper in the reference stack asserts without testing.

**Method:** free-field baseline sweep. Introduce a metal sheet at a surveyed bearing and range; sweep. Move the sheet to a second surveyed position; sweep.

**Decision rule, fixed before the data is examined:**

| Observation | Attribution |
|---|---|
| Spurious peak translates with the sheet | Multipath |
| Spurious peak remains stationary | Dictionary error |
| Partial translation | Quantifies the mixture |

Cost: one sheet of metal and one afternoon. **If anything on the stretch list is promoted, this one is promoted** — it produces ground truth for the multipath itself, which is precisely what makes the attribution possible and precisely what the existing literature lacks.

### 7.6 Stretch B — Stationarity penalty

**Purpose:** quantify the architectural limit of mechanical sweeping.

**Method:** gate the transmitter at duty cycles from continuous down to one-eighth at TDMA-like frame timing. Sweep durations from fastest achievable to sixty seconds. Software-only on the transmit side.

**Output:** bearing degradation as a joint function of duty cycle and sweep duration.

This produces the number that bounds the rotating-antenna architecture for bursty emitters, and is the evidence base for any successor-architecture decision — switched-element or multi-mode pattern diversity, both of which preserve the single-receiver signal chain while making the measurement effectively instantaneous.

### 7.7 E5 — Supporting ML analysis

Detailed in Section 8. Uses only data collected in E1 through E4; requires no additional field time.

---

## 8. The Machine Learning Component

### 8.1 The shape of the data, first

By the end of the campaign we will hold roughly **300–600 sweeps**. Each contributes one bearing estimate, one known truth, and ten to fifteen logged numbers.

That is a table of approximately **500 rows by 15 columns**. Small, tabular, structured.

This determines everything that follows. Deep learning on raw IQ requires tens of thousands of labelled examples. What we have is exactly the regime where random forests and gradient boosting perform well and neural networks do not. Any proposal here that begins with a CNN has not looked at the row count.

### 8.2 E5a — Which quality metrics actually predict bearing error

This is the component we build. It is not ML bolted onto a DSP project; it is the same research question approached from a second, independent direction.

**Features per sweep** (all logged anyway):

- Median and minimum per-angle SNR
- Forward–reverse mismatch, dB
- Encoder residual RMS
- Gain index
- Clip fraction
- Noise floor and its drift across the sweep
- Front-end temperature and its drift
- Carrier frequency offset drift
- Singular-value spread of the Toeplitz matrix
- MDL-selected model order
- Peak-to-second-peak ratio of the angular spectrum
- Residual norm after fit
- Sweep duration
- Site identifier
- Antenna identifier

**Target:** absolute bearing error against surveyed truth.

**Model:** `RandomForestRegressor` or `GradientBoostingRegressor`. Roughly thirty lines of scikit-learn. Permutation importances reported with cross-validated confidence intervals, using **grouped cross-validation by session** so that sweeps from one session cannot appear in both train and test folds — without grouping, temporal correlation within a session inflates performance and distorts importances.

**Why it earns its place:** it reuses data already collected, needs no extra field time, sits in the correct data regime, and genuinely complements E3. The factorial gives *causal* effects for the two factors deliberately controlled. The ML gives *associational* rankings across all fifteen merely logged. Together they answer a broader question than either alone.

> **Honesty requirement, to be stated in the writeup.** Feature importance is correlational, not causal. With roughly 500 rows and correlated features, importances are unstable and must be reported with cross-validated intervals. The factorial is the causal evidence; this is the exploratory map indicating where to look next.

### 8.3 E5b — Per-sweep confidence prediction

Same features, different target: a binary classifier for "bearing error below [TBD] degrees," or a quantile regressor producing a predictive interval.

**Why this matters beyond the thesis:** multi-node triangulation requires a weight per bearing. A bearing of unknown reliability is close to useless for fusion; two bearings with known σ produce a proper error ellipse. This is the cleanest bridge from the six-month project to the localization work that follows, and it is the version of "future work" a reviewer will believe, because the interface would already exist and be calibrated.

**Evaluation:** report **calibration**, not just accuracy. The fraction of sweeps whose true error falls inside the predicted interval should match the nominal level. A confident-but-miscalibrated predictor is worse than none.

### 8.4 The data-collection consequence — plan for this from week one

If the QC gates work as intended, most retained sweeps will pass and cluster at low error, leaving too few high-error examples for any model to learn from.

**We deliberately keep the failures.** Sweeps are run with AGC enabled, with injected encoder offsets, at low SNR, with deliberate calibration staleness. They are logged fully, excluded from the DoA analysis, and fed to the model.

The degraded arm of E3 does double duty here: it is exactly the failure population the classifier needs. The metadata schema is designed in week five with E5 in mind, so nothing has to be retrofitted or re-collected.

### 8.5 What is deliberately excluded, and why

**End-to-end neural DoA regression** — mapping the power curve directly to a bearing. Four reasons, any one sufficient:

1. Trained on simulation, it learns the simulator's quirks rather than the physics and will not transfer.
2. It has no error bound, so it cannot be compared to the CRB — and the CRB comparison is the most credible validation this project has.
3. It fails *silently*, returning confident nonsense where classical estimators degrade visibly.
4. **It destroys the thesis.** The entire contribution is decomposing where error originates; an uninterpretable component means error can now hide inside a box we cannot open.

**Unrolled FISTA (LISTA).** Elegant — replace a thousand iterations with fifteen learned layers — but it optimises something that is not constrained here. FISTA on this problem is roughly 130,000 multiply-accumulates per iteration; even a thousand iterations is well under 100 ms on modest hardware, while one mechanical sweep is seconds. The latency budget is overwhelmingly mechanical. Good follow-on paper, wrong project.

**Learned residual correction to the antenna pattern.** Genuinely promising — keep the measured pattern, let a low-capacity model absorb the residual from mast, tripod and nearby scatterers. But it is a research project in itself, risks overfitting one site, and muddies a clean story. Future work unless comfortably ahead by week eighteen.

### 8.6 Effort and placement

One to two weeks of Student B's time in weeks 22–26, on data already in hand. Framed as a supporting analysis, not a headline.

The sentence it should produce:

> "A tree-based model trained on logged acquisition metadata identified encoder residual and per-angle SNR as the strongest predictors of bearing error, consistent with the factorial finding that acquisition quality dominates estimator choice."

That is ML corroborating the main result through an independent route — ML doing real work rather than ML included because it is expected.

---

## 9. Timeline and Division of Labour

**Student A** owns hardware and acquisition. **Student B** owns DSP and analysis. Both attend every field experiment, because two people are needed and because it halves the transcription errors.

### Weeks 1–4

| Student | Tasks |
|---|---|
| **A** | Procure and build the rotator; verify step accuracy, backlash and phase-centre alignment |
| **B** | Audit predecessor datasets for AGC state and gain logging; reproduce predecessor figures from raw data; implement corrected pipeline; verify CRB convergence in simulation |

> **Critical week-one item: the AGC audit.** If AGC was enabled, that data becomes the degraded arm, which is convenient. If disabled, the degraded arm must be constructed deliberately — cleaner, but slightly more work. Either answer is needed before week four.

### Weeks 5–8

| Student | Tasks |
|---|---|
| **A** | Acquisition software; SigMF writer with `df:` extension designed with E5 features in mind; conducted loopback calibration with splitter and step attenuator over 40 dB |
| **B** | Estimator library complete; blind decision layer complete |

### Weeks 9–12

**Together:** E1, the pattern measurement.
**B, in parallel:** fit M; compute the dictionary residual.

> **Gate:** if fitted M implies eighteen samples were aliasing, that is a reportable finding independent of everything downstream.

### Weeks 13–16

| Student | Tasks |
|---|---|
| **A** | QC script; construct and validate the degraded-arm configuration |
| **B** | Analysis scripts written and tested against synthetic data, so E3 analysis is ready before E3 runs |

### Weeks 17–21

**Together:** E2, E3, E4. Stretch A if E3 completes early.

### Weeks 22–26

**B:** E5 analysis.
**Both:** writing, dataset packaging, reproducibility check.

### Risk concentration

Weeks 1–8 carry the schedule risk. If the rig is not producing reliable data by week ten, we cut **E4 first, then E2, and protect E3**.

**E3 alone is a paper.**

---

## 10. Success Criteria

Every criterion below can be failed, which is what makes passing meaningful.

- [ ] Fifty consecutive unattended sweeps with zero operator interventions and complete metadata on every capture
- [ ] Measured M reported with its fit residual; measured-versus-simulated dictionary residual reported in dB
- [ ] Encoder residual below [TBD]°; forward–reverse hysteresis below [TBD] dB
- [ ] Corrected pipeline converges to the CRB in simulation above approximately −5 dB SNR
- [ ] QC gates detect at least 90% of deliberately corrupted captures in a seeded-fault test, with false-rejection rate reported
- [ ] E3 main effects reported in degrees with confidence intervals; hypothesis accepted or rejected against the stated criterion
- [ ] σ_r and σ_R both reported
- [ ] E5 feature importances reported with grouped cross-validation and confidence intervals; confidence predictor reported with calibration, not accuracy alone
- [ ] A third party regenerates one named figure from the released dataset using only the documentation

---

## 11. Scope Boundaries

### Explicitly out of scope

Stated up front so that nobody discovers it later:

- Multi-node localization and triangulation
- Real-time operation
- Tactical or operator UI
- Time-Difference-of-Arrival (TDoA)
- RF fingerprinting and signal classification
- The 5 km link
- Independent cross-validation against a second DF system
- Tracking
- Any deployment claim

### Explicitly not claimed

- A new estimator
- A deployable system
- Competitive accuracy against commercial correlative interferometers, which achieve sub-two-degree RMS instantaneously and will continue to

> Scope discipline stated up front reads as competence. Overreach discovered under questioning reads as the opposite.

---

## 12. Candidate Titles

1. *Where Does the Error Come From? An Error-Budget Study of Low-Cost Single-Antenna Direction Finding*
2. *Calibration or Algorithm? Quantifying the Dominant Error Sources in SDR-Based Direction Finding*
3. *Measuring the Sampling Kernel: Antenna Angular Bandwidth and Sampling Adequacy in Rotating-Antenna Direction Finding*
4. *An Automated Testbed for Controlled Comparison of Direction-of-Arrival Estimators on Low-Cost SDR Hardware*

> Titles 1 and 2 are contingent on the hypothesis surviving E3. If the algorithm factor dominates, the corresponding title inverts to something like *"Estimator Choice Dominates: Revisiting the Engineering-versus-Algorithm Trade-off in Low-Cost Direction Finding."* **No title is committed before the data exists.**

---

## 13. Items to Settle Before Commencement

### 13.1 Removal of localization from the six-month scope

The argument: bearings with a characterised error budget and a stated σ are the correct input to a fusion stage; bearings with unknown bias are not, because a measurement whose uncertainty has never been measured cannot be sensibly weighted. **Doing the error budget first is the shortest path to localization, not a detour from it.** E5b makes this concrete by delivering the per-bearing confidence that fusion actually consumes.

### 13.2 The week-one AGC audit

Determines the structure of the degraded arm in E3 and cannot wait. This is a single-line check in the predecessor's acquisition code and determines roughly two weeks of downstream schedule.

---

## Appendix — Notation Summary

| Symbol | Meaning |
|---|---|
| θ | Bearing / direction of arrival |
| G | Antenna magnitude pattern |
| \|G\|² | Antenna power pattern (the sampling kernel) |
| M | Half the angular bandwidth; pattern expressible over m ∈ [−M, +M] |
| 2M+1 | Angular bandwidth of the power pattern |
| L | Number of angular samples per sweep |
| L/(2M+1) | Sampling adequacy ratio; must be ≥ 1 for valid deconvolution |
| K | Model order — number of sources assumed present |
| σ² | Receiver noise variance, measured from transmitter-off captures |
| 2σ² | Deterministic noise bias on each power measurement |
| λ | Sparsity regularization parameter in FISTA |
| σ_r | Repeatability — within-session standard deviation of bearing |
| σ_R | Reproducibility — across-session standard deviation of bearing |
