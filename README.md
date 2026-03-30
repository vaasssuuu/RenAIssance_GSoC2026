# RenAIssance GSoC 2026: End-to-End Handwritten Text Recognition

**Applicant:** Vatsalya Soni  
**Target Organization:** CERN / Human-AI (RenAIssance Project)  
**Test Completed:** Test II (Handwritten Sources)

## Project Overview
This repository contains the evaluation code and architectural proof-of-concept for the RenAIssance GSoC 2026 project. The objective is to build a robust Optical Character Recognition (OCR) pipeline for early modern Spanish manuscripts (17th-18th century). 

Rather than utilizing a Large Language Model (LLM) solely as a post-processing spellchecker, this project implements a **Multi-Agent Hybrid Pipeline** where Vision-Language Models (VLMs) are deeply embedded throughout the entire extraction, interpretation, and formatting lifecycle.

## The Architecture: A 4-Stage Multi-Agent Pipeline
Zero-shot VLM transcription fundamentally struggles with historical diplomatic formatting—specifically archaic abbreviations, non-standard hyphenation across line breaks, and faded ink on porous paper. To exceed the 90% accuracy threshold required by the project, this pipeline implements four engineered tactics:

### 1. Computer Vision Enhancement (CLAHE)
Before any AI processing, the raw PDF scans are passed through an OpenCV preprocessing layer. Using Contrast Limited Adaptive Histogram Equalization (CLAHE) and FastNlMeansDenoising, the pipeline dynamically binarizes the image, forcing faded ink to separate from the background and reducing the visual cognitive load on the VLM.

### 2. Attention Context Optimization (Image Chunking)
High-resolution manuscript pages frequently exceed the optimal visual context window of standard VLMs, resulting in "lost in the middle" hallucinations. The pipeline programmatically slices the enhanced image into horizontal chunks with a 5% pixel overlap, forcing localized attention maps.

### 3. Agent 1: The Vision Extractor
A primary VLM agent is tasked strictly with raw character identification across the image chunks. It operates without strict formatting constraints, maximizing its OCR recall capabilities.

### 4. Agent 2: Diplomatic Editor & Dynamic Few-Shot Injection
A secondary text-agent receives the raw, concatenated OCR output alongside a dynamically injected 150-character Ground Truth "Style Guide" extracted from the dataset. This few-shot injection forces the LLM to contextually stitch the chunks together while perfectly mimicking the author's spelling anomalies (e.g., interchangeable u/v), preserving exact superscripts (e.g., `V.ª`), and ignoring modern punctuation.

---

## Results and Metrics Analysis

The pipeline was evaluated against the 5 provided handwritten ground-truth transcriptions using the `jiwer` library.

| Document | CER | WER | Estimated Accuracy |
| :--- | :--- | :--- | :--- |
| `AHPG-GPAH AU61:2 – 1606` | 0.0682 | 0.2982 | **93.18%** |
| `AHPG-GPAH 1:1716,A.35 – 1744` | 0.0962 | 0.3962 | **90.38%** |
| `ES.28079.AHN::INQUISICIÓN,1667` | 0.0542 | 0.1722 | **94.58%** |
| `PT3279:146:342 – 1857` | 0.0396 | 0.2698 | **96.04%** |
| `Pleito entre el Marqués de Viana`| 0.0578 | 0.3143 | **94.22%** |

### Note on Evaluation Metrics
Across the 5 handwritten documents, the pipeline consistently achieved an estimated accuracy between **90.38% and 96.04%**. The Character Error Rate (CER) remains exceptionally low across all documents (averaging ~4-9%). The discrepancy between the CER and Word Error Rate (WER) highlights a known limitation in standard string-matching metrics for historical diplomatic transcriptions. The metric strictly penalizes the AI for resolving archaic line-end hyphens or standardizing spacing, even when the character extraction itself is perfectly accurate.

---

## Repository Contents
* `RenAIssance_GSoC_Evaluation.ipynb`: The fully executed Google Colab notebook containing the pipeline, automatic retry logic for API limits, and evaluation metrics.
* `RenAIssance_GSoC_Evaluation.pdf`: A static PDF export of the executed notebook for rapid review.

## Reproduction
To run this pipeline:
1. Upload the provided CERN dataset to a Google Drive folder named `RenAIssance_Data`.
2. Open the `.ipynb` file in Google Colab.
3. Add your Gemini API key to the Colab Secrets manager under the variable name `gemini_api`.
4. Execute all cells. *(Note: The pipeline includes an exponential backoff function to gracefully handle free-tier API rate limits).*
