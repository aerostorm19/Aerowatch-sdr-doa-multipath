# Aerowatch

**Low-cost SDR-based radio direction finding, built to survive multipath — with a clear path to full emitter localization.**

## About

Aerowatch estimates the direction a radio signal is coming from using a single software-defined radio and a directional antenna — no expensive antenna arrays, no multi-channel receivers. It's built to work in the real world, where signals bounce off walls and buildings before they ever reach you, not just in a clean lab test.

The project started by reproducing and validating three known single-antenna DoA techniques — Annihilating Filter, Beamforming, and Compressive Sensing (FISTA) — against simulated and real RF data. It's now being extended with a Constant Modulus Algorithm (CMA) based restoration stage to actively separate a transmitter's direct signal from its own reflections, and a MUSIC-based subspace estimator, as the foundation for higher-accuracy, multipath-resistant direction finding.

The long-term goal is a modular platform that grows from single-antenna DoA into a multi-receiver system capable of full emitter localization — combining bearing estimates from multiple nodes to pinpoint a transmitter's actual location, not just its direction.

## Status

Actively in development. Core DoA algorithms are validated in simulation; multipath-mitigation and array-based methods are being built out and tested before hardware integration. This is an early-stage research platform, not yet a finished product — progress and limitations are tracked openly as the project evolves.

##Contributors:-
#Gaurav Kumar Singh
#Abhijit Rai
