# Technical Implementation Details

## NPM Large-Scale Hallucinated Package Detection Experimental Dataset

## Overview

This directory contains experimental datasets for NPM package hallucination detection research. These datasets include a large number of technical questions and answers, primarily used for researching and analyzing hallucination phenomena that may occur when LLMs process technical questions.

## Dataset Construction and Details

By leveraging the extensive knowledge base of **Stack Overflow**, the famous online Q&A service for software programmers, we have collected a substantial dataset of over **10,000 JavaScript-related questions** distributed across two main files:

#### Data Collection Process

The dataset construction follows a systematic approach:

1. **Source Selection**: Questions are sourced directly from Stack Overflow's JavaScript tag, ensuring relevance to real-world programming scenarios. We specifically target questions with sufficient engagement (minimum view count and interaction metrics) to ensure quality baseline while maintaining diversity.

2. **Scale and Scope**: The collection encompasses over 10,000 questions, providing sufficient diversity for comprehensive analysis. This sample size was determined through statistical power analysis to ensure representative coverage of JavaScript development patterns.

3. **No Pre-filtering**: Questions are gathered without any content-based filtering or quality screening to maintain the raw, authentic nature of developer inquiries. This includes questions with varying levels of clarity, completeness, and technical accuracy to mirror real-world developer interactions.

4. **Diversity Assurance**: This unfiltered approach ensures the dataset captures the full spectrum of JavaScript-related queries, from basic syntax questions to complex implementation challenges. The dataset includes questions across different JavaScript frameworks (React, Vue, Angular), runtime environments (Node.js, browser), and application domains (web development, mobile apps, server-side applications).
---

# Performance Metrics for the LightGBM IP Classifier

This document details the performance of our LightGBM classifier, designed to distinguish real developer IP addresses from automated scripts based on access patterns and other features.

## Methodology

To rigorously evaluate the classifier, we established a ground truth corpus and validated it through a multi-step process:

### Annotation
A sample of **100 IPs** was manually annotated by experts. A detailed annotation guide was established beforehand to ensure consistency.

### Categorization
Each IP was classified into one of four categories:
- **Real User**: IPs with strong evidence of human developer activity.
- **Automated Script**: IPs with clear signs of automated behavior (e.g., crawlers, bots).
- **Weak User Signal**: IPs with some indicators of human activity but insufficient for a definitive classification.
- **Weak Script Signal**: IPs with some indicators of automated activity but not conclusive.

### IP Address Classification Guidelines

A detailed annotation guide was established beforehand to ensure consistency in the classification process. The classification criteria are organized into three main categories as shown in the following table:

#### Table: IP Address Classification Rule

| **Type** | **Category** | **Attribute / Condition / Observation** |
|----------|--------------|------------------------------------------|
| **Static Attribute** | IP Reputation | Officially marked as malicious |
| | Threat Type | Associated threat intelligence indicates specific keywords |
| | Network Context | Identified as originating from a server environment/ an end-user environment |
| | Cloud Services | Attributed to a Cloud Service Provider AND evidence suggests non-individual user |
| | ISP Type | Attributed to a conventional Internet Service Provider |
| | Benign Status | No known malicious markers AND no identified threat types associated with the IP |
| **Behavioral Characteristic** | Connection Patterns | Predominance of numerous, very short-duration connections |
| | | Connection durations are exactly identical across multiple distinct sessions |
| | Operating System | Identified as a server-grade OS (e.g., Linux Server variants, Windows Server) |
| | | Identified as a desktop-grade OS (e.g., Windows 10/11, macOS, common Linux desktops) |
| | User Account Type | Username indicative of a service account (e.g., "admin", "service", "system" and "runner") |
| | | Username appears personalized or non-generic |
| | Hostname Structure | Hostname follows a common server naming convention (e.g., 'srv-', 'db-', patterned, cloud-instance IDs) |
| | | Hostname appears personalized or non-generic |
| | CPU Architecture | CPU model identified as server-grade (e.g., Xeon, EPYC) |
| | | CPU model identified as consumer-grade (e.g., Core i-series, Ryzen) |
| | Display/GUI | Absence of a graphical user interface indicated (e.g., "N/A (Linux/No X)", no screen resolution) |
| | | Presence of a common desktop display resolution |
| | Session Interactivity | Evidence of an interactive session (e.g., real-time command execution, user-driven events) |
| | | Evidence of a non-interactive or scripted session (e.g., automated task execution) |
| | Process Footprint | Presence of web browser processes detected |
| | | Absence of web browser processes detected |
| | | Presence of office productivity software processes (e.g., Microsoft Office suite) |
| | Process Enumeration | Failure to retrieve or access process information |
| | System Uptime | Exceptionally long system uptime (e.g., continuously operational for >30 days) |
| | Feature Aggregation (Server) | ≥3 distinct behavioral characteristics indicative of a server environment present |
| | Feature Aggregation (Desktop) | ≥3 distinct behavioral characteristics indicative of a desktop/user environment present |
| **Special Case Handling** | VPN/Proxy Usage | Accompanied by demonstrable user-like behavioral patterns/ Lacks demonstrable user-like behavioral patterns; activity appears automated |
| | Cloud Service User Behavior | IP originates from a cloud service, but exhibits clear, strong user-like behaviors/ no user behavior, and current cumulative score is less than 5 |



## LLM Interaction and Package Extraction

### LLM Interaction Methodology

A key aspect of our approach involves how LLMs are utilized in analyzing these questions.

Specifically, LLMs are instructed to generate code snippets only when a code-based solution is deemed necessary and appropriate for addressing the query.

In instances where a question can be resolved through explanation, clarification or points to conceptual misunderstandings rather than requiring a tangible code implementation, the LLMs are programmed to simply respond with `"None"`.

This deliberate methodology, while potentially resulting in a lower overall count of recommended software packages by LLMs, is designed to more accurately reflect real-world scenarios.

### Extract Package Name

In order to get hallucinated packages, it is necessary to extract package names from LLMs response. To address this issue, we adopt the following approach to identify package name in LLMs response.

First, we query selected LLMs that contain a specific system message with prompts. The System Message for LLMs is:

> *"You are a coding assistant that generates JavaScript code. Provide only the code and add additional explanatory text only when absolutely necessary (e.g., 'If you determine that packages are needed to run the code, please provide the command in the format npm install xx, where xx represents one or more package names.') If no code is required to answer the question, simply reply 'None'."*

The `npm install` command are searched in the generated JavaScript code through regular expression and get package names. The final step involves querying package names against the npm registry. A package is subsequently classified as hallucinated if the registry lookup confirms its nonexistence.

## Classification Report

The following table presents the mean performance metrics and their standard deviations across all folds:

### Table: IP Classifier Performance (10-Fold Cross-Validation)

| **Metric** | **Mean** | **Standard Deviation** |
|------------|----------|------------------------|
| Accuracy | 0.950 | 0.051 |
| Weighted Precision | 0.962 | 0.045 |
| Weighted Recall | 0.950 | 0.051 |
| Weighted F1-Score | 0.955 | 0.048 |

### Metrics Explanation

- **Precision**: Measures the accuracy of positive predictions. For example, a precision of 1.000 for Real User means every IP the model classified as a Real User was correct.
- **Recall**: Measures the model's ability to identify all relevant instances of a class. For example, a recall of 1.000 for Real User means the model successfully identified every true Real User in the dataset.
- **F1-Score**: The harmonic mean of precision and recall, providing a single metric that balances both.
- **Support**: The number of actual occurrences of the class in the dataset.

---
