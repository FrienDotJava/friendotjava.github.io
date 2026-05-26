---
title: "Anime Recommendation App"
date: 2026-05-26
draft: false
tags: ["MLOps", "Collaborative Filtering", "FastAPI", "Next.js", "MLflow", "DVC"]
description: "An end-to-end MLOps implementation of a product recommender system using an anime dataset as a proxy use case, covering the full ML lifecycle from data versioning to live deployment."
image: /images/projects/anime-recommendation-app.png
---

## Overview
This project is an end-to-end MLOps implementation of a product recommender system, using an anime dataset as a proxy for real-world product recommendation scenarios. Users rate anime titles they have watched, and the system generates personalized recommendations based on those ratings. The project covers the full machine learning lifecycle, from raw data ingestion and experiment tracking to model deployment and a live user-facing frontend, demonstrating how a recommendation system is built, versioned, and shipped in a production-like environment.

---

## The Problem
Recommendation systems are one of the most common and impactful applications in industry, but most tutorials stop at model training. There is a large gap between a trained model in a notebook and a recommendation system that is actually deployed, monitored, and maintainable. At the same time, building on real proprietary product data is not always feasible for learning purposes. This project addresses both problems by using a publicly available anime rating dataset as a proxy and building the entire MLOps pipeline around it, from data versioning to continuous deployment, so the architecture can be adapted to any product domain.

---

## Goals & Scope
- Build a complete, end-to-end recommendation pipeline covering data, model, and serving layers
- Use MLflow for experiment tracking to ensure reproducibility and easy model comparison
- Version all data and model artifacts so every experiment can be reproduced from a known state
- Deploy the model as a REST API with a CI/CD pipeline that ships updates automatically
- Provide a simple frontend so users can interact with the system and see real recommendations

Out of scope: real-time model retraining, user authentication, and A/B testing infrastructure.

---

## System Design & Architecture

The system is divided into three decoupled layers that communicate through well-defined interfaces.

The **data and experimentation layer** handles everything before the model is deployed. Raw anime and rating data is versioned with DVC, which stores the actual files in Google Cloud Storage while keeping lightweight pointers in Git. Training runs are tracked with MLflow, deployed on DagsHub, which logs all hyperparameters, evaluation metrics, and saved model artifacts. This setup simulates a real-world team environment where multiple people can compare experiments in a shared interface.

The **backend layer** wraps the best trained model in a FastAPI server. It exposes a single `/recommend` endpoint that accepts a user's rated titles and returns a ranked list of recommendations. Every push to the main branch triggers a GitHub Actions workflow that runs checks and redeploys the API to Render automatically.

The **frontend layer** is a Next.js application deployed on Vercel. It gives users a form to rate anime they have seen, sends those ratings to the FastAPI backend, and displays the returned recommendations. Because the frontend and backend are fully decoupled, either layer can be updated independently without affecting the other.

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
      <td>Frontend</td>
      <td><strong>Next.js (TypeScript)</strong></td>
      <td>Clean, component-based UI that handles API calls and state management well</td>
    </tr>
    <tr>
      <td>API Layer</td>
      <td><strong>FastAPI</strong></td>
      <td>Lightweight async backend that is easy to wrap around a trained ML model</td>
    </tr>
    <tr>
      <td>Model</td>
      <td><strong>Collaborative Filtering (Matrix Factorization)</strong></td>
      <td>Well-suited for user-item rating data; generalizes to any product domain</td>
    </tr>
    <tr>
      <td>Experiment Tracking</td>
      <td><strong>MLflow on DagsHub</strong></td>
      <td>Logs parameters, metrics, and artifacts; DagsHub provides a shared remote UI</td>
    </tr>
    <tr>
      <td>Data Versioning</td>
      <td><strong>DVC + Google Cloud Storage</strong></td>
      <td>Tracks large data files outside Git with a lightweight pointer system</td>
    </tr>
    <tr>
      <td>CI/CD</td>
      <td><strong>GitHub Actions</strong></td>
      <td>Automated checks and deployment trigger on every push to main</td>
    </tr>
    <tr>
      <td>Deployment (Backend)</td>
      <td><strong>Render</strong></td>
      <td>Free-tier hosting for the FastAPI service with automatic redeploy support</td>
    </tr>
    <tr>
      <td>Deployment (Frontend)</td>
      <td><strong>Vercel</strong></td>
      <td>Zero-config deployment for Next.js with fast global CDN</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Data & Experimentation Setup
The first step was building a reproducible foundation. Raw anime metadata and user rating CSVs were registered with DVC and pushed to a Google Cloud Storage bucket as the remote. This meant the exact dataset used for any training run could always be retrieved from its DVC hash, even months later. MLflow was configured to log to a remote DagsHub server so experiment results would persist beyond the local machine and be accessible from any browser. I ran several training experiments, varying the number of latent factors and regularization strength, and compared them in the MLflow UI before selecting the best model.

### Phase 2: Model Serving and CI/CD Pipeline
Once the best model was identified, the next step was serving it. I exported the trained model artifact from MLflow and built a FastAPI server with a `/recommend` endpoint. The endpoint accepts a list of user ratings as input, runs them through the model, and returns ranked recommendations with predicted scores. The GitHub Actions workflow was configured to run on every push to main and trigger a fresh Render deployment when checks pass. This made the update cycle simple: improve the model, push to main, and the live API updates automatically.

### Phase 3: Frontend and End-to-End Testing
The Next.js frontend was built as a standalone application that only knows about the FastAPI endpoint URL. Users can search for anime titles, assign star ratings, and submit their list to get back personalized recommendations. I tested the full flow end-to-end, including edge cases like users with very few ratings or titles the model had not seen during training, and added graceful fallback messages for those cases.

---

## Challenges & How I Solved Them

**Challenge 1: Keeping data and code in sync**
Large data files cannot be stored in Git, but the dataset version used for a training run needs to be traceable. Using DVC solved this by storing a small `.dvc` pointer file in the repo that references the exact dataset version in Google Cloud Storage. Any past commit can be checked out and the matching dataset pulled with a single command, making experiments fully reproducible.

**Challenge 2: Cold start for new users**
Collaborative filtering depends on having enough rating history to make meaningful recommendations. New users with only one or two ratings get poor recommendations because the model has little signal to work with. I handled this by adding a minimum ratings threshold on the frontend and showing a message that encourages users to rate at least a few titles before submitting, which produces noticeably better results.

**Challenge 3: Model artifact portability**
The trained model needed to move cleanly from the MLflow artifact store into the FastAPI container at deployment time. I solved this by logging the model in a standardized format during training and writing a load function that pulls the artifact by run ID during the API startup sequence. This decoupled the serving code from any specific file path on the local machine.

**Challenge 4: Connecting two separately deployed services**
The Next.js frontend on Vercel and the FastAPI backend on Render are deployed independently, so CORS had to be configured explicitly on the backend to allow requests from the Vercel domain. I also used environment variables on both sides to store the endpoint URLs, so switching between local development and production required only a change to the `.env` file and not the source code.

---

## Results & Impact
- Full end-to-end pipeline from raw data to live user-facing application is working and deployed
- Every training run is tracked and reproducible through MLflow on DagsHub
- Dataset versions are pinned with DVC so any experiment can be reproduced from a known state
- CI/CD pipeline ships backend updates automatically on every push to main
- Public demo is live and accessible without any account or setup required

---

## What I Learned
The biggest lesson from this project is that the model is only a small part of a working recommendation system. Most of the engineering effort went into the surrounding infrastructure: versioning data, tracking experiments, building a clean API boundary, and wiring up the deployment pipeline. I also learned that MLflow and DVC solve different but complementary problems and are much more useful when used together than either one alone.

Building the CI/CD pipeline with GitHub Actions was a practical exercise in understanding what it actually means to "deploy a model." The definition is not just saving a model file; it is having a reproducible process that takes source code and produces a running service without manual steps. That shift in thinking is something I will carry into every future ML project.

---

## Future Work
- Add a scheduled retraining pipeline that ingests new rating data and redeploys the model automatically when performance improves
- Implement a proper cold start strategy using content-based filtering for users with few ratings
- Add experiment comparison tooling so multiple model versions can be evaluated side by side on the same test set before promotion
- Extend the frontend to support a watchlist feature so users can save recommendations for later

---

## Links
- [GitHub Repository](https://github.com/FrienDotJava/anime-recommendation-app)
- [Live Demo](https://anime-recommendation-next-app.vercel.app/)
- [API Docs](https://fastapi-example-265p.onrender.com/docs)