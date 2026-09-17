# Adaptive ML Model Selection

## Overview

**Adaptive ML Model Selection** is a self-adaptive software system that dynamically selects and switches between multiple machine learning models based on changing runtime conditions.

The system uses **SVM, Random Forest, and XGBoost** models with different performance characteristics. Runtime metrics such as **accuracy, latency, CPU utilization, memory utilization, and workload** are continuously monitored.

A **MAPE-K-inspired adaptation engine** evaluates these conditions against predefined SLA requirements and decides whether the current model should be retained or another model should be selected.

The system also uses **persistence detection, cooldown, and hysteresis** to prevent unnecessary or repeated model switching.

---

## Problem Statement

Traditional ML applications generally evaluate multiple models offline and deploy one selected model.

```text
Dataset
   ↓
Train Multiple Models
   ↓
Evaluate Models
   ↓
Select Best Model
   ↓
Deploy ONE Model
   ↓
Always Use Same Model
```

However, runtime conditions can change after deployment. A model that performs well during normal workload may produce higher latency or consume more resources during heavy workload.

Frequent model switching can also introduce additional overhead and instability.

Therefore, an adaptive system is required to dynamically select a suitable model while maintaining **SLA requirements and system stability**.

---

## Proposed Solution

The proposed system maintains multiple ML models:

```text
SVM
Random Forest
XGBoost
```

Each model is evaluated based on its measured performance.

The runtime workflow is:

```text
Dataset
   ↓
Preprocessing
   ↓
Model Training
   ↓
Offline Evaluation
   ↓
Multiple Models
   ↓
Runtime Monitoring
   ↓
SLA Evaluation
   ↓
MAPE-K Adaptation
   ↓
Adaptive Model Selection
   ↓
Stable Model Switching
   ↓
Continue Monitoring
```

The system does not switch models for every small fluctuation. A model switch is considered when an SLA violation or performance degradation **persists for a defined period**.

---

## Objectives

* Train and maintain multiple ML models with different performance characteristics.
* Monitor CPU, memory, latency, workload, and model performance at runtime.
* Define and continuously evaluate SLA requirements.
* Dynamically select a suitable model according to current conditions.
* Use MAPE-K for continuous runtime adaptation.
* Detect persistent SLA violations before switching models.
* Prevent unnecessary switching using cooldown and hysteresis.
* Compare adaptive model selection with fixed-model deployment.
* Provide explainable adaptation decisions and performance metrics.

---

## Existing System

The conventional approach selects one model after offline evaluation.

```text
SVM
Random Forest
XGBoost
     ↓
Offline Evaluation
     ↓
Select One Model
     ↓
Deploy
     ↓
Continue Using Same Model
```

### Limitations

* Uses a fixed or pre-selected model.
* Selection mainly depends on offline performance.
* Runtime workload and resource conditions are not continuously considered.
* A high-performing model may cause increased latency or resource consumption.
* Changing runtime conditions can lead to SLA violations.
* No stability-aware mechanism for runtime model adaptation.

---

## Proposed System

The proposed system dynamically evaluates the available models according to runtime conditions.

### Runtime Parameters

* Prediction accuracy
* Response latency
* CPU utilization
* Memory utilization
* Current workload
* SLA requirements

### Example SLA

```text
Accuracy ≥ 85%
Latency ≤ 100 ms
CPU ≤ 70%
```

The adaptive selector evaluates the available models and chooses a model that satisfies the required constraints.

```text
Runtime Conditions
       ↓
SLA Evaluation
       ↓
MAPE-K Analysis
       ↓
Evaluate Models
       ↓
Select Suitable Model
       ↓
Stability Check
       ↓
Switch if Required
```

---

## Machine Learning Models

### SVM

* Suitable for classification problems.
* Can provide good predictive performance.
* May require more computation as dataset size increases.

### Random Forest

* Ensemble-based classification model.
* Provides a balance between predictive performance and computational cost.
* Suitable as a middle-level option.

### XGBoost

* Gradient-boosting based model.
* Often provides strong predictive performance.
* Can require more computational resources depending on configuration.

> Actual accuracy, latency, and resource usage will be measured experimentally rather than assumed beforehand.

---

## SLA Management

The SLA Manager defines the conditions that the active model should satisfy.

Example:

| Metric   | Requirement |
| -------- | ----------- |
| Accuracy | ≥ 85%       |
| Latency  | ≤ 100 ms    |
| CPU      | ≤ 70%       |

The system continuously checks whether the active model satisfies these requirements.

If the requirements are satisfied:

```text
SLA Satisfied
     ↓
Keep Current Model
```

If a violation persists:

```text
SLA Violation
     ↓
Persistence Check
     ↓
MAPE-K Adaptation
     ↓
Evaluate Alternative Models
```

---

## MAPE-K Adaptation

The adaptation engine follows the MAPE-K cycle.

### Monitor

Collect runtime metrics such as CPU, memory, latency, workload, and model performance.

### Analyze

Check SLA compliance and determine whether the current model is still suitable.

### Plan

Select an appropriate alternative model if adaptation is required.

### Execute

Activate the selected model and continue prediction.

### Knowledge

Store runtime metrics, previous decisions, model history, and adaptation results.

```text
Monitor
   ↓
Analyze
   ↓
Plan
   ↓
Execute
   ↓
Knowledge
   ↺
```

---

## Stability-Aware Model Switching

A major feature of the system is preventing unnecessary model switching.

A temporary change should not immediately trigger adaptation.

```text
Latency = 105 ms
      ↓
Temporary violation
      ↓
Continue monitoring
```

If the violation continues:

```text
Latency > SLA
      ↓
Persistent violation
      ↓
MAPE-K Analysis
      ↓
Evaluate Models
      ↓
Switch if required
```

The system can use:

* **Persistence detection** – confirms that a violation is sustained.
* **Cooldown** – prevents immediate switching again.
* **Hysteresis** – requires a meaningful improvement before switching back.

This helps avoid:

```text
SVM → Random Forest → SVM → Random Forest
```

caused by small runtime fluctuations.

---

## System Architecture

```text
                    React Dashboard
                           ↓
                     FastAPI Backend
                           ↓
                    Prediction Layer
                           ↓
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
         SVM          Random Forest       XGBoost
          └────────────────┼────────────────┘
                           ↓
                  Adaptive Model Selector
                           ↓
                       MAPE-K Engine
                           ↓
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
        Prometheus     SLA Manager   Stability Manager
        Monitoring
             └─────────────┼─────────────┘
                           ↓
                     Model Adaptation
                           ↓
                    Continue Monitoring
```

---

## Modules

### 1. Data Management

* Dataset loading and cleaning.
* Data preprocessing and feature preparation.
* Train/test data preparation.

### 2. Model Training

* Train SVM, Random Forest, and XGBoost.
* Save trained models for runtime selection.

### 3. Offline Evaluation

* Compare model accuracy.
* Measure latency and resource usage.
* Generate baseline performance results.

### 4. Prediction Service

* Provides REST APIs using FastAPI.
* Executes predictions using the active model.
* Provides model status information.

### 5. Runtime Monitoring

* Monitors CPU, memory, latency, workload, and model performance.
* Uses Prometheus for metric collection.

### 6. SLA Manager

* Defines SLA constraints.
* Continuously checks the active model against those constraints.

### 7. MAPE-K Adaptation Engine

* Performs Monitor, Analyze, Plan, and Execute operations.
* Uses stored knowledge to support adaptation.

### 8. Adaptive Model Selector

* Evaluates available models.
* Selects a suitable model based on SLA and runtime conditions.

### 9. Stability Manager

* Detects persistent violations.
* Applies cooldown and hysteresis.
* Prevents rapid model oscillation.

---

## Technology Stack

### Machine Learning

* Python 3.12+
* Scikit-learn
* XGBoost
* NumPy
* Pandas

### Backend

* FastAPI
* Uvicorn

### Frontend

* React
* Vite

### Database

* PostgreSQL

### Monitoring

* Prometheus

### Deployment

* Docker
* Docker Compose

### Development & DevOps

* Git
* GitHub
* GitHub Actions
* Trivy

---

## Workflow

```text
              Dataset
                 ↓
          Data Preprocessing
                 ↓
           Model Training
                 ↓
       ┌─────────┼─────────┐
       ↓         ↓         ↓
      SVM    Random Forest XGBoost
       └─────────┼─────────┘
                 ↓
          Offline Evaluation
                 ↓
          Prediction Service
                 ↓
         Runtime Monitoring
                 ↓
           SLA Evaluation
                 ↓
            MAPE-K Engine
                 ↓
       Persistent Violation?
             /       \
           No         Yes
           ↓           ↓
     Keep Current   Evaluate
        Model       Alternatives
                       ↓
                 Select Model
                       ↓
                Stability Check
                       ↓
                  Switch Model
                       ↓
              Continue Monitoring
                       ↺
```

---

## Implementation Status

### Completed / Implemented

* Project architecture and adaptive model-selection concept.
* Multiple-model approach.
* SVM, Random Forest, and XGBoost model integration.
* Model training and offline evaluation framework.
* FastAPI prediction service.
* SLA-based evaluation design.
* MAPE-K adaptation workflow.
* Persistent-condition detection logic.
* Stability mechanisms for model switching.
* Prometheus monitoring integration.
* Docker-based deployment structure.

### Planned / In Progress

* Complete runtime adaptation experiments.
* Fixed-model vs adaptive-model comparison.
* React monitoring dashboard.
* Detailed adaptation-performance analysis.
* GitHub Actions CI/CD pipeline.
* Trivy security scanning.
* Final experimental evaluation and documentation.

---

## Testing and Evaluation

The system will be tested under different runtime conditions.

### Normal Workload

```text
Stable CPU
Stable Latency
      ↓
SLA Satisfied
      ↓
Keep Current Model
```

### Heavy Workload

```text
CPU ↑
Latency ↑
      ↓
Check Persistence
      ↓
SLA Violation Persists
      ↓
Trigger Adaptation
      ↓
Select Suitable Model
```

### Recovery

```text
Workload ↓
     ↓
Runtime Conditions Improve
     ↓
Re-evaluate Current Model
     ↓
Adapt Only When Required
```

### Evaluation Metrics

* Model accuracy
* Response latency
* CPU utilization
* Memory utilization
* Number of model switches
* SLA violation duration
* Adaptation overhead
* System stability

---

## Docker Deployment

The application can be containerized using Docker Compose.

```text
┌──────────────────┐
│ React Frontend   │
└────────┬─────────┘
         ↓
┌──────────────────┐
│ FastAPI Backend  │
└────────┬─────────┘
         ↓
┌──────────────────┐
│   ML Models      │
│ SVM | RF | XGB   │
└────────┬─────────┘
         ↓
┌──────────────────┐
│   PostgreSQL     │
└──────────────────┘

Prometheus → Runtime Metrics
```

Run the application with:

```bash
docker compose config
docker compose build
docker compose up -d
```

Stop the application with:

```bash
docker compose down
```

---

## Project Structure

```text
adaptive-ml-model-selection/
│
├── src/
│   ├── backend/
│   │   ├── models/
│   │   ├── adaptation/
│   │   ├── monitoring/
│   │   ├── prediction/
│   │   └── main.py
│   │
│   └── frontend/
│
├── data/
├── tests/
├── docs/
├── prometheus/
├── results/
├── reports/
│
├── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## Development Roadmap

1. **Project Foundation**

   * Repository structure and architecture.

2. **ML Model Development**

   * Dataset preparation.
   * SVM, Random Forest, and XGBoost training.
   * Offline evaluation.

3. **Prediction Service**

   * FastAPI implementation.
   * Prediction and model-management APIs.

4. **Runtime Monitoring**

   * Prometheus integration.
   * Runtime metric collection.

5. **Adaptive Engine**

   * SLA Manager.
   * MAPE-K implementation.
   * Adaptive model selection.

6. **Stable Adaptation**

   * Persistence detection.
   * Cooldown mechanism.
   * Hysteresis.
   * Controlled model switching.

7. **Deployment & Evaluation**

   * Docker Compose.
   * CI/CD.
   * Security scanning.
   * Fixed vs adaptive experiments.

---

## Limitations

* The initial prototype uses a limited number of ML models.
* SLA thresholds are initially configuration-based.
* Model switching introduces some adaptation overhead.
* Real-time accuracy measurement may depend on the availability of ground-truth data.
* Runtime behavior depends on the selected dataset, hardware, and model configuration.
* The initial system focuses primarily on accuracy, latency, CPU, memory, and workload.

---

## Future Scope

* Use reinforcement learning or other ML techniques for intelligent model selection.
* Predict workload changes before SLA violations occur.
* Include GPU, network, and energy consumption.
* Support a larger pool of ML models.
* Extend adaptation to distributed and cloud environments.
* Use historical knowledge to improve future model-selection decisions.
* Develop advanced monitoring and analytics dashboards.

---

## Expected Outcome

The system aims to demonstrate that ML applications can adapt to changing runtime conditions instead of relying on a single fixed model.

```text
Monitor Runtime Conditions
          ↓
Check SLA
          ↓
Detect Persistent Violation
          ↓
Analyze Available Models
          ↓
Select Suitable Model
          ↓
Switch Safely
          ↓
Maintain SLA & Stability
          ↺
```

The final goal is to build a **resource-aware, SLA-aware, and stability-aware adaptive ML system** capable of dynamically selecting between SVM, Random Forest, and XGBoost according to runtime requirements.

---

## License

This project is developed as an academic prototype. An appropriate open-source license can be added when the project is finalized.
