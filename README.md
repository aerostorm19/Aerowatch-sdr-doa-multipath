# Aerowatch

# Quantifying Error Sources in Low-Cost SDR-Based Radio Direction Finding

## Project Summary

Radio direction finding determines where a transmitter is by rotating a directional antenna and recording signal strength at each angle. The resulting curve is not the answer directly — it is a blurred version of it, blurred by the antenna's own beam shape. Estimation algorithms mathematically reverse this blur to recover the true bearing. Critically, this only works if the beam shape is known accurately; an incorrect assumed shape produces not just poor accuracy but spurious peaks that closely resemble genuine signals.

A previous implementation at this institution demonstrated the technique using inexpensive SDR hardware and reported bearing errors of roughly 15–20 degrees. However, at least seven factors could have caused those errors — an unmeasured antenna pattern, possibly insufficient angular sampling, manual rotation, unlogged receiver gain settings, indoor reflections, deviations from the underlying theory, and manual selection of the reported result. All were present simultaneously and none was measured independently. The error therefore cannot be explained, and no proposed improvement can be meaningfully evaluated against it.

This project builds a controlled experimental platform in which these factors can be varied independently: a motorized rotator with encoder feedback, acquisition software logging complete measurement metadata, a corrected processing pipeline, and automated quality-control checks.

The central experiment applies four direction-finding algorithms to both carefully collected and deliberately degraded data, then compares the improvement attributable to better measurement against that attributable to better algorithms. Supporting experiments measure the true antenna pattern, determine the required angular sampling density, and quantify repeatability and reproducibility. A machine learning analysis ranks which logged acquisition parameters best predict bearing error and estimates confidence for individual measurements.

The outcome is a quantified error budget identifying which system components dominate bearing accuracy — replacing trial-and-error improvement with evidence about where engineering effort actually matters.
## Contributors:-
### Gaurav Kumar Singh
### Abhijit Rai
hello world
