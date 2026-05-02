---
title: "FlowGuard AI"
date: 2024-01-01
draft: false
tags: ["Node-RED", "Ollama", "FastAPI"]
description: "Behavioral Shadow Auditing & Fault Injection for Industrial Node-RED Flows."
image: /images/projects/flowguard.png
---

## Overview
A 2-3 sentence elevator pitch. What is it, who is it for, and what problem does it solve?

---

## The Problem
What pain point or gap motivated this project? 
Tell it like a story, what did you observe, read, or experience that made you think 
"this needs to exist"? This is where you hook the reader.

---

## Goals & Scope
What were you trying to achieve? Use a short list:
- Goal 1: e.g. detect anomalous flow behavior in real time
- Goal 2: e.g. simulate fault conditions without affecting production
- Goal 3: e.g. provide explainable audit logs via LLM

Also briefly mention what was intentionally out of scope.

---

## System Design & Architecture
Explain how the system is structured at a high level. 
Include an architecture diagram if you have one:

<img src="/images/projects/flowguard.png" alt="Architecture Diagram" style="width: 100%; max-width: 700px; border-radius: 8px; display: block; margin: 1rem auto;">

Walk through the major components and how they interact. 
This is the section engineers will read most carefully.

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
      <td><strong>Node-RED</strong></td>
      <td>Industry standard for IIoT flows</td>
    </tr>
    <tr>
      <td>LLM Backend</td>
      <td><strong>Ollama (Llama 3)</strong></td>
      <td>Local inference, no data leaves the network</td>
    </tr>
    <tr>
      <td>API Layer</td>
      <td><strong>FastAPI</strong></td>
      <td>Async, lightweight, easy to document</td>
    </tr>
    <tr>
      <td>Frontend</td>
      <td><strong>React + Tailwind</strong></td>
      <td>Component-based UI with utility styling</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Research & Prototyping
What did you explore first? What assumptions did you test?

### Phase 2: Core Implementation
What was the hardest part to build? What approach did you take?

### Phase 3: Testing & Iteration
How did you validate it works? What broke and how did you fix it?

---

## Challenges & How I Solved Them
This is the most valuable section for showing problem-solving ability.

**Challenge 1: [Name the problem]**
Describe what went wrong or was harder than expected, 
then explain your solution and the tradeoff you made.

**Challenge 2: [Name the problem]**
Same structure, problem, solution, tradeoff.

---

## Results & Impact
Be concrete. Numbers > adjectives.
- Reduced false positive rate by X%
- Processed X flows per second under load
- Used in X production environments
- Demo/research presented at [event]

---

## What I Learned
Honest reflection. What would you do differently? 
What skills or concepts did this project teach you?
This shows self-awareness and growth mindset.

---

## Future Work
What's next if you continue the project?
- Feature idea 1
- Scalability improvement
- Known limitation you'd address

---

## Links
- [GitHub Repository](https://github.com/...)
- [Live Demo](https://...)
- [Paper / Report / Presentation](#) *(if applicable)*