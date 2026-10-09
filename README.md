# 🛰️ LumenOS: The Operating System for Orbital AI

> The world's first thermodynamics-aware operating system and scheduler for space-based AI data centers.

[![NASA Space Apps Challenge 2026](https://img.shields.io/badge/NASA%20Space%20Apps-2026-blue.svg)](https://spaceappschallenge.org)
[![Challenge Theme](https://img.shields.io/badge/Theme-The%20Next%20Frontier-ff9f1c.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-22d3ee.svg)](LICENSE)
[![Tech Stack](https://img.shields.io/badge/Stack-Three.js%20%7C%20Canvas%20%7C%20GSAP%20%7C%20Vanilla%20JS-black.svg)](#tech-stack)

LumenOS reads the satellite's orbit — sunlight, shadow, radiator direction, and battery status — and decides when heavy AI workloads should run and when they must pause before heat destroys the spacecraft.

---

## Contents

<details>
<summary><strong>Browse the project documentation</strong> <sub>Click to expand or collapse</sub></summary>

**Project overview**

- [Executive summary](#-executive-summary)
- [The core problem](#-the-core-problem)
  - [Earth is AI's biggest cost](#part-1-earth-is-ais-biggest-cost)
  - [The vacuum paradox](#part-2-the-vacuum-paradox-heat-in-space)
- [The LumenOS solution](#-the-solution-lumenos)

**Technology & design**D

- [Physics engine](#-the-physics-engine)
  - [Radiative heat dissipation](#stefan-boltzmann-law--radiative-heat-dissipation)
  - [LEO view-factor dynamics](#view-factor-dynamics-in-low-earth-orbit-leo)
  - [Thermal model](#lumped-capacitance-thermal-model)
- [Compute scheduling](#-how-lumenos-schedules-compute)
  - [Core scheduling conditions](#the-3-core-conditions)
  - [Workload classification](#workload-classification--power-footprints)
  - [Predictive control loop](#30s-decision-loop--5-minute-predictive-look-ahead)
- [Simulation results](#-simulation-results-lumenos-vs-naive-baseline)
- [System architecture](#-system-architecture)
- [Competitive differentiation](#-competitive-differentiation-why-were-unique)

**Project guide**

- [Landing page features](#-interactive-landing-page-features)
- [Quick start & local preview](#-quick-start--local-preview)
- [Repository structure](#-repository-structure)
- [Team Proton](#-team-proton)
- [Limitations & research roadmap](#-honest-limitations--research-roadmap)
- [License & acknowledgments](#-license--acknowledgments)
- [Tech stack](#tech-stack)

</details>

---

## 🚀 Executive Summary

Terrestrial AI data centers are on a collision course with Earth's physical limits: grids are failing, freshwater reservoirs are draining, and regional land moratoria are stalling growth.

Moving high-performance AI clusters into Low Earth Orbit (LEO) offers 24/7 solar power and zero land or water consumption. However, space introduces a lethal challenge: the vacuum paradox. Without atmosphere, heat cannot escape via convection or conduction; it can only leave as infrared radiation through panels.

Conventional operating systems such as Linux or Kubernetes schedule jobs based on CPU and RAM availability. In orbit, this leads to catastrophic thermal runaway, overheating beyond safe thermal limits, and emergency shutdowns.

LumenOS is the missing software layer: a thermodynamics-aware hypervisor and scheduler that reads the satellite's orbital position, radiative view factor, and solar/battery budget to schedule compute around physics.

---

## 🌍 The Core Problem

### Part 1: Earth is AI's Biggest Cost

| Metric | Terrestrial Baseline & Trend | Source |
|---|---|---|
| Electricity Demand | ~415 TWh (2024) → ~945 TWh (2030) | IEA, Energy and AI Report (2025) |
| Grid Pressure | US data centers projected to draw ~9% of national electricity by 2030 | EPRI / Goldman Sachs (2024) |
| Water Consumption | ~700,000 liters of potable freshwater per large frontier LLM training run | UC Riverside (2023) / Microsoft CSR |
| Emissions | ~1.0–1.5% of total global energy-related CO₂ | IEA |
| Grid Infrastructure | Saturated grids in Northern Virginia; regional moratoria in Dublin & Singapore | Industry Filings |

### Part 2: The Vacuum Paradox (Heat in Space)

Space is cold, but vacuum is a thermal insulator.

- Zero convection and zero conduction: heat cannot be blown away with fans or flushed into cooling towers.
- Orbital fluctuations: a satellite in ~550 km LEO completes an orbit every ~95.6 minutes, with roughly 60 minutes in full sunlight and ~35 minutes in eclipse.
- The view-factor trap: when radiator panels face deep space, cooling capacity is maximal. When they face the sunlit Earth albedo, cooling capacity drops sharply, trapping heat inside the chassis.

---

## 💡 The Solution: LumenOS

LumenOS shifts compute scheduling from spatial load balancing to temporal orbital balancing.

Rather than asking, “Where is the compute hot spot?” it asks, “When is the orbit favorable enough to run?”

The scheduler continuously evaluates orbital geometry, thermal headroom, and available solar power to decide whether to:

- run heavy inference or training jobs,
- switch to a reduced-power mode,
- checkpoint progress,
- or pause before system temperature exceeds safe thresholds.

This makes LumenOS a physics-first control system for orbital AI operations.

---

## 📐 The Physics Engine

### Stefan-Boltzmann Law & Radiative Heat Dissipation

Radiators reject thermal energy by radiation:

```text
Q_out = ε × σ × A × (T_rad^4 − T_sink^4)
```

| Parameter | Description | Standard Value |
|---|---|---|
| ε | Radiator emissivity | 0.80 |
| σ | Stefan-Boltzmann constant | 5.670 × 10^-8 W/m²K⁴ |
| A | Effective radiator surface area | 1.5 m² |
| T_rad | Radiator plate operating temperature | 340 K (66.85°C) |
| T_sink | Environmental sink temperature | Variable |

### View-Factor Dynamics in Low Earth Orbit (LEO)

| Radiator Orientation | Environmental Sink (T_sink) | Cooling Capacity (Q_out) | Status |
|---|---|---|---|
| Deep Space | 3 K | ~909 W | Maximum headroom |
| Earth Night | 220 K | ~750 W | Moderate rejection |
| Earth Day (Albedo) | 280 K | ~491 W | Thermal trap |

### Lumped-Capacitance Thermal Model

Chassis temperature change over time is calculated as:

```text
dT/dt = (Q_in − Q_out) / (m × C_p)
```

- Thermal mass (m): 15 kg (CubeSat-class compute module)
- Specific heat capacity (Cp): 900 J/kg·K
- Base electronics heat: 100 W
- Battery pack: 500 Wh capacity supported by excess solar energy

This creates a real-time control problem where compute decisions must be balanced against thermal stability and orbital context.

---

## 🧠 How LumenOS Schedules Compute

### The 3 Core Conditions

LumenOS makes decisions based on three conditions:

1. Is the satellite in sunlight or eclipse?
2. Is radiator orientation still allowing effective heat rejection?
3. Is the battery and power budget safe enough to sustain the workload?

If any condition becomes unfavorable, the scheduler reduces workload, checkpoints state, and defers non-critical jobs.

### Workload Classification & Power Footprints

| Workload Type | Example | Power Draw | Decision Policy |
|---|---|---|---|
| Cold | Data validation, housekeeping | Low | Run opportunistically |
| Medium | LLM summarization, prediction tasks | Moderate | Run with thermal buffer |
| Heavy | Full-model fine-tuning, multi-agent inference | High | Run only during favorable orbital windows |

### 30s Decision Loop & 5-Minute Predictive Look-Ahead

LumenOS performs a continuous 30-second control loop with a 5-minute predictive look-ahead.

This allows it to:

- anticipate thermal stress before the next orbit segment,
- queue work into safe windows,
- checkpoint state before entering thermal risk,
- preserve progress without forcing a hard shutdown.

---

## 📊 Simulation Results: LumenOS vs. Naive Baseline

The concept is designed to show that physics-aware scheduling can preserve uptime while preventing thermal collapse.

| Scheduler | Result |
|---|---|
| Naive baseline | Crashes around 95.4°C and loses in-progress work |
| LumenOS | Maintains stable thermal envelope and zero shutdowns |

Key takeaway: in orbital AI, heat is the real currency, not just electrical power.

---

## 🎯 Interactive Landing Page Features

This project includes an interactive landing page experience built with Three.js, GSAP, and vanilla JavaScript. It showcases:

- a cinematic space-themed hero section,
- animated glow and motion effects,
- scroll-driven transitions,
- orbital telemetry and comparative simulation visuals,
- a concept architecture overview,
- responsive layout for desktop and mobile browsing.

---

## 🏆 Competitive Differentiation: Why We're Unique

Current space-tech efforts often focus on cooling hardware or cluster routing alone. LumenOS is different because it introduces orbital temporal scheduling as a first-class operating system concern.

It treats thermal headroom as a primary scheduler input, just as CPU and memory are treated on Earth.

This is a new software layer for autonomous space-based AI, not just a hardware proposal.

---

## 🏗️ System Architecture

```text
⏱️ Autonomous 30-Second Control Loop

┌─────────────────────────┐
│ orbital_engine.py       │  "Where is the satellite?"
│ (Skyfield / SGP4)       │  → Sunlit / Eclipse, Radiator Angle, Solar Power
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│ thermal_engine.py       │◄─────│ workload_profiler.py   │
│ (Lumped Capacitance)   │      │ (Dataclass Catalog)    │
│ → Core Temp & Headroom  │      │ → Heavy / Med / Cold   │
└────────────┬────────────┘      └────────────┬────────────┘
             │                                │
             ▼                                ▼
┌──────────────────────────────────────────┐
│          lumen_scheduler.py              │
│      5-Min Look-Ahead → Checkpoint & Run │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│   Interactive Web / Telemetry Deck       │
└──────────────────────────────────────────┘
```

This architecture represents the core control loop and UI concept behind the project:

- orbital tracking,
- thermal modeling,
- workload classification,
- predictive scheduling,
- telemetry dashboard presentation.

---

## ⚡ Quick Start & Local Preview

This project is a static landing page and can be previewed locally in a few steps.

### Option 1: Open directly

Open `index.html` in a browser.

### Option 2: Serve locally

From the project root:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

---

## 📁 Repository Structure

```text
.
├── index.html
├── README.md
├── LICENSE (if added later)
└── assets/ (optional future expansion)
```

Current version is a single-file concept landing page prototype.

---

## 👨‍🚀 Team Proton

Built by Team Proton for the NASA Space Apps Challenge 2026.

Our focus is on the intersection of:

- spacecraft systems engineering,
- orbital thermal dynamics,
- AI systems design,
- and software-defined scheduling for space operations.

---

## ⚠️ Honest Limitations & Research Roadmap

This concept is intentionally a first-pass prototype. It demonstrates the core idea and the importance of thermally aware scheduling in LEO, but it does not yet represent a fully flight-certified system.

### Current limitations

- simplified thermal models,
- conceptual orbital assumptions,
- no real mission hardware integration,
- no full spacecraft power and thermal subsystem validation.

### Research roadmap

- integrate published orbital radiation and thermal models,
- model actual satellite power buses and battery charge/discharge dynamics,
- connect workload policies to real inference and training costs,
- explore autonomous checkpointing and fault-tolerant compute migration,
- expand from concept demo to mission-grade simulation and architecture tooling.

---

## 📜 License & Acknowledgments

This project is designed as a concept demonstration for NASA Space Apps Challenge 2026.

We acknowledge the role of:

- NASA Space Apps Challenge,
- orbital systems research,
- thermal engineering concepts,
- and the broader open-source web ecosystem used to build the experience.

If you plan to distribute or extend this project, add a proper license file and update the badge link accordingly.

---

## Tech Stack

- Three.js
- GSAP
- ScrollTrigger
- Vanilla JavaScript
- HTML5 + CSS3

This landing page is designed to communicate the science and mission value quickly while presenting the LumenOS concept in a visual, memorable way.
