---
title: "FlowGuard AI"
date: 2024-01-01
draft: false
tags: ["Node-RED", "Ollama", "FastAPI"]
description: "Behavioral Shadow Auditing & Fault Injection for Industrial Node-RED Flows."
image: /images/projects/flowguard.png
---

## Overview
FlowGuard AI is a system that simulates a real-world manufacturing environment where an AI agent continuously monitors and audits industrial automation logic for security vulnerabilities and logic flaws.

{{< youtube SWnxta0uE7A >}}
---

## The Problem
During an internship at a manufacturing company, I observed that many business processes were still handled manually, which introduced the risk of human error. One area that stood out was in the Engineering department, where developers had to hunt for errors in automation systems by hand, a process that could take several hours with no guarantee of completeness. This experience motivated me to build an AI-powered system capable of analyzing industrial automation logic and identifying faults automatically.

---

## Goals & Scope
- Detect anomalous flow behavior in real time
- Simulate fault conditions without affecting production
- Generate explainable audit logs using an LLM
- Run entirely in a local environment so no data is sent to the cloud or external services.

---

## System Design & Architecture
<img src="/images/projects/flowguard/arch.png" alt="Architecture Diagram" style="width: 100%; max-width: 700px; border-radius: 8px; display: block; margin: 1rem auto;">

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
      <td>Flow Runtime</td>
      <td>Node-RED</td>
      <td>Industry standard for IIoT flows</td>
    </tr>
    <tr>
      <td>LLM Backend</td>
      <td>Ollama (Llama 3.2)</td>
      <td>Local inference, no data leaves the network</td>
    </tr>
    <tr>
      <td>API Layer</td>
      <td>FastAPI</td>
      <td>Async, lightweight, easy to document</td>
    </tr>
    <tr>
      <td>Frontend</td>
      <td>Next.js (React + Tailwind)</td>
      <td>Component-based UI with utility styling</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Research & Prototyping
Having no prior exposure to industrial technologies, I started with a research phase to build foundational understanding. I studied Node-RED and related industrial concepts including Modbus, PLCs, and IIoT. Node-RED stood out as a widely adopted standard in the industry, and I became interested in connecting it to an AI system using Python. During this phase I also discovered the pymodbus library, an open-source Python implementation of the Modbus protocol, which became a key part of the integration.

### Phase 2: Core Implementation
The original design relied on tool-calling to let the AI agent fetch and analyze active Node-RED flows. However, this approach proved unreliable with locally hosted models given my hardware constraints. I revised the architecture so that FastAPI fetches the active flow directly, returns the data as JSON, and passes it to the model for analysis. This made the pipeline deterministic and consistent.
To validate detection capability, I implemented a fault injection mechanism that deliberately introduces flaws into Node-RED flows. These faults ranged from missing connections between nodes, incorrect data type mappings, and misconfigured Modbus register addresses to logical errors such as inverted conditional branches and missing error-handling nodes. The injected flows were then passed through the analysis pipeline to test whether the model could correctly identify and describe each flaw. This approach allowed systematic validation without touching any production environment, keeping the testing process entirely self-contained.

### Phase 3: Testing & Iteration
Testing was conducted by compiling a list of known flaws and verifying whether the model correctly identified each one. This approach confirmed the system's detection capability, though automated evaluation has not yet been implemented. 

---

## Challenges & How I Solved Them

**Challenge 1: Inconsistent tool calling with local LLMs**
Getting a locally hosted LLM to call tools reliably was more difficult than anticipated. Hardware limitations, specifically available memory and RAM, made it hard to run a model large enough to handle tool use consistently. To resolve this, I moved tool invocation out of the model entirely: FastAPI now handles all API calls programmatically and supplies the resulting JSON directly to the model for analysis. This change improved reliability significantly and simplified the overall flow.

---

## Results & Impact
Be concrete. Numbers > adjectives.
- Reduced flow analysis time by 60%
- Detected 90% of introduced flaws across test cases

---

## What I Learned
This project highlighted how hardware constraints directly shape the design decisions of locally hosted AI systems. A model that performs well on a cloud server may behave inconsistently when run on a consumer machine with limited RAM, which pushed me to design around reliability rather than assuming the model would always behave as expected. I also gained practical experience in prompt engineering, learning how to structure inputs that produce consistent, useful outputs aligned with user needs. Beyond the technical side, this project taught me the value of pivoting early when an architectural decision proves fragile. Switching from tool-calling to a programmatic pipeline was a significant change, but making it sooner rather than later saved considerable debugging time and produced a more robust system overall. 

---

## Future Work
- Implement structured model evaluation with automated metrics
- Optimize inference speed for more responsive analysis
- Explore lightweight quantized models that can run reliably within tighter memory budgets
- Add a feedback loop so flagged faults can be reviewed and used to refine future prompts and detection logic

---

## Links
- [GitHub Repository](https://github.com/FrienDotJava/manufacture-security)