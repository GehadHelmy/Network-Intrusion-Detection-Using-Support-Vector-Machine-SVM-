# Network Intrusion Detection Using Support Vector Machine (SVM)

## KDD Cup 1999 Dataset — Binary Classification: Normal vs. Attack

## About the Dataset

The KDD Cup 1999 dataset is derived from the 1998 DARPA Intrusion Detection Evaluation Program. It simulates a U.S. Air Force network environment subjected to various cyberattacks. The raw TCP dump data was processed into connection records, offering a rich source for intrusion detection research.

**Dataset Characteristics:**
*   **Number of instances:** Over 10,000
*   **Number of features:** 41
*   **Target variable:** Label (normal or attack type)

## Task

The primary goal is to build a network intrusion detector capable of distinguishing between:

*   **Normal connections:** Legitimate network traffic.
*   **Attack connections:** Malicious or suspicious activity.

This notebook specifically focuses on **binary classification** (Normal vs. Attack), consolidating various attack types into a single 'attack' class due to class imbalance and the inherent challenges of multi-class SVM with highly skewed data.

## Methodology

The project followed a structured machine learning pipeline:

### 1. Data Inspection

*   Loaded the `kdd_dataset.csv` into a pandas DataFrame.
*   Examined data shape, information (data types, non-null counts), and descriptive statistics.
*   Checked for missing values (none found) and duplicate rows (21,723 duplicates identified but not removed at this stage, as the focus is on flow-based anomaly detection rather than unique connection instances).

### 2. Data Splitting

*   The target variable `label` was converted from multi-class to a binary `'normal'` vs. `'attack'` target.
*   The dataset was split into training and testing sets (80/20 split) using `stratify=y_binary` to preserve class proportions due to the severe imbalance in the original multi-class labels.

### 3. Exploratory Data Analysis (EDA)

*   **Class Distribution:** Analyzed the distribution of the target variable, confirming severe class imbalance among the original multi-class labels, which justified the binary classification approach.
*   **Numerical Features:** Univariate analysis revealed extreme right-skewness and the presence of significant outliers in most numerical features (e.g., `duration`, `src_bytes`, `dst_bytes`). Features like `land`, `wrong_fragment`, and `urgent` were found to be predominantly zero.
*   **Categorical Features:** Explored `protocol_type`, `service`, and `flag`. `TCP` was dominant, `HTTP` and `Private` were common services, and `SF` and `S0` flags were frequent. High cardinality in `service` was noted, indicating a need for appropriate encoding.
*   **Correlation & Redundancy:** A correlation matrix highlighted high multicollinearity among several feature pairs (e.g., `serror_rate` and `srv_serror_rate`), suggesting opportunities for dimensionality reduction.
*   **Zero-Variance & Percentage of Zeros:** Identified features with 100% zero values (`num_outbound_cmds`, `is_host_login`) and highly sparse features (>99% zeros), which offer little predictive power.

### 4. Data Cleaning (Outlier Treatment)

*   **IQR Method with Factor 3.0:** Outliers in numerical features were handled using the Interquartile Range (IQR) method with a robust factor of 3.0 (instead of 1.5). This approach was chosen to accommodate legitimate extreme values in network traffic and to cap (Winsorize) outliers rather than dropping them, preserving data integrity.
*   **Statistics from Training Data Only:** Outlier fence values were computed exclusively from the training set and then applied to both training and test sets to prevent data leakage.

### 5. Feature Engineering

*   **Categorical Encoding:** `OrdinalEncoder` was selected for categorical features (`protocol_type`, `service`, `flag`). This choice was made to manage the high cardinality of the `service` feature (70 categories) and prevent an explosion in dimensionality that one-hot encoding would cause. The encoder was fit only on training data.
*   **StandardScaler:** Applied `StandardScaler` to all numerical and encoded categorical features. This is crucial for distance-based algorithms like SVM, ensuring features with larger ranges don't dominate the model. The scaler was fit only on training data.

### 6. Pipeline Construction

A `sklearn` pipeline was constructed to streamline the preprocessing and modeling steps, ensuring no data leakage and reproducibility.

*   **`DropColumns` Transformer:** A custom transformer was implemented to remove the identified zero-variance features (`num_outbound_cmds`, `is_host_login`).
*   **`ColumnTransformer`:** Applied `StandardScaler` to numerical features and `OrdinalEncoder` to categorical features.
*   **Full Preprocessor Pipeline:** Chained `DropColumns` and `ColumnTransformer` into a `preprocessor` pipeline.

### 7. Dimensionality Reduction (PCA)

*   **Why PCA?** Addressed multicollinearity and reduced dimensionality, which is beneficial for SVM training efficiency. All features were already scaled, meeting PCA's requirements.
*   **Elbow Method:** PCA was applied, and the cumulative explained variance was plotted. An elbow point was identified at **9 components**, retaining **95% of the variance**, while reducing the feature space from 39 to 9 dimensions (a 76.9% reduction). This choice balances information retention with dimensionality reduction.

### 8. SVM Model Development

*   **Kernel Selection:** Four SVM kernels were evaluated: LinearSVC, RBF, Polynomial, and Sigmoid.
*   **Kernel Comparison Results:** The **RBF kernel** achieved the best overall performance, with near-perfect F1 scores and no signs of overfitting, consistent with literature on KDD Cup 1999. `class_weight='balanced'` was used to mitigate class imbalance.
*   **Final Pipeline:** The complete pipeline was assembled, incorporating the `preprocessor`, `PCA` with `n_components=9`, and `SVC` with the `RBF` kernel and `class_weight='balanced'`.

### 9. Model Evaluation

The final RBF SVM model was evaluated using a comprehensive set of metrics, with a focus on Recall due to its critical importance in intrusion detection (minimizing missed attacks).

*   **Classification Report:** Showed near-perfect scores (1.00 for precision, recall, and F1) for both 'normal' and 'attack' classes.
*   **Confusion Matrix:**
    *   **True Negatives:** 6957 (Normal traffic correctly allowed)
    *   **True Positives:** 8365 (Real attacks correctly flagged)
    *   **False Positives:** 21 (Legitimate connections wrongly flagged – false alarms)
    *   **False Negatives:** 7 (Real attacks that went undetected – missed attacks). This represents a 99.92% detection rate, which is excellent for a critical security system.
*   **ROC Curve & AUC:** The ROC curve demonstrated almost perfect separation, with an **AUC of 0.9994**, indicating exceptional discriminative power.

## Summary of Results

| Metric    | Score  |
|-----------|--------|
| Accuracy  | 0.9982 |
| Precision | 0.9990 |
| Recall    | 0.9970 |
| F1 Score  | 0.9980 |
| ROC-AUC   | 0.9994 |

The RBF SVM model achieved outstanding performance on the KDD Cup 1999 dataset for binary intrusion detection, demonstrating high accuracy, precision, recall, and F1-score, with a very low rate of missed attacks. These results are highly competitive and align with benchmarks for this dataset, highlighting the effectiveness of the chosen methodology and the RBF kernel.
