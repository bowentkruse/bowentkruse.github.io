---
layout: post
title: Confidence Calibration Part 2 - Post-training Improvements (Includes Google Colab Demo)
---

*Updated September 2025*

Click here to the demo available in Google Colab

[![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bowentkruse/confidenceCalibration/blob/main/demo.ipynb)
<p>&nbsp;</p>

### Introduction
As discussed in [Part 1 of this series](../on_calibration_part_1/index.md), **Confidence Calibration** is a model’s ability to provide an accurate probability of correctness for a given prediction. For example, among predictions made with 90% confidence by a perfectly calibrated model, 90% will actually be correct.

In this second part, we'll discuss the techniques I've found most effective at improving a system's calibration **after** training. There are opportunities to improve a model's calibration before, and during training, but those are beyond the scope of this article. We'll also use the four calibration evaluation **metrics** from Part 1 to evaluate each one.

### Confidence Calibration Techniques
#### Isotonic Regression
Isotonic regression is a non-parametric method that fits a piecewise constant, non-decreasing function to map raw model confidences to calibrated probabilities. It performs well with large validation datasets but can overfit on small or noisy data. Think of it as a post-hoc inference. First, your computer vision model does inference, and then this secondary model predicts actual confidence based on its training. 

```python
def train_isotonic_calibrator(train_confidences: np.ndarray, train_correctness: np.ndarray) -> IsotonicRegression:
    """
    Trains an isotonic regression model for calibration.

    Args:
        train_confidences: array of model confidence scores from training/validation set
        train_correctness: array of binary correctness labels (1 for correct, 0 for incorrect)

    Returns:
        iso_model: trained IsotonicRegression model
    """
    iso_model = IsotonicRegression(out_of_bounds='clip')
    iso_model.fit(train_confidences, train_correctness)
    return iso_model
```
```python
# Fit isotonic regression model
iso_model = train_isotonic_calibrator(train_confidences, train_correctness)
# Use model to calibrate test confidences
iso_calibrated_confidences = iso_model.predict(test_confidences)
```
#### Histogram Binning
Histogram binning divides predictions into fixed-width confidence intervals and assigns the average accuracy of each bin to the predictions within it. It's simple and interpretable but sensitive to the number and placement of bins. If you have a lot of data, I'd try this one first. It's my personal favorite. 
```python
def train_histogram_binning(train_confidences: np.ndarray, train_correctness: np.ndarray, n_bins: int = 10):
    """
    Trains a histogram-binning calibration model.

    Args:
        train_confidences: array of model confidence scores from training/validation set
        train_correctness: array of binary correctness labels (1 for correct, 0 for incorrect)
        n_bins: number of bins to discretize the confidence range [0,1]

    Returns:
        bin_edges: array of bin edges
        bin_accs: array of calibrated probabilities per bin
    """
    # Create equally spaced bins
    bin_edges = np.linspace(0, 1, n_bins + 1)

    # Initialize array to hold the average correctness per bin
    bin_accs = np.zeros(n_bins)

    # Digitize confidences into bins
    bin_ids = np.digitize(train_confidences, bin_edges) - 1
    # Clip to valid indices (in case a value == 1 falls outside last bin)
    bin_ids = np.clip(bin_ids, 0, n_bins - 1)

    for i in range(n_bins):
        mask = bin_ids == i
        if np.any(mask):
            bin_accs[i] = train_correctness[mask].mean()
        else:
            # fallback: use midpoint of bin if empty
            bin_accs[i] = (bin_edges[i] + bin_edges[i + 1]) / 2

    return bin_edges, bin_accs

def apply_histogram_binning(bin_edges: np.ndarray, bin_accs: np.ndarray, test_confidences: np.ndarray):
  """
  Apply histogram-binning calibration to a set of test confidences.

  Args:
      bin_edges: array of bin edges returned by train_histogram_binning
      bin_accs: array of calibrated probabilities per bin returned by train_histogram_binning
      test_confidences: array of model confidence scores to calibrate

  Returns:
      calibrated_probs: array of calibrated probabilities
  """
  n_bins = len(bin_accs)

  # Digitize test confidences
  bin_ids = np.digitize(test_confidences, bin_edges) - 1
  bin_ids = np.clip(bin_ids, 0, n_bins - 1)  # handle edge cases

  # Map each confidence to its calibrated probability
  calibrated_probs = bin_accs[bin_ids]

  return calibrated_probs
```
```python
histo_calibrator = train_histogram_binning(train_confidences, train_correctness, n_bins=10)
histo_calibrated_confidences = apply_histogram_binning(histo_calibrator[0], histo_calibrator[1], test_confidences)
```
#### Bayesian Binning into Quantiles (BBQ)
Honestly, I haven’t used this one in real projects—I just think the name BBQ is fun. BBQ takes histogram binning and adds a Bayesian twist, averaging over different ways to split up the data. This helps avoid overfitting and deals with uncertainty better, especially when you don’t have much data.
```python
class BayesianBinningCalibrator:
    def __init__(self, n_bins=10, alpha_prior=1.0, beta_prior=1.0):
        """
        Bayesian histogram binning calibration.

        Args:
            n_bins (int): Number of bins to discretize confidences into.
            alpha_prior (float): Beta prior alpha parameter.
            beta_prior (float): Beta prior beta parameter.
        """
        self.n_bins = n_bins
        self.alpha_prior = alpha_prior
        self.beta_prior = beta_prior
        self.bins = None
        self.bin_posteriors = None  # list of (alpha, beta) tuples per bin

    def fit(self, confidences, correctness):
        """
        Fit the Bayesian bins using training confidences and correctness labels.

        Args:
            confidences (np.array): Prediction confidences (0-1)
            correctness (np.array): Binary correctness (0 or 1)
        """
        # Define bin edges
        self.bins = np.linspace(0, 1, self.n_bins + 1)
        self.bin_posteriors = []

        # Compute posterior Beta parameters for each bin
        for i in range(self.n_bins):
            mask = (confidences >= self.bins[i]) & (confidences < self.bins[i+1])
            correct_in_bin = np.sum(correctness[mask])
            incorrect_in_bin = np.sum(mask) - correct_in_bin

            alpha_post = self.alpha_prior + correct_in_bin
            beta_post = self.beta_prior + incorrect_in_bin

            self.bin_posteriors.append((alpha_post, beta_post))

    def predict(self, confidences, return_posterior=False):
        """
        Apply Bayesian binning to new confidences.

        Args:
            confidences (np.array): Array of predicted confidences
            return_posterior (bool): If True, return full Beta distributions
        Returns:
            np.array: Calibrated confidences
        """
        calibrated = np.zeros_like(confidences)
        posterior_distributions = []

        for idx, conf in enumerate(confidences):
            # Find bin
            bin_idx = np.searchsorted(self.bins, conf, side='right') - 1
            bin_idx = np.clip(bin_idx, 0, self.n_bins - 1)

            alpha_post, beta_post = self.bin_posteriors[bin_idx]
            posterior_distributions.append(beta(alpha_post, beta_post))

            # Calibrated probability = mean of Beta
            calibrated[idx] = alpha_post / (alpha_post + beta_post)

        if return_posterior:
            return calibrated, posterior_distributions
        return calibrated
```
```python
# Train BBQ calibrator
bbq = BayesianBinningCalibrator(n_bins=10, alpha_prior=1.0, beta_prior=1.0)
bbq.fit(train_confidences, train_correctness)

# Apply to test set
bbq_calibrated_test_confidences = bbq.predict(test_confidences)
```
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
