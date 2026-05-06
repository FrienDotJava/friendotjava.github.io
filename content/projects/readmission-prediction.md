---
title: "BPJS Hospital Readmission Prediction"
date: 2026-05-06
draft: false
tags: ["Machine Learning", "XGBoost", "FastAPI", "Streamlit", "Next.js"]
description: "A production-grade ML pipeline predicting 30-day hospital readmission risk for BPJS patients."
image: /images/projects/bpjs-readmission.png
---

## Overview
This project builds a machine learning system to predict 30-day hospital readmission risk for patients enrolled in BPJS, Indonesia's national health insurance program. The system includes a full ML pipeline from raw data to model serving, an interactive EDA dashboard, and a web application for real-time inference.

---

## The Problem
Hospital readmission within 30 days of discharge is a key indicator of healthcare quality and patient outcomes. For insurance providers like BPJS, unplanned readmissions represent a significant cost burden and often signal gaps in post-discharge care. Identifying high-risk patients before they are discharged allows healthcare providers to intervene early, coordinate follow-up care, and allocate resources more effectively. This project addresses that problem by developing a predictive model trained on historical BPJS patient data, covering demographics, primary care visits, and inpatient hospitalization records.

---

## Goals & Scope
- Build a predictive model for 30-day readmission risk using historical BPJS patient data
- Analyze patient demographics, visit patterns, and clinical factors that contribute to readmission
- Identify the key drivers of readmission to support clinical interpretation
- Provide accessible interfaces for model inference and data exploration

---

## System Design & Architecture
<img src="/images/projects/bpjs/arch.png" alt="Architecture Diagram" style="width: 100%; max-width: 700px; border-radius: 8px; display: block; margin: 1rem auto;">

---

## Tech Stack
<table class="table table-bordered mt-3 text-light">
  <thead>
    <tr>
      <th>Layer</th>
      <th>Technology</th>
      <th>Why I chose it</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ML Model</td>
      <td>XGBoost</td>
      <td>Strong performance on tabular healthcare data, built-in feature importance</td>
    </tr>
    <tr>
      <td>Pipeline Orchestration</td>
      <td>DVC</td>
      <td>Reproducible ML pipeline with dependency tracking</td>
    </tr>
    <tr>
      <td>Experiment Tracking</td>
      <td>MLflow + DagsHub</td>
      <td>Centralized logging of runs, metrics, and model versions</td>
    </tr>
    <tr>
      <td>Data Validation</td>
      <td>Great Expectations</td>
      <td>Automated data quality checks at each pipeline stage</td>
    </tr>
    <tr>
      <td>Feature Engineering</td>
      <td>Featuretools</td>
      <td>Automated multi-table feature creation across linked datasets</td>
    </tr>
    <tr>
      <td>Hyperparameter Tuning</td>
      <td>Optuna</td>
      <td>Fast and automated hyperparameter tuning</td>
    </tr>
    <tr>
      <td>Interpretability</td>
      <td>SHAP</td>
      <td>Per-prediction explanations for clinical trust</td>
    </tr>
    <tr>
      <td>Drift Monitoring</td>
      <td>Evidently AI</td>
      <td>Post deployment monitoring tool</td>
    </tr>
    <tr>
      <td>API Layer</td>
      <td>FastAPI</td>
      <td>Async, lightweight backend for model serving</td>
    </tr>
    <tr>
      <td>Dashboard</td>
      <td>Streamlit</td>
      <td>Rapid interactive EDA without frontend overhead</td>
    </tr>
    <tr>
      <td>Frontend</td>
      <td>Next.js (React + Tailwind)</td>
      <td>Modern, responsive web app for model inference</td>
    </tr>
    <tr>
      <td>Cloud Storage</td>
      <td>Google Cloud Storage</td>
      <td>Centralized data storage accessible across pipeline stages</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Research & Prototyping
The project began with understanding the BPJS dataset structure, which consists of three separate data sources: Peserta (patient demographics), FKTP (primary care records), and FKRTL (hospital inpatient records). Before writing any model code, I spent time understanding how these tables relate to each other through patient and visit identifiers, and how readmission events are defined. I also researched best practices for healthcare ML, particularly around class imbalance and the risk of temporal data leakage, since these are common pitfalls specific to clinical prediction problems.

### Phase 2: Core Implementation
The pipeline was built in stages using DVC (data is tracked using S3 DVC remote), so each step from data loading to model evaluation could be run independently and reproduced consistently. Raw data was stored in Google Cloud Storage and loaded into the pipeline at the first stage, mimicking how a production system would pull from a centralized data lake. DVC remote storage was configured on Amazon S3 (Dagshub), which meant the full pipeline state including data, intermediate artifacts, and model files could be versioned and restored consistently across environments. This setup was a deliberate choice to simulate the infrastructure patterns used in real production ML projects, even though the scale here is a research dataset. Data validation using Great Expectations was introduced early in the pipeline to catch schema issues and missing values before they could silently corrupt downstream features. Feature engineering drew from all three data sources, producing temporal features such as visit frequency and length of stay, as well as patient-level features covering demographics, insurance class, and geographic location. XGBoost was selected as the model due to its proven performance on tabular healthcare data and its built-in support for handling class imbalance through the `scale_pos_weight` parameter. Hyperparameter optimization was performed using Optuna with cross-validation, and all experiments were tracked in MLflow via DagsHub.

### Phase 3: Testing & Iteration
Model evaluation focused on recall as the primary metric, since missing a patient at risk of readmission carries a higher clinical cost than a false positive. Several techniques for handling class imbalance were tested, including SMOTE and undersampling, and compared against the baseline `scale_pos_weight` approach. The final model was evaluated on a held-out test set split by discharge date to prevent leakage of future information into training. SHAP values were computed to validate that the model's decisions were grounded in clinically meaningful features rather than spurious correlations.

---

## Challenges & How I Solved Them

**Challenge 1: Class imbalance**
Readmission events represent only about 10 to 15 percent of cases in the dataset, which caused early models to default to predicting no readmission almost universally. To address this, I applied `scale_pos_weight` in XGBoost to penalize false negatives, experimented with SMOTE for synthetic oversampling of the minority class, used AUC-PR instead of accuracy as the primary evaluation metric, and applied stratified cross-validation to maintain class distribution across folds.

**Challenge 2: Integrating three separate datasets**
The three data sources had inconsistent column naming and required careful linkage through patient and visit IDs. I implemented a multi-step cleaning process with standardized column names across all files, used temporal alignment to match records by visit date, and applied Featuretools for automated cross-table feature creation. Great Expectations validation rules were added at the joining stage to flag any records that failed to link correctly.

**Challenge 3: Preventing temporal data leakage**
Because this is a time-series prediction problem, using information from after a patient's discharge date in the training set would produce misleadingly high performance metrics. I enforced a time-based train/test split at the 80th percentile of discharge dates, and carefully reviewed all engineered features to ensure only historical information was used.

**Challenge 4: Making predictions explainable**
Healthcare models face a higher bar for adoption because clinicians need to understand and trust the predictions. I addressed this by computing SHAP values for individual predictions, exposing feature importance scores through the API, and including a confusion matrix visualization in the dashboard to help clinical users understand the model's error profile.

**Challenge 5: Monitoring model health after deployment**
A deployed model can silently degrade over time if the incoming data distribution shifts away from what it was trained on. To address this, I integrated Evidently AI to monitor data drift across the feature set. Evidently generates statistical reports comparing the training data distribution against incoming inference data, flagging features where drift is detected. This means the system can signal when a model retrain may be needed, rather than relying on manual inspection of prediction quality.

---

## Results & Impact
- ROC-AUC of 0.956 on the held-out test set
- Recall of 92.3%, identifying 923 out of every 1,000 actual readmission cases
- Only 54 missed readmission cases out of 703 in the test set
- Precision of 45.1%, an acceptable tradeoff for a high-sensitivity screening tool
- F1-Score of 0.606 on the positive class, with a Macro F1-Score of 0.756

---

## What I Learned
This project was my first experience building a machine learning pipeline at something close to a production scale. Working with three linked datasets taught me that data integration and validation are often harder than model training itself. I learned how easy it is to introduce data leakage in time-series problems, and how important it is to design the train/test split before touching any features. Using DVC for pipeline orchestration also changed how I think about ML projects: treating each stage as a reproducible unit made iteration much faster and debugging significantly easier. On the modeling side, I gained a deeper understanding of the precision-recall tradeoff and why accuracy is a poor metric for imbalanced clinical datasets. Setting up GCS for data storage and S3 as a DVC remote also gave me hands-on experience with cloud infrastructure that I had only read about before. Connecting these pieces together, versioned data on S3, experiment tracking on DagsHub, and drift monitoring via Evidently, made the project feel substantially closer to how ML systems are maintained in practice compared to a notebook-only workflow.

---

## Future Work
- Integrate real-time patient data ingestion so the model can score patients at the point of discharge rather than in batch
- Expand the feature set to include medication records and lab results, which are strong predictors of readmission in clinical literature
- Evaluate fairness metrics across demographic groups to ensure the model does not systematically under-serve any patient population

---

## Links
- [GitHub Repository](https://github.com/FrienDotJava/bpjs-hospital-readmission)
- [EDA Dashboard](https://bpjs-eda-report.streamlit.app/)
- [Model Inference App](https://bpjs-next-app.vercel.app/)
- [API (Deployed on Render)](https://bpjs-readmission-api.onrender.com/)
