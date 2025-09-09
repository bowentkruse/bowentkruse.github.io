---
layout: post
title: Confidence Calibration Part 2: Post-training Improvements (Includes Google Colab Demo)
---

*Updated September 2025*

Click here to the demo available in Google Colab

[![Colab](https://colab.research.google.com/assets/colab-badge.svg)](bowentkruse/confidenceCalibration/blob/main/demo.ipynb)
<p>&nbsp;</p>

### Introduction
As discussed in [Part 1 of this series](../on_calibration_part_1/index.md), **Confidence Calibration** is a model’s ability to provide an accurate probability of correctness for a given prediction. For example, among predictions made with 90% confidence by a perfectly calibrated model, 90% will actually be correct.

In this second part, we'll discuss the techniques I've found most effective at improving a system's calibration **after** training. There are opportunities to improve a model's calibration before, and during training, but those are beyond the scope of this article. We'll also use the four calibration evaluation **metrics** from Part 1 to evaluate each one.

### Confidence Calibration Techniques
#### Isotonic Regression
Isotonic regression is a non-parametric method that fits a piecewise constant, non-decreasing function to map raw model confidences to calibrated probabilities. It performs well with large validation datasets but can overfit on small or noisy data. Think of it as a post-hoc inference. First, your computer vision model does inference, and then this secondary model predicts actual confidence based on its training. 

#### Histogram Binning
Histogram binning divides predictions into fixed-width confidence intervals and assigns the average accuracy of each bin to the predictions within it. It's simple and interpretable but sensitive to the number and placement of bins. If you have a lot of data, I'd try this one first. It's my personal favorite. 

#### Bayesian Binning into Quantiles (BBQ)
Honestly, I haven’t used this one in real projects—I just think the name BBQ is fun. BBQ takes histogram binning and adds a Bayesian twist, averaging over different ways to split up the data. This helps avoid overfitting and deals with uncertainty better, especially when you don’t have much data.

#### Comparing Results
Let's take our raw (no calibration) confidence calibration evaluation metrics and compare them against the three post-hoc calibration techniques implemented.

| Method     | ECE       | MCE       | Brier     |
|------------|-----------|-----------|-----------|
| raw        | 0.125523  | 0.407441  | 0.029312  |
| isotonic   | 0.006802  | 0.352941  | 0.013082  |
| histogram  | 0.002862  | 0.318681  | 0.012970  |
| bbq        | 0.003185  | 0.319829  | 0.012977  |

#### Conclusion
Any calibration technique is better than none. Also, in production, your customer will notice ECE every day, but MCE is what will stand out on the days things go wrong. As always, data science is a people industry, choose your metrics accordingly.
