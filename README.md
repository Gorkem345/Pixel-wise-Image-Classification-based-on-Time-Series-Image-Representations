# Pixel-wise-Image-Classification-based-on-Time-Series-Image-Representations

## 📖 Project Overview
This project evaluates a machine learning classifier capable of performing pixel-wise classification of plant species based on vegetation index time series. By monitoring the leaf-changing patterns of Cerrado-savanna vegetation through daily digital imaging, the code extracts color information at the individual plant level to analyze phenological changes.

The primary goal is to classify regions of interest (ROIs) of different plant species using K-Nearest Neighbors (KNN) applied to raw time series data and Recurrence Plots (RPs).

## ⚙️ Dependencies
You can run git clone "the address on github" to install the code on to your device.
Ensure you have the following Python libraries installed:
* numpy
* matplotlib
* scipy
* scikit-learn
* Pillow (PIL)
* pygame (for specific variable imports, if required by your environment)

## 📂 Dataset Structure
The code expects the dataset to be located in a `raw/` directory relative to the project folder:
* **Images:** Located in `raw/images/1/`. Contains the JPEG images captured daily.
* **Masks:** Located in `raw/masks/`. Contains binary PGM masks defining the regions of interest (ROIs) for the 6 different plant species.

## 🛠️ Configuration Modes and Usage
The notebook is designed to be highly modular. By changing the boolean flags and integer values in the Configuration section of the code, you can experiment with different data representations and preprocessing steps.

Here is a detailed explanation of how to use the different modes and what they do under the hood:

### 1. CHANNELS_TO_BE_USED_MODE
* **What it does:** This mode determines which specific color channels from the RGB Chromatic Coordinates ($r_{cc}$, $g_{cc}$, $b_{cc}$) will be fed into the classification model.
* **How to use it:** Set the variable to an integer between 0 and 5.
  * 0: All 3 channels (Red, Green, Blue)
  * 1: Red Channel only
  * 2: Green Channel only
  * 3: Blue Channel only
  * 4: Red & Green Channels
  * 5: Green & Blue Channels
* **Why use it?** Different plant species exhibit distinct color variations during phenological events (e.g., leaf flushing might be highly visible in the green channel, while senescence might spike in the red channel). This mode allows you to isolate the most informative color indices and drop noisy ones.

### 2. USE_SLIDING_WINDOW
* **What it does:** Applies a moving average smoothing filter to the 1D time series before generating the recurrence plots or feeding them to the classifier.
* **How to use it:** Set to `True` to enable smoothing, or `False` to use the raw, noisy time-series data.
* **Why use it?** Outdoor imaging sensors are subject to noise caused by weather, lighting changes, and camera artifacts. The sliding window acts as a filter that blocks out temporary noise and allows the actual phenological trends to show through, making the signal much more stable for the KNN model.

### 3. USE_HISTOGRAM_FOR_PREDICTION
* **What it does:** Instead of flattening the 140 × 140 continuous Recurrence Plot into a massive 58,800-dimensional feature vector, this mode computes a cumulative histogram of the recurrence plot values (e.g., summarizing each channel into 32 bins, resulting in a compact 96-dimensional vector).
* **How to use it:** Set to `True` to use the histogram representation, or `False` to use the full flattened recurrence plot.
* **Why use it?** Flattening 2D recurrence plots results in massive memory costs and makes the KNN algorithm incredibly slow due to the "curse of dimensionality." The histogram summarizes the patterns of the image rather than the exact temporal positions. This makes the model highly resilient to phase shifts in the data and drastically speeds up training and prediction times.

### 4. USE_PCA (Principal Component Analysis)
* **What it does:** Applies standard scaling (mean 0, variance 1) followed by PCA to reduce the dimensionality of the feature vectors while retaining over 95% of the variance.
* **How to use it:** Set to `True` to apply PCA, or `False` to skip it. *(Note: If USE_HISTOGRAM_FOR_PREDICTION is set to True, the code automatically skips PCA, as reducing a 96-dimensional vector is unnecessary).*
* **Why use it?** If you choose to run the model on the full, raw recurrence plot feature vectors (`USE_HISTOGRAM_FOR_PREDICTION = False`), PCA is essentially mandatory to make the KNN computationally feasible. It extracts the most important features, saving both time and space complexity.

### 5. SELECTED_HOUR
* **What it does:** The dataset contains 5 images per hour. This variable selects one specific hour of the day to build the daily time series across the 37 days of observation.
* **How to use it:** Set to an integer representing the hour (e.g., 12).
* **Why use it?** Setting it to 12 ensures that the images used are from noon, which generally provides the most consistent lighting conditions (minimizing shadows and sun glare) across the observation period.
* **Note** Some hours do not work because there are not exactly 37 images for that hour. Some hours known to work are: 6, 8 and 12

## 🚀 Running the Code
1. Ensure your `raw/` data directory is placed accurately relative to the notebook.
2. Tweak the variables in the Configuration cell according to your experimental needs.
3. Run the notebook sequentially. The code features built-in error handling to alert you if ROIs are empty or if directories are misplaced.