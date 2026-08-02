# Project Definition

## Project Title

**Quantifying Error Sources in Low-Cost SDR-Based Radio Direction Finding: A Controlled Experimental Framework**

---

## Project Definition

This project aims to design and develop a controlled experimental framework for measuring **why** low-cost Software Defined Radio (SDR) based direction finding systems make the errors they do.

Current SDR direction-finding systems can estimate the direction of a radio transmitter using a rotating directional antenna and signal processing algorithms. Many different algorithms have already been proposed in the literature, and newer algorithms continue to be developed. However, when these algorithms are tested on inexpensive SDR hardware, their performance is typically reported as an aggregate accuracy figure — usually RMS error — in which systematic bias and random variance are combined into a single number. This matters, because the two point to completely different causes: bias generally indicates calibration or modelling error, while variance indicates noise. Reporting them together conceals exactly the information needed to diagnose the system.

The measured error could originate from:

- inaccurate antenna models,
- insufficient angular sampling,
- mechanical rotation errors,
- receiver gain variations,
- multipath reflections,
- noise,
- or the estimation algorithm itself.

Since all of these factors occur simultaneously in a typical experiment, it becomes impossible to determine whether improving the algorithm actually improves the system.

This project develops a controlled experimental platform capable of varying the dominant error sources independently and measuring their contribution to the final bearing error separately.

Rather than proposing another direction-finding algorithm, the project focuses on understanding the complete measurement process and identifying which parts of the system deserve future research effort.

---

## Problem Statement

Low-cost SDR-based direction finding systems have become increasingly popular because they are inexpensive and easy to build. However, despite many published direction estimation algorithms, there is still no systematic method for determining **why** these systems produce inaccurate bearing estimates when implemented on practical hardware.

Existing research generally evaluates only the final bearing or localization accuracy under laboratory conditions — anechoic chambers, laboratory-grade instrumentation, and carefully measured antenna patterns. It does not separate errors caused by hardware, antenna modelling, calibration, data acquisition, or signal processing algorithms, and it does not report bias separately from variance.

As a result, researchers often compare algorithms on datasets whose measurement quality is unknown. If one algorithm performs better than another, it is impossible to determine whether the improvement comes from the algorithm itself or from differences in the measurement process.

Without understanding these error sources, future improvements become largely trial and error.

Therefore, there is a need for a standardized experimental framework that can isolate, measure, and quantify the contribution of the dominant error sources in SDR-based direction finding systems.

---

## Aim of the Project

To design and develop a reliable experimental framework that identifies and quantifies the major sources of error in low-cost SDR-based radio direction finding systems, enabling fair evaluation of direction-finding algorithms and improving the reliability of future SDR localization systems.

**Definition of "reliable" as used in this project:** bearing error measured over the full 360° azimuth range, with systematic bias reported separately from RMS error, at a stated operating range and signal-to-noise ratio. This definition is used consistently throughout, and all accuracy claims are made against it.

---

## Why is this Project Needed?

Current research focuses primarily on developing better algorithms.

However, very little work investigates whether the errors observed in practical systems actually originate from the algorithms or from the measurement hardware itself.

If the measurement system is inaccurate, then even the best algorithm cannot produce accurate results.

This project attempts to answer a more fundamental question:

> **Do the errors observed in practical low-cost direction finding systems originate primarily in the estimation algorithms, or in the measurements supplied to them?**

This question is posed neutrally and deliberately. The project's working hypothesis is that measurement quality dominates, but the experimental design permits the opposite conclusion, and that outcome would be equally reportable and equally useful — it would indicate that algorithm development on low-cost hardware is well founded, which is currently assumed rather than demonstrated.

---

## Research Objectives

The project has the following objectives:

**O1.** Develop an automated SDR-based data acquisition system for direction finding, with complete metadata logging for every recorded measurement.

**O2.** Build a motorized rotating antenna platform capable of accurate and repeatable angular measurements, reporting true measured angle rather than commanded angle.

**O3.** Measure the real antenna radiation pattern in its installed configuration, instead of relying on simulated antenna models, and quantify the difference between the two.

**O4.** Develop a corrected signal-processing pipeline based on established theoretical models, including automatic model-order selection and a blind peak-selection rule fixed in advance of data collection.

**O5.** Introduce automated quality-control checks to ensure that only reliable measurements are analyzed. These include a forward/reverse sweep consistency check, in which the antenna is rotated clockwise and then anticlockwise over the same angular range; any disagreement between the two indicates mechanical or thermal error in the apparatus rather than any physical property of the signal, making this the most direct diagnostic available for rotation-related error.

**O6.** Experimentally separate and quantify the effect of the following on final bearing accuracy:
- antenna pattern modelling (simulated versus measured),
- angular sampling density,
- overall acquisition and calibration quality (treated as a single combined factor),
- and estimation algorithm choice.

**O7.** Evaluate multiple direction-finding algorithms under identical experimental conditions, with systematic bias and RMS error reported separately for every configuration. Separating these two quantities is essential to the project, since bias and variance indicate different underlying causes and call for different engineering responses.

**O8.** Measure system repeatability (same session, same operator, same site) and reproducibility (different days, different operators, with recalibration between sessions) and report both.

**O9.** Use machine learning to identify which acquisition parameters are the strongest predictors of bearing error, and to estimate a confidence value for each individual measurement.

**O10.** Release the experimental framework, datasets, software, and documentation so that future researchers can reproduce and extend the work.

---

## Research Questions

The project attempts to answer four primary questions:

### RQ1

**How much of the final bearing error is caused by the measurement process, and how much is caused by the estimation algorithm?**

Answered by a controlled two-factor experiment in which acquisition quality and estimation algorithm are varied independently, with the effect of each reported in degrees, and with bias and RMS error reported separately.

### RQ2

**How much does the measured antenna radiation pattern differ from the simulated model, and how much bearing error is attributable to that difference?**

Answered by measuring the installed antenna pattern, computing its normalized residual against the simulated pattern used in previous work, and running the identical processing pipeline with each pattern in turn.

### RQ3

**How many angular measurements are required before the system becomes limited by noise rather than by insufficient sampling?**

Answered by recording a single high-resolution angular sweep and decimating it offline to synthesize several lower sample counts, then plotting bearing error against the number of samples relative to the measured angular bandwidth of the antenna pattern.

### RQ4

**Which acquisition parameters most strongly influence bearing accuracy, and can they be used to predict the reliability of an individual measurement?**

Answered by training a tree-based model on the logged acquisition metadata, reporting cross-validated feature importances, and evaluating a per-measurement confidence estimate on its calibration rather than on accuracy alone.

---

## Scope of the Project

This project focuses on a **single-node** SDR direction finding system.

**It includes:**

- SDR hardware integration
- automated antenna rotation
- antenna characterization
- signal acquisition
- quality control
- algorithm comparison
- experimental error analysis
- machine learning based error prediction

**The project does not attempt to build:**

- a complete localization system,
- multi-node triangulation,
- real-time tracking,
- tactical deployment,
- RF fingerprinting,
- or a new direction-finding algorithm.

**The project does not claim:**

- accuracy competitive with commercial direction-finding equipment, which achieves substantially better performance instantaneously and at substantially higher cost,
- a deployable or field-hardened system,
- or a complete accounting of every error source. The decomposition covers the dominant sources listed in O6; remaining sources are logged and reported but not independently controlled.

These are considered future extensions.

---

## Expected Outcome

At the end of the project, we expect to deliver:

- an automated SDR direction-finding experimental platform,
- a fully characterized antenna measurement system, including the measured installed radiation pattern and its deviation from the simulated model,
- a corrected processing pipeline verified against established theory,
- an experimental comparison of multiple direction-finding algorithms under identical conditions, with bias and RMS error reported separately,
- a quantified error budget showing the contribution of the **dominant** system components to bearing error,
- measured repeatability and reproducibility figures,
- a publicly reproducible dataset with complete metadata,
- and a machine learning model capable of predicting the confidence of individual bearing estimates.

---

## Note on Interpretation of Results

Two distinct classes of evidence are produced by this work, and they are reported as such.

**Causal evidence** comes from the controlled experiments, in which factors are deliberately varied while others are held constant. This applies to acquisition quality, angular sampling density, antenna pattern source, and algorithm choice.

**Associational evidence** comes from the machine learning analysis, which ranks logged parameters by their observed relationship with bearing error. Feature importances are correlational, are computed on a limited dataset, and are reported with cross-validated confidence intervals. They indicate where further controlled investigation is warranted; they are not presented as causal findings.

Keeping this distinction explicit is a deliberate part of the methodology, since conflating the two is a common weakness in applied machine learning work.
