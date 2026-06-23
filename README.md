# Pixel-wise-Image-Classification-based-on-Time-Series-Image-Representations

## AI statement
AI was used to generate the structure of this READ ME section as well as to help find python functions and packages that we did not yet know. All the text in the cells next to code we wrote ourselves. Therefore, all analyses and interpretations in this notebook are our own. 

##  Dataset
The code expects the dataset to be located in a `raw/` directory relative to the project folder:
* **Images:** Located in `raw/images/1/`. Contains the JPEG images of the vegetation
* **Masks:** Located in `raw/masks/`. Contains binary PGM masks defining the regions of interest (ROIs) for the 6 different plant species

## Modes and Usage
We investigated several methods to get to best level of classification. To still allow access the methods that did not give the best results, we included modes in the project. By changing the boolean flags and integer values in the Configuration section of the code, you can experiment with different data representations and preprocessing steps. They are explained below:

### 1. CHANNELS_TO_BE_USED_MODE
* **What it does:** This mode determines which specific color channels from the RGB Chromatic Coordinates ($r_{cc}$, $g_{cc}$, $b_{cc}$) will be fed into the classification model.
* **How to use it:** Set the variable to an integer between 0 and 5.
  * 0: All 3 channels (Red, Green, Blue)
  * 1: Red Channel only
  * 2: Green Channel only
  * 3: Blue Channel only
  * 4: Red & Green Channels
  * 5: Green & Blue Channels
* **Why use it?** The red and green channels are better predictors than the blue channel when looking for senesence and leaf browning in vegetation. By removing channels that mostly generate noise (blue) we improve accuracy. 

### 2. USE_SLIDING_WINDOW
* **What it does:** Applies a moving average smoothing filter to the 1D time series before feeding them to the classifier. We do not use the smoothed version for the recurrence plots
* **How to use it:** Set to `True` to enable smoothing, or `False` to use the raw (noisy) time-series data.
* **Why use it?** The sliding window acts is a filter that blocks out temporary noise, making the signal more stable for the KNN model

### 3. USE_HISTOGRAM_FOR_PREDICTION
* **What it does:** Instead of flattening the 140 × 140 continuous Recurrence Plot into a 58,800-dimensional feature vector, this mode computes a cumulative histogram of the recurrence plot values (summarizing 3 channels into 32 bins, resulting in a compact 96-dimensional vector)
* **How to use it:** Set to `True` to use the histogram representation, or `False` to use the full flattened recurrence plot.
* **Why use it?** Flattening 2D recurrence plots results in high memory costs and makes the KNN algorithm slow due to the "curse of dimensionality", the histogram summarizes the patterns of the image rather than the exact temporal positions

### 4. USE_PCA (Principal Component Analysis)
* **What it does:** Applies standard scaling (mean 0, variance 1) followed by PCA to reduce the dimensionality of the feature vectors
* **How to use it:** Set to `True` to apply PCA, or `False` to skip it. *(Note: If USE_HISTOGRAM_FOR_PREDICTION is set to True, the code automatically skips PCA, as reducing a 96-dimensional vector is unnecessary).*
* **Why use it?** If you choose to run the model on the full, raw recurrence plot feature vectors (`USE_HISTOGRAM_FOR_PREDICTION = False`), PCA is essentially mandatory to make the KNN computationally feasible. It extracts the most important features, saving both time and space complexity

### 5. SELECTED_HOUR
* **What it does:** The dataset contains hourly photos for every hour of the day with sunlight. This variable selects one specific hour of the day to build the daily time series across the 37 days of observation
* **How to use it:** Set to an integer representing the hour (e.g., 12).
* **Why use it?** Setting it to 12 ensures that the images used are from noon, which generally provides the most consistent lighting conditions (minimizing shadows and sun glare) across the observation period.
* **Note** Some hours do not work because there are not exactly 37 images for that hour due to missing photos. Some hours known to work are: 6, 8 and 12

## Running the Code
1. Ensure your `raw/` data directory is placed accurately relative to the notebook.
2. Tweak the variables in the Configuration cell according to your experimental needs.
3. Run the notebook. The code features built-in error handling to alert you if ROIs are empty or if directories are misplaced.