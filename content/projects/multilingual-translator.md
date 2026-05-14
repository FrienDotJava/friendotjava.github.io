---
title: "Multilingual Translator"
date: 2026-05-14
draft: false
tags: ["HuggingFace", "PyTorch", "Streamlit", "mT5", "ByT5", "NLLB"]
description: "Fine-tuning multilingual Transformer models to translate Javanese, Sundanese, Balinese, Minangkabau, and Dayak Maanyan to Bahasa Indonesia."
image: /images/projects/translator.png
---

## Overview
This project is my undergraduate thesis. I fine-tuned three multilingual Transformer models (mT5, ByT5, and NLLB) to translate five local Indonesian languages to Bahasa Indonesia, and compared their performance against bilingual baselines. The goal was to show that a multilingual approach can outperform training one model per language pair, even when working with limited data.

{{< youtube 1faZ8aFsjxw >}}
---

## The Problem
Indonesia has over 700 local languages, but UNESCO reports that 139 of them are at risk of extinction. One contributing factor is the lack of digital support for these languages. When a language disappears from the digital space, it loses visibility and becomes harder to preserve.

Machine translation is one way to keep local languages alive in the digital world. However, most existing research only covers a single language pair at a time, and the translation quality is still low. Average BLEU scores for local Indonesian language pairs sit around 10 to 30, far below what is common for high-resource languages. The situation is even worse for very low-resource languages like Dayak Maanyan, which has almost no prior NMT research at all.

This project addresses that gap by training multilingual models that cover five local languages at once, and by running a structured comparison of three different Transformer architectures under the same conditions.

---

## Goals & Scope
- Develop and fine-tune multilingual NMT models based on Transformer architecture to translate five local Indonesian languages to Bahasa Indonesia under low-resource conditions
- Compare the performance of multilingual models against bilingual (one language pair per model) baselines
- Determine which Transformer model (mT5, ByT5, or NLLB) performs best for low-resource local language translation
- Reach a minimum average BLEU score of 40 across all translation scenarios

---

## System Design & Architecture
<img src="/images/projects/translator.png" alt="Architecture Diagram" style="width: 100%; max-width: 700px; border-radius: 8px; display: block; margin: 1rem auto;">

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
      <td>Data Preprocessing</td>
      <td>Pandas</td>
      <td>Flexible data manipulation for merging and cleaning multi-sheet Excel files</td>
    </tr>
    <tr>
      <td>Models</td>
      <td>mT5, ByT5, NLLB (via HuggingFace)</td>
      <td>Three architecturally distinct multilingual Transformers for a fair comparative study</td>
    </tr>
    <tr>
      <td>Training Framework</td>
      <td>HuggingFace Transformers + PyTorch</td>
      <td>Industry-standard tools for fine-tuning and managing pretrained language models</td>
    </tr>
    <tr>
      <td>Evaluation</td>
      <td>SacreBLEU + Evaluate</td>
      <td>Standard metrics (BLEU and Chrf++) for measuring translation quality</td>
    </tr>
    <tr>
      <td>Compute</td>
      <td>Google Colaboratory (NVIDIA Tesla H100)</td>
      <td>Cloud GPU access for running large model training without local hardware</td>
    </tr>
    <tr>
      <td>Inference App</td>
      <td>Streamlit</td>
      <td>Fast to build, integrates directly with HuggingFace and PyTorch for live demo</td>
    </tr>
    <tr>
      <td>Model Hosting</td>
      <td>HuggingFace Hub</td>
      <td>Easy public deployment of the fine-tuned model for online access</td>
    </tr>
  </tbody>
</table>

---

## Development Journey

### Phase 1: Research & Prototyping
The project started with a literature review to understand the current state of NMT for local Indonesian languages. Most prior work trained a separate model for each language pair and reported low BLEU scores, typically between 10 and 30. I identified three research gaps: the dominance of bilingual approaches, the near-total absence of NMT research on Dayak Maanyan, and the lack of any structured comparison between mT5, ByT5, and NLLB on the same dataset and conditions.

I also studied the architectural differences between the three models. mT5 is a general-purpose multilingual text-to-text model pretrained on 101 languages. ByT5 skips subword tokenization entirely and operates on raw UTF-8 bytes, making it potentially stronger for morphologically rich or non-standard text. NLLB (No Language Left Behind), developed by Meta AI, is a translation-dedicated model built specifically for low-resource languages and supports over 200 languages directly without using a pivot language like English.

### Phase 2: Core Implementation
The dataset was provided by a research team funded by the Ministry of Culture. It contains approximately 4,000 parallel sentence pairs per language, covering Javanese, Sundanese, Balinese, Minangkabau, and Dayak Maanyan paired with Bahasa Indonesia. The raw data was stored in Excel files with one sheet per language.

Data preprocessing involved merging all sheets into a single CSV, removing empty rows, normalizing Unicode (NFD normalization to strip diacritics), standardizing punctuation, and removing extra whitespace. Language tags were then added to each sentence so the model could identify the source and target language during training.

Feature engineering focused on two things: tokenization strategy and language tag insertion. Each model uses a different tokenizer. mT5 and NLLB use SentencePiece subword tokenization with large vocabulary sizes (around 250,000 tokens). ByT5 tokenizes at the byte level, meaning every character is broken into its UTF-8 bytes. This makes ByT5 immune to out-of-vocabulary problems, since it never encounters a truly unknown token.

All three models were fine-tuned using supervised learning with cross-entropy loss, training for up to 20 epochs with early stopping based on Chrf++ validation score. I ran two scenarios: one-directional (local language to Bahasa Indonesia only) and bidirectional (both directions). For the baseline comparison, each model was also trained separately on individual language pairs.

### Phase 3: Testing & Iteration
Model evaluation used BLEU and Chrf++ as the primary metrics. BLEU measures word-level n-gram overlap between the model output and the reference translation. Chrf++ adds character-level n-gram scoring and is more sensitive to morphological variation, which matters a lot for languages like Javanese and Balinese.

Training curves were analyzed for each model. mT5 showed the most stable convergence, with training and validation loss moving closely together. ByT5 triggered early stopping at epoch 12 in both scenarios due to mild overfitting. NLLB ran all 20 epochs without early stopping, as its validation loss continued to decrease gradually throughout, though a widening gap between training and validation loss indicated some degree of overfitting, especially in the bidirectional scenario.

Statistical significance tests were conducted using SacreBLEU to confirm that performance differences between models were not due to random variation.

---

## Challenges & How I Solved Them

**Challenge 1: Very limited training data per language**  
With only around 4,000 sentence pairs per language, models with large parameter counts like NLLB (600 million parameters) and ByT5 are prone to memorizing training data rather than learning general patterns. I addressed this by applying early stopping based on the best validation Chrf++ score, so the saved model weights always correspond to the best generalization point rather than the final epoch. Training all five languages together in the multilingual setup also acted as implicit regularization, since the model was exposed to a much larger and more varied dataset overall.

**Challenge 2: Tokenization mismatch for low-resource languages**  
Subword tokenizers like SentencePiece are trained on large multilingual corpora, so rare languages like Dayak Maanyan may be underrepresented in the vocabulary. This leads to excessive fragmentation of words into many small subword pieces, which hurts the quality of semantic representation. ByT5 sidesteps this entirely by processing raw bytes, so it never fragments a word in a way that loses meaning. For mT5 and NLLB, I monitored vocabulary coverage during data exploration and used the SentencePiece tokenizers with truncation set to the optimal max_length derived from sentence length distribution analysis.

**Challenge 3: Language interference in multilingual training**  
When training a single model on five languages simultaneously, there is a risk that higher-resource languages dominate and the model produces outputs that mix languages. To prevent this, I added explicit language identification tokens to each input sentence. For NLLB, this was handled through its built-in Language ID token system (for example, `jav_Latn` for Javanese and `sun_Latn` for Sundanese). For mT5 and ByT5, custom language tags were prepended to each source sentence. This approach ensured the decoder could always generate text in the correct target language.

**Challenge 4: Comparing models with very different architectures fairly**  
mT5, ByT5, and NLLB have different parameter counts, tokenization strategies, and pretraining objectives. To make the comparison as fair as possible, I used the same dataset splits, the same training hyperparameters, the same evaluation metrics, and the same compute environment (Google Colab with H100 GPU) for all three models. Both multilingual and bilingual (baseline) scenarios were run for each model so that the benefit of the multilingual approach could be measured independently of architecture differences.

---

## Results & Impact

**Multilingual vs. Baseline (Bilingual)**  
The multilingual approach outperformed the bilingual baseline across all three models and both evaluation metrics. The largest BLEU gain was seen in mT5, which improved by 5.01 points (from 56.99 to 62.01). ByT5 gained 4.70 points and NLLB gained 4.64 points. This confirms that cross-lingual transfer, where the model borrows knowledge from linguistically similar languages, helps even when the languages are distinct.

**Overall Best Model: NLLB**  
NLLB achieved the highest overall scores in the one-directional scenario with a BLEU of 75.18 and Chrf++ of 86.23. It was the most robust model across the majority of language pairs, particularly for Javanese, Sundanese, and Balinese, which have more prior multilingual representation in NLLB's pretraining data.

**ByT5 Shines on Low-Resource and Morphologically Complex Languages**  
ByT5 outperformed NLLB on Minangkabau and Dayak Maanyan. For Minangkabau to Bahasa Indonesia, ByT5 reached a BLEU of 79.51. For Dayak Maanyan to Bahasa Indonesia, ByT5 reached a BLEU of 72.91, well above what was expected for a language with almost no prior NMT research. The byte-level approach made ByT5 more adaptive to languages with non-standard morphology and limited training data.

**mT5 as a Competitive General-Purpose Option**  
mT5 had the lowest scores overall but showed the most stable training curves with minimal overfitting. As a general-purpose model not specialized for translation, it still delivered competitive results and showed the largest relative gain from the multilingual setup.

All three models exceeded the minimum BLEU target of 40, with average scores well above that threshold.

---

## What I Learned

This project was my first experience conducting research at a scale close to production NLP work. A few things stood out.

Data quality and preprocessing mattered more than I initially expected. Normalizing Unicode characters and adding language tags seemed like minor steps, but removing diacritics and standardizing punctuation had a measurable effect on tokenization quality.

The choice of evaluation metric changes the story. BLEU and Chrf++ sometimes disagreed on which model performed better for a given language pair. BLEU penalizes any word that does not exactly match the reference, while Chrf++ gives partial credit for character-level overlap. For morphologically rich languages, Chrf++ gave a more nuanced picture of translation quality.

Architecture matters as much as model size. NLLB has 600 million parameters, but ByT5 outperformed it on Dayak Maanyan despite having fewer parameters and no translation-specific pretraining. The byte-level approach was simply a better fit for the linguistic characteristics of that language.

Running bilingual baselines alongside the multilingual experiments was one of the most valuable parts of the project. Without that comparison, it would be impossible to isolate how much of the performance came from the multilingual training strategy versus the pretrained model weights.

---

## Future Work
- Expand the dataset size and include conversational text, not just news articles, to reduce domain bias
- Evaluate human judgment alongside automatic metrics (BLEU and Chrf++) for a more complete quality assessment
- Explore larger model variants and unsupervised pretraining specifically for Nusantara language families
- Extend coverage to other endangered local languages in Indonesia beyond the five studied here
- Evaluate fairness across dialect variation within each language

---

## Links
- (Private)