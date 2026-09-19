# UrbanFlow AI — Urban Traffic Flow & Incident Intelligence

> An AI-powered traffic decision-support and simulation platform for urban traffic analysis,
> congestion detection, incident intelligence, forecasting, and simulated intervention evaluation.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Objectives](#3-objectives)
4. [Key Features](#4-key-features)
5. [SUMO Simulation](#5-sumo-simulation)
6. [Simulation Scenarios](#6-simulation-scenarios)
7. [Traffic Data Collection](#7-traffic-data-collection)
8. [AI Traffic Analysis Pipeline](#8-ai-traffic-analysis-pipeline)
9. [Decision Engine](#9-decision-engine)
10. [Infrastructure Impact Simulation](#10-infrastructure-impact-simulation)
11. [Data Flow](#11-data-flow)
12. [Project Structure](#12-project-structure)
13. [Technology Stack](#13-technology-stack)
14. [Quick Start](#14-quick-start)
15. [Dashboard](#15-dashboard)
16. [Evaluation Metrics](#16-evaluation-metrics)
17. [Robustness and Explainability](#17-robustness-and-explainability)
18. [Hackathon Scope](#18-hackathon-scope)
19. [Demo Scenario](#19-demo-scenario)
20. [Future Scope](#20-future-scope)
21. [Conclusion](#21-conclusion)
22. [Project Status](#22-project-status)

---

## 1. Project Overview

**UrbanFlow AI** is an AI-powered traffic decision-support system built to understand changing urban
traffic conditions. It detects congestion and abnormal traffic behavior, forecasts traffic states
15–60 minutes ahead, recommends simulated traffic-management and diversion actions, and evaluates
longer-term infrastructure and network modifications through counterfactual simulation.

This is **not a navigation app or a generic chatbot**. It is a decision-support and simulation
platform designed to help analysts and planners understand what is happening on a road network,
why it is happening, what is likely to happen next, and what simulated actions could improve
conditions.

**SUMO (Simulation of Urban MObility)** is the traffic simulation backbone. Python and TraCI
control simulation execution, apply scenario changes, and collect road-level traffic measurements
at every stage of the pipeline.

### Core Workflow

```
Traffic / Road Network Data
          ↓
SUMO Network + Routes + Demand
          ↓
Simulation
          ↓
Traffic Measurements
          ↓
AI Traffic Analysis
          ↓
Congestion + Anomaly Detection
          ↓
15–60 Minute Forecasting
          ↓
Decision Engine
          ↓
Simulated Diversion / Infrastructure Scenario
          ↓
Baseline vs Counterfactual SUMO Simulation
          ↓
Impact Metrics
          ↓
Dashboard
```

---

## 2. Problem Statement

Urban traffic is a complex, continuously changing system. The challenge is not simply that roads
get congested — it is that congestion is driven by multiple interacting factors that shift faster
than manual operators can respond:

- Rapidly changing traffic demand throughout the day
- Peak-hour congestion on arterial corridors and flyovers
- Signalized junction delays and poorly timed signal phases
- Persistent bottlenecks that recur at the same locations daily
- Road capacity limitations that cannot be changed quickly
- Road works reducing lane availability
- Weather-related slowdowns affecting large parts of the network
- Event-driven traffic surges concentrated in short time windows
- Traffic incidents causing sudden, localized capacity loss
- Congestion spillback propagating queues to neighboring roads
- Network-wide side effects when diversions shift demand elsewhere

Simply visualizing congestion on a map is not enough. Operators need a system that can clearly
answer:

| Question | Required system capability |
|----------|---------------------------|
| What is happening? | Current congestion state, anomalies, incidents |
| Why is it happening? | Contributing traffic indicators and context |
| What is likely to happen next? | 15–60 minute traffic forecasts |
| What can be done? | Candidate simulated diversions or interventions |
| Why is that action recommended? | Evidence, reasoning, and confidence |
| What evidence supports it? | Traffic indicators, model outputs, scenario data |
| What is the simulated impact? | Before/after comparison from SUMO counterfactual runs |

UrbanFlow AI is designed to answer all of these questions within a single, unified simulation and
AI pipeline.

---

## 3. Objectives

1. Build continuously updated traffic and network conditions using datasets and SUMO simulation.
2. Detect and classify congestion across road segments.
3. Detect abnormal traffic behavior using machine learning.
4. Classify supported incident scenarios where available data supports classification.
5. Forecast traffic conditions at +15, +30, +45, and +60 minutes.
6. Generate evidence-based simulated diversion and traffic-management recommendations.
7. Identify recurring bottlenecks from repeated simulation runs.
8. Simulate hypothetical infrastructure and network modifications.
9. Compare baseline and counterfactual SUMO scenarios.
10. Display measurable impact, evidence, and confidence through an interactive dashboard.

---

## 4. Key Features

### 4.1 Congestion Detection

Congestion is detected using road-level traffic indicators collected from SUMO:

- Mean speed
- Free-flow speed
- Vehicle count and flow
- Occupancy
- Queue length
- Travel time
- Delay

The primary classification signal is the **speed ratio**:

```
speed_ratio = mean_speed / free_flow_speed
```

Traffic states are classified as:

| State | Speed Ratio | Description |
|-------|-------------|-------------|
| Free Flow | > 0.80 | Normal conditions |
| Moderate | 0.60 – 0.80 | Reduced speed, manageable |
| Heavy | 0.40 – 0.60 | Significant congestion |
| Severe | < 0.40 | Near-standstill conditions |

Thresholds are calibrated using the available dataset and simulation results rather than fixed
universal values.

---

### 4.2 Abnormal Traffic Detection

Abnormal traffic behavior is detected using machine-learning-based anomaly detection.
**Isolation Forest** is a suitable baseline for identifying road segments whose traffic
measurements deviate significantly from expected patterns.

Features used:

- Mean speed
- Vehicle count
- Occupancy
- Queue length
- Mean travel time
- Delay

**An anomaly flag does not automatically mean an accident.** It may indicate a demand surge,
signal failure, road works, or another disruption. Further classification determines the cause.

---

### 4.3 Incident Scenario Classification

Controlled SUMO scenarios are created to represent distinct traffic conditions:

| Scenario | Description |
|----------|-------------|
| Normal | Baseline traffic conditions |
| Demand Surge | Increased vehicle demand on a corridor |
| Signal Delay | Modified signal timing introducing delays |
| Capacity Reduction | Reduced lane availability or road capacity |
| Road Obstruction | Controlled blockage representing an abnormal condition |

When labeled scenario data is available, a supervised classifier such as **Random Forest** can
classify incoming traffic measurements into one of these scenario types.

> Train/test splitting must be performed at the simulation-run level to prevent data leakage
> between rows from the same simulation run.

---

### 4.4 Traffic Forecasting

The forecasting module predicts traffic conditions on key road segments at four horizons:

| Horizon | Use |
|---------|-----|
| +15 minutes | Short-term, high-confidence operational decisions |
| +30 minutes | Medium-term planning |
| +45 minutes | Medium-term planning |
| +60 minutes | Longer-term, wider uncertainty |

Prediction targets include mean speed, vehicle count, queue length, travel time, and congestion
state. Forecasts feed the decision engine so it can act before congestion peaks rather than after.

Evaluation metrics: **MAE**, **RMSE**, **MAPE** (where appropriate).

---

### 4.5 Congestion Spillback Early Warning

The system monitors whether congestion and queues on a road segment are likely to propagate to
neighboring roads or upstream junctions. Early warning of spillback allows the decision engine to
recommend preemptive action before congestion spreads network-wide.

---

### 4.6 Recurring Bottleneck Detection

By analyzing repeated simulation runs, the system identifies roads and junctions that consistently
experience poor traffic conditions:

- High congestion duration
- High queue length
- High delay
- Low mean speed
- High travel time

This separates **persistent network limitations** — which require infrastructure-level responses —
from **one-off incidents** that can be managed operationally.

---

### 4.7 AI Traffic Intervention Lab

The core innovation of UrbanFlow AI is the ability to not just detect and forecast traffic
problems, but to **simulate, compare, and explain candidate interventions**.

**Predict → Compare → Simulate → Explain**

The system generates multiple candidate interventions, simulates each in SUMO, compares their
network-wide effects, and presents:

| Output | Description |
|--------|-------------|
| Baseline performance | Traffic metrics before intervention |
| Intervention performance | Traffic metrics after simulated intervention |
| Travel-time change | Improvement or degradation |
| Queue change | Queue length before and after |
| Speed change | Mean speed before and after |
| Throughput change | Vehicles completing trips |
| Delay change | Extra travel time before and after |
| Side effects | Network-wide impact on other roads |
| Evidence | Traffic indicators supporting the recommendation |
| Confidence | Model confidence and data quality assessment |

No single intervention is presented as universally optimal. The dashboard presents the evidence
and lets the analyst make the final decision.

---

### 4.8 Network-Wide Side-Effect Checker

A diversion that reduces congestion on one road may increase congestion elsewhere by shifting
demand onto roads that cannot absorb it. Every candidate diversion is therefore evaluated using
**network-wide metrics**, not just the metrics of the directly affected segment.

---

### 4.9 Confidence-Aware Recommendations

Every recommendation includes:

- Supporting traffic indicators and evidence
- Model confidence score
- Expected impact range
- Data quality warnings where applicable

If the available evidence is insufficient to support a recommendation with reasonable confidence,
the dashboard reduces the confidence score or displays a warning rather than presenting an
unsupported conclusion.

---

## 5. SUMO Simulation

SUMO is the simulation backbone of UrbanFlow AI. It provides the controlled traffic environment
from which all measurements, scenarios, and counterfactual comparisons are derived.

### SUMO File Structure

```
sumo/
├── network.net.xml       # Road network topology
├── routes.rou.xml        # Vehicle routes
├── demand.rou.xml        # Traffic demand definition
├── simulation.sumocfg    # Simulation configuration
└── scenarios/            # Controlled traffic scenario configs
```

| File | Purpose |
|------|---------|
| `network.net.xml` | Roads, lanes, junctions, and signal plans |
| `routes.rou.xml` | Vehicle routes through the network |
| `demand.rou.xml` | Vehicle types, volumes, and timing |
| `simulation.sumocfg` | Master config linking network, routes, and outputs |
| `scenarios/` | Per-scenario configuration files |

**SUMO-GUI** is used for visual validation of the network and scenario behavior.
**NetEdit** is used for network editing and modification.

### TraCI Control

Python controls SUMO programmatically through **TraCI (Traffic Control Interface)**:

- Start and stop simulation runs
- Step through simulation time
- Read road-level traffic measurements at each step
- Apply scenario changes (capacity reduction, signal modification, route changes)
- Collect measurements for the AI pipeline
- Run baseline and counterfactual simulations under identical demand conditions

---

## 6. Simulation Scenarios

> All scenarios below are **simulation scenarios only**. They do not represent real-world
> interventions.

### Normal Scenario
Baseline traffic conditions representing typical off-peak or moderate demand. Used as the
reference point for all comparisons.

### Peak-Hour Scenario
Increased traffic demand concentrated in a defined time window, representing morning or evening
peak conditions on arterial corridors.

### Signal-Delay Scenario
Modified signal timing introducing additional delays at one or more junctions. Simulates the
effect of signal failure or a suboptimal signal plan.

### Capacity-Reduction Scenario
Reduced road capacity or lane availability on a selected segment, representing road works, a lane
closure, or a partial obstruction.

### Obstruction Scenario
A controlled road obstruction is introduced to represent an abnormal traffic condition such as a
stalled vehicle or incident. Used to generate labeled training data for incident classification.

### Diversion Scenario
A candidate route diversion is introduced into the simulation. Traffic is re-routed away from a
congested segment and the network-wide effect is measured against the baseline.

### Infrastructure Scenario
A hypothetical capacity increase, lane addition, or junction modification is applied to the SUMO
network. The modified network is simulated under the same demand as the baseline and results are
compared.

---

## 7. Traffic Data Collection

Traffic measurements are collected through TraCI at fixed intervals — for example, every
**60 simulated seconds**. Each record captures the state of a road segment at a point in time.

### Measurement Schema

```
timestamp          — Simulation time (seconds)
edge_id            — Road segment identifier
mean_speed         — Mean vehicle speed (m/s)
free_speed         — Free-flow speed for the segment (m/s)
vehicle_count      — Number of vehicles on the segment
flow               — Vehicles per hour passing a reference point
occupancy          — Fraction of time the segment is occupied (0–1)
queue_length       — Number of vehicles in queue
mean_travel_time   — Mean time to traverse the segment (seconds)
delay              — Extra travel time vs free-flow (seconds)
scenario           — Scenario label for the current run
incident_label     — Incident classification label (if applicable)
```

### Preprocessing and Feature Engineering

Raw measurements are cleaned to handle missing values and outliers. Derived features include:

- `speed_ratio = mean_speed / free_speed`
- Rolling averages over recent time windows
- Lag features for forecasting models
- Congestion state labels derived from speed ratio thresholds

---

## 8. AI Traffic Analysis Pipeline

```
Raw Traffic Data
      ↓
Data Cleaning
  (missing values, outliers, type validation)
      ↓
Feature Engineering
  (speed ratio, rolling averages, lag features, state labels)
      ↓
Congestion Detection
  (speed ratio thresholds → Free Flow / Moderate / Heavy / Severe)
      ↓
Anomaly Detection
  (Isolation Forest → anomaly flag per segment per timestep)
      ↓
Incident Classification
  (Random Forest on labeled scenario data → scenario type)
      ↓
Traffic Forecasting
  (regression / time-series model → +15/+30/+45/+60 min predictions)
      ↓
Decision Engine
  (congestion + forecast + anomaly → candidate interventions)
```

Each stage produces structured outputs that feed the next. The decision engine receives the
combined outputs of all upstream stages before generating any recommendation.

---

## 9. Decision Engine

The decision engine converts predictions and detected conditions into simulated, advisory actions.
It does not control any real-world infrastructure.

### Inputs

| Input | Source |
|-------|--------|
| Current congestion state | Congestion detection module |
| Forecasted congestion | Forecasting module |
| Anomaly / incident flags | Anomaly and incident modules |
| Road capacity | SUMO network data |
| Alternate routes | NetworkX graph analysis |
| Queue length | TraCI measurements |
| Travel time | TraCI measurements |
| Network topology | SUMO network graph |

### Outputs

| Output | Description |
|--------|-------------|
| Candidate diversion | Alternative route with expected impact |
| Traffic-management recommendation | Advisory action with supporting evidence |
| Expected impact | Predicted change in travel time, speed, queue, throughput |
| Infrastructure scenario | Hypothetical network modification for simulation |
| Evidence | Traffic indicators and model outputs |
| Confidence | Confidence score and data quality assessment |

> All outputs are **simulated and advisory**. No output triggers any real-world action.

---

## 10. Infrastructure Impact Simulation

Recurring bottlenecks can be analyzed by creating a modified copy of the baseline SUMO network
and simulating a hypothetical infrastructure change.

### Simulated Modifications

- Capacity increase on a bottleneck segment
- Lane configuration change
- Addition of a route connection
- Junction geometry or signal modification
- Network-level re-routing

### Comparison Metrics

The modified scenario runs under the same demand as the baseline:

| Metric | Baseline | Modified | Change |
|--------|----------|----------|--------|
| Average travel time | — | — | — |
| Mean speed | — | — | — |
| Queue length | — | — | — |
| Throughput | — | — | — |
| Delay | — | — | — |
| Congestion duration | — | — | — |

> **The result is a simulation, not a claim about real-world construction outcomes.**

---

## 11. Data Flow

```
Traffic / Road Network Inputs
        ↓
SUMO Network + Routes + Demand
        ↓
Simulation Runs
        ↓
Traffic Measurements
  (TraCI — every 60 simulated seconds)
        ↓
Preprocessing & Feature Engineering
        ↓
Congestion + Anomaly + Forecasting
        ↓
Recommendation
  (Diversion / Infrastructure Scenario)
        ↓
Baseline vs Counterfactual SUMO Runs
        ↓
Impact Metrics
        ↓
Dashboard
```

---

## 12. Project Structure

```
urbanflow-ai/
├── sumo/
│   ├── network.net.xml          # Road network topology
│   ├── routes.rou.xml           # Vehicle routes
│   ├── demand.rou.xml           # Traffic demand
│   ├── simulation.sumocfg       # SUMO configuration
│   └── scenarios/               # Controlled scenario configs
├── data/
│   ├── raw/                     # Raw TraCI output data
│   ├── processed/               # Cleaned and feature-engineered data
│   └── sample/                  # Sample data for development/testing
├── ai/
│   ├── congestion_model.py      # Congestion detection logic
│   ├── anomaly_model.py         # Isolation Forest anomaly detection
│   └── evaluation.py            # Classification evaluation metrics
├── forecasting/
│   ├── train.py                 # Forecasting model training
│   ├── predict.py               # +15/+30/+45/+60 min predictions
│   └── evaluation.py            # MAE / RMSE / MAPE evaluation
├── decision/
│   ├── routing.py               # NetworkX graph and route analysis
│   └── advisory.py              # Recommendation and confidence logic
├── simulation/
│   ├── traci_runner.py          # TraCI connection and data collection
│   ├── scenario_manager.py      # Scenario setup and execution
│   └── impact.py                # Baseline vs counterfactual comparison
├── backend/
│   └── main.py                  # FastAPI backend
├── frontend/
│   └── ...                      # React.js dashboard
├── notebooks/
│   └── analysis.ipynb           # Exploratory analysis and prototyping
├── README.md
└── requirements.txt
```

---

## 13. Technology Stack

| Area | Technology |
|------|------------|
| Traffic Simulation | SUMO + SUMO-GUI + NetEdit |
| Simulation Control | TraCI / Python |
| Data Processing | Python, Pandas, NumPy |
| Machine Learning | Scikit-learn; XGBoost if appropriate |
| Network Analysis | NetworkX |
| Backend | FastAPI |
| Frontend | React.js |
| Database | SQLite for prototype; PostgreSQL / PostGIS if required |
| Visualization | Interactive map + charts |

---

## 14. Quick Start

### Install SUMO

Download and install SUMO from the official site:
**https://sumo.dlr.de/docs/Installing/index.html**

Verify the installation:

```bash
sumo --version
```

Set the `SUMO_HOME` environment variable:

```bash
# Windows
set SUMO_HOME=C:\path\to\sumo

# macOS / Linux
export SUMO_HOME=/path/to/sumo
```

### Create Python Environment

```bash
python -m venv venv
```

Activate:

```bash
# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

Install dependencies:

```bash
pip install pandas numpy scikit-learn networkx fastapi uvicorn traci
```

Or from the requirements file:

```bash
pip install -r requirements.txt
```

### Validate the SUMO Network

Before running programmatic simulations, open the network in SUMO-GUI to validate the road
network, routes, and signal plans:

```bash
sumo-gui -c sumo/simulation.sumocfg
```

### Collect Traffic Data

Run the TraCI controller to execute the baseline simulation and collect traffic measurements at a
fixed interval (e.g., every 60 simulated seconds):

```bash
python simulation/traci_runner.py --scenario baseline --interval 60
```

Collected data is written to `data/raw/`.

### Run the Intelligence Pipeline

```bash
python backend/main.py
```

The pipeline executes in sequence:

```
Traffic Data
→ Preprocessing
→ Congestion Analysis
→ Anomaly Detection
→ Forecasting
→ Recommendation
→ SUMO Counterfactual Simulation
→ Impact Analysis
```

### Evaluate a Scenario

Run a counterfactual scenario and compare it against the baseline:

```bash
python simulation/impact.py --baseline baseline --scenario diversion_01
```

Metrics reported:

- Average travel time
- Mean speed
- Queue length
- Throughput
- Delay
- Congestion duration

---

## 15. Dashboard

The dashboard is the primary interface for analysts and decision-makers. It presents simulation
state, AI outputs, recommendations, and impact comparisons in a single view.

### Traffic Network Map

Displays the current simulated road network with congestion levels color-coded by traffic state
(Free Flow / Moderate / Heavy / Severe). Segment-level indicators are available on hover.

### Incident Alerts

Lists detected anomalies and classified incident scenarios with location (road segment ID),
classification, confidence score, supporting traffic indicators, and estimated impact on
surrounding roads.

### Forecast Panel

Displays +15, +30, +45, and +60 minute predictions for selected road segments, including
predicted congestion state and key traffic indicators.

### Recommendation Panel

Presents the decision engine output for a selected congested segment:

- Candidate simulated diversion route
- Expected change in travel time, queue, and speed
- Capacity check on the proposed alternate route
- Reasoning and supporting evidence
- Confidence score and data quality warnings

### Infrastructure Panel

Presents the recurring bottleneck analysis and infrastructure simulation results:

- Identified recurring bottleneck segment
- Selected hypothetical intervention
- Baseline performance metrics
- Simulated post-intervention performance metrics
- Estimated before/after impact

### Simulation Panel

Provides scenario controls for running and comparing SUMO simulations directly from the
dashboard — select and launch a scenario, view baseline vs counterfactual results side by side,
and export comparison metrics.

---

## 16. Evaluation Metrics

| Area | Metric | Purpose |
|------|--------|---------|
| Congestion | Accuracy / F1 | Measure traffic-state detection quality |
| Anomaly / Incident | Precision / Recall / F1 | Measure abnormal-condition detection |
| Forecasting | MAE / RMSE / MAPE | Measure 15–60 minute prediction accuracy |
| Routing | Travel time / Delay | Measure simulated diversion effect |
| Network Impact | Queue / Speed / Throughput / Delay | Compare baseline vs intervention |
| Robustness | Performance under changed demand, noisy data, missing values | Check generalization |

---

## 17. Robustness and Explainability

### Robustness Testing

The system is tested against conditions that deviate from the training distribution:

- Missing traffic measurements on one or more segments
- Noisy measurements (simulated by adding noise to SUMO outputs)
- Sudden unexpected traffic changes mid-simulation
- Increased demand beyond the training range
- Unexpected congestion patterns not seen during training

### Explainability

Every major recommendation must be able to answer:

```
What happened?
  → Congestion state, anomaly flag, incident classification

Why was it detected?
  → Traffic indicators: speed ratio, occupancy, queue length, delay

What is expected next?
  → +15/+30/+45/+60 minute forecasts for affected segments

What can be done?
  → Candidate simulated diversion or traffic-management action

Why is the action being recommended?
  → Network graph analysis, alternate route capacity, forecast improvement

What evidence supports it?
  → Traffic measurements, model outputs, SUMO counterfactual results
```

If the available evidence is insufficient, the system reduces the confidence score or displays an
explicit warning rather than presenting an unsupported conclusion.

---

## 18. Hackathon Scope

### Included

- SUMO-centered traffic simulation as the core environment
- Dataset and simulation-based traffic analysis
- Congestion detection and classification
- Abnormal traffic detection using machine learning
- 15–60 minute traffic forecasting
- Simulated route diversion with network-wide impact evaluation
- Recurring bottleneck identification from repeated simulation runs
- Before/after infrastructure simulation and comparison
- Explainable, evidence-based recommendations
- Confidence and data quality display
- Interactive decision-support dashboard

### Not Included

- Live traffic-signal control
- Real-world road construction or physical infrastructure changes
- Live camera or CCTV feed access
- GPS-device integration
- Roadside sensor integration
- Municipal infrastructure access
- Actual traffic intervention of any kind

**All actions remain simulated and advisory.**

---

## 19. Demo Scenario

The following sequence demonstrates the full UrbanFlow AI pipeline end to end:

1. Start a SUMO peak-hour scenario and display the simulated road network on the dashboard.
2. Identify a congested road segment using the congestion detection module.
3. Show the traffic indicators (speed ratio, occupancy, queue length, delay) that caused the
   congestion classification.
4. Introduce or detect a controlled abnormal scenario (e.g., capacity reduction or obstruction).
5. Display +15, +30, +45, and +60 minute traffic forecasts for the affected segment and its
   neighbors.
6. Generate a candidate diversion route using the decision engine.
7. Run the diversion scenario in SUMO under the same demand as the baseline.
8. Compare baseline and diversion results: travel time, queue length, throughput, delay, and
   side effects.
9. Identify a recurring bottleneck from aggregated simulation history.
10. Define a hypothetical infrastructure modification (e.g., capacity increase or lane addition).
11. Run the modified SUMO scenario under baseline demand.
12. Display before/after traffic impact on the dashboard.
13. Present the full reasoning chain: what happened, why, what is expected, what was recommended,
    what evidence supports it, and what the limitations are.

---

## 20. Future Scope

The following capabilities are outside the current software-only hackathon scope but represent
natural extensions of the platform:

- Integration with real-time traffic data feeds
- Weather API integration as a simulation input
- Public event information for demand forecasting
- Roadside sensor data ingestion
- Traffic camera analytics for incident detection
- Advanced spatiotemporal forecasting models (e.g., Graph Neural Networks, Transformers)
- Large-scale city-wide road network simulation
- Digital-twin traffic simulation with continuous real-world data synchronization

---

## 21. Conclusion

UrbanFlow AI provides a unified, SUMO-centered traffic decision-support workflow that goes beyond
visualization:

```
RAW TRAFFIC / ROAD DATA
        ↓
SUMO SIMULATION
  (Network + Demand + Scenarios)
        ↓
TRAFFIC UNDERSTANDING
  (Congestion Detection + Anomaly Detection)
        ↓
CONGESTION & INCIDENT INTELLIGENCE
  (Classification + Spillback Warning + Bottleneck Detection)
        ↓
15–60 MINUTE FORECAST
  (Speed + Queue + Travel Time + Congestion State)
        ↓
SIMULATED DIVERSION
  (Candidate Route + Network-Wide Side-Effect Check)
        ↓
RECURRING BOTTLENECK ANALYSIS
  (Persistent vs One-Off Congestion)
        ↓
INFRASTRUCTURE IMPACT SIMULATION
  (Baseline vs Counterfactual SUMO Run)
        ↓
DECISION DASHBOARD
  (Evidence + Confidence + Before/After Impact)
```

The system does not merely show that a road is congested. It uses simulation and AI to understand
the traffic state, forecast how it will evolve, test possible responses inside SUMO, quantify the
simulated network impact of each response, and present the evidence clearly so that analysts can
make informed decisions.

---

## 22. Project Status

```
Project        : UrbanFlow AI — Urban Traffic Flow & Incident Intelligence
Hackathon      : NEURAX HACKATHON 3.0
Type           : Software-only AI traffic decision-support system
Simulation     : SUMO (Simulation of Urban MObility)
Workflow       : SUMO-centered — planned and in development
Status         : In Development
```

---

*UrbanFlow AI — NEURAX HACKATHON 3.0*
