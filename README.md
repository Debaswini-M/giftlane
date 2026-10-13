# Customers Like You
## Product Recommendations and Customer Segmentation

## 1. Project Overview

This project explores customer purchasing behaviour using the Online Retail II dataset. The goal is to identify similarities between products and customers, build a simple product recommendation system, and discover customer segments based on purchasing patterns.

The project combines data engineering and machine learning techniques, with a focus on preparing transaction data, constructing a sparse customer–product matrix, applying dimensionality reduction, evaluating recommendations, and visualising customer segments.

### Objectives

- Clean and profile the transaction dataset.
- Separate training and test data using a chronological split.
- Build and analyse a sparse customer–product matrix.
- Apply Truncated Singular Value Decomposition (SVD) for product recommendations.
- Compare SVD with Principal Component Analysis (PCA).
- Evaluate recommendations against a best-seller baseline.
- Explore customer segments using a two-dimensional SVD representation and K-Means clustering.

## 2. Dataset

The project uses the **Online Retail II** Excel workbook, containing two worksheets:

- `Year 2009-2010`
- `Year 2010-2011`

The two worksheets were combined into one dataset.

| Dataset property | Result |
|---|---:|
| Rows in 2009–2010 sheet | 525,461 |
| Rows in 2010–2011 sheet | 541,910 |
| Total raw rows | 1,067,371 |
| Number of columns | 8 |

The columns used include invoice number, stock code, product description, quantity, invoice date, unit price, customer ID, and country.

The input workbook is expected at:

`data/raw/online_retail_II.xlsx`

If the workbook is not included in the submitted project, it must be obtained separately and placed at this path, or the notebook's input path must be updated.

## 3. Data Cleaning and Preparation

The original dataset contained missing customer IDs, cancellation invoices, non-positive quantities and prices, and stock codes representing non-product transactions.

The following cleaning rules were applied:

1. Remove cancellation invoices whose invoice numbers begin with `C`.
2. Remove transactions with quantities less than or equal to zero.
3. Remove transactions with prices less than or equal to zero.
4. Remove rows with missing customer IDs.
5. Remove the explicitly identified non-product stock codes: `POST`, `M`, `D`, and `BANK CHARGES`.
6. Convert invoice dates to datetime values and remove invalid dates.
7. Record the number of rows before and after each cleaning rule in `outputs/cleaning_log.csv`.

### Cleaning results

| Property | Result |
|---|---:|
| Raw rows | 1,067,371 |
| Rows after cleaning | 802,965 |
| Rows removed | 264,406 |
| Unique customers after cleaning | 5,862 |
| Unique stock codes after cleaning | 4,627 |
| Duplicate rows identified | 26,060 |

Duplicate rows were identified but not removed. The cleaning rules remove the explicitly specified non-product codes; they do not guarantee that every possible non-merchandise transaction has been identified.

The original data profile and cleaning log are saved in the `outputs/` directory.

## 4. Time-Based Train/Test Split

To evaluate recommendations using later transactions, the cleaned data was sorted by invoice date and divided chronologically.

The latest transaction was recorded on **9 December 2011 at 12:50**. The cutoff was set two calendar months earlier, on **9 October 2011 at 12:50**.

| Split | Rows | Date range |
|---|---:|---|
| Training | 684,357 | 1 December 2009 – 9 October 2011, before the cutoff |
| Test | 118,608 | 9 October 2011 onward, including the cutoff |

The split was validated to ensure that training transactions occur before the cutoff and test transactions occur at or after it.

The training data was used to construct the customer–product matrix, select eligible products, and fit the recommendation model. Held-out test purchases were used for evaluation rather than model fitting.

For recommendation evaluation, a test purchase was eligible only when the customer was known in the training data, the product belonged to the training-defined eligible product catalogue, and the customer had not previously purchased that product in training.

## 5. Sparse Customer–Product Matrix

A customer–product matrix was constructed from training transactions only. Each row represents a customer, and each column represents a product stock code. The quantity purchased was aggregated by customer and product, and a `log1p` transformation was applied to reduce the influence of very large purchase quantities.

Products purchased by at least 10 distinct training customers were retained.

| Matrix property | Result |
|---|---:|
| Customers | 5,484 |
| Products | 4,537 |
| Matrix shape | 5,484 × 4,537 |
| Non-zero entries | 417,809 |
| Sparsity | 98.32% |
| Estimated dense memory | 189.83 MB |
| Sparse matrix memory | 4.80 MB |

Most customer–product combinations have no recorded purchase, making a sparse representation appropriate. The sparse matrix requires substantially less memory than a dense representation.

The matrix was saved as `Customer_product_matrix.npz`. Customer and product index mappings were also saved as JSON files to support reproducibility.

## 6. Method Selection: Truncated SVD

Truncated Singular Value Decomposition (SVD) was selected as the primary dimensionality-reduction method.

The customer–product matrix is sparse, and Truncated SVD can operate directly on sparse input without requiring the centering step used by conventional PCA. It represents customers and products through a smaller set of latent factors that capture patterns in purchasing behaviour.

Product similarity was estimated using cosine similarity between product factor vectors. Customer recommendations were generated by ranking products according to their total similarity to products the customer purchased in training, excluding products already purchased.

PCA was fitted on the same training matrix for comparison.

## 7. SVD Component Experiment

The following component counts were evaluated: 10, 20, 50, 100, and 200.

| Components (k) | Approximate relative reconstruction error | Factor storage (MB) |
|---:|---:|---:|
| 10 | 0.8766 | 0.76 |
| 20 | 0.8466 | 1.53 |
| 50 | 0.7905 | 3.82 |
| 100 | 0.7289 | 7.65 |
| 200 | 0.6409 | 15.30 |

The approximate relative reconstruction error decreases as the number of components increases. This is expected because a larger latent representation captures more of the matrix's energy, but it also requires more storage.

**Measurement note:** The reported error is an energy-based estimate calculated from the retained singular values. It is not a direct calculation of the full reconstructed matrix's error. The factor-storage figures estimate the memory required for the transformed customer factors and product components; they are not measurements of total model memory.

The plot is saved as `svd_reconstruction_curve.png`.

## 8. Recommendation Evaluation

The SVD-based recommender was evaluated against a best-seller baseline using eligible held-out test purchases.

The metric was **Hit Rate@10**: the proportion of eligible evaluation customers for whom at least one held-out eligible product appears in the top 10 recommendations.

| Evaluation metric | Result |
|---|---:|
| Eligible evaluation customers | 1,913 |
| SVD model Hit Rate@10 | 20.3346% |
| Best-seller baseline Hit Rate@10 | 17.4072% |
| Absolute improvement | 2.9274 percentage points |
| Relative improvement | Approximately 16.8% |
| Time to score all evaluation customers | 28.97 seconds |

The SVD-based recommendation method achieved a higher Hit Rate@10 than the best-seller baseline on this test split. The result suggests that purchase-pattern similarity adds useful ranking information beyond popularity alone.

These results apply to the implemented evaluation procedure and this particular held-out period. They do not guarantee the same improvement on other time periods or customer populations.

The scoring time is the measured time for the evaluation loop in the current environment and may vary with hardware and software configuration.

## 9. PCA Comparison

PCA was fitted using 50 components on the same training matrix for comparison.

| PCA metric | Result |
|---|---:|
| Number of components | 50 |
| Explained variance ratio, summed across components | 34.8584% |
| Factor storage estimate | 3.82 MB |
| Reported fitting time | 0.31 seconds |

The PCA explained variance ratio and the SVD energy-based reconstruction estimate are different measurements and should not be compared as if they were the same metric.

SVD was retained as the primary method because it works directly with the sparse customer–product matrix and supports the factor-based similarity approach used in the recommender. PCA provides a useful comparison, but the reported results alone do not establish that one method is universally superior.

The PCA/SVD comparison report is intended to be saved as `outputs/pca_svd_comparison.csv` when the corresponding notebook cell is run successfully.

## 10. Customer Segmentation

A two-component Truncated SVD representation of the customer–product matrix was used to project customers into two dimensions. K-Means clustering was then applied to the two-dimensional customer coordinates using four clusters.

The plot is saved as `customer_segments.png`.

The observed segment sizes were:

| Segment | Number of customers |
|---|---:|
| Segment 1 | 1,114 |
| Segment 2 | 4,005 |
| Segment 3 | 353 |
| Segment 4 | 12 |
| **Total** | **5,484** |

The clusters are uneven in size. Segment 2 contains most customers, while Segment 4 contains only 12 customers and appears separated from the main concentration in the projection. This small cluster may represent customers with unusually distinct purchasing patterns, but it should be interpreted cautiously because it contains very few customers.

The notebook also reports leading products and countries for each segment based on training transactions. These summaries provide a starting point for interpreting the groups, rather than definitive customer personas.

Customer IDs are retained internally for analysis but should not be included in the submitted presentation or public-facing figures.

## 11. Project Structure

```text
giftlane/
├── MINIPROJECT.ipynb
├── README.md
├── requirements.txt
├── data/
│   └── raw/
│       └── online_retail_II.xlsx
├── Customer_product_matrix.npz
├── customer_index.json
├── product_index.json
├── svd_reconstruction_curve.png
├── customer_segments.png
├── presentation/
│   └── customers_like_you_presentation_with_segments.pptx
└── outputs/
    ├── data_profile.csv
    ├── cleaning_log.csv
    ├── split_report.csv
    ├── matrix_report.csv
    ├── train.parquet
    ├── test.parquet
    └── pca_svd_comparison.csv
```

Some generated files may be absent until the notebook cells that create them have been run successfully. Verify the actual contents of `outputs/` before submission.

## 12. Setup and Reproducibility

### Requirements

The project uses Python and the packages listed in `requirements.txt`, including:

- pandas
- NumPy
- SciPy
- scikit-learn
- Matplotlib
- openpyxl
- PyArrow

### Running the project

1. Clone or copy the project into your local environment.
2. Place `online_retail_II.xlsx` in `data/raw/`, or update the notebook's `file_path`.
3. Create and activate a Python virtual environment.
4. Install dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

5. Open `MINIPROJECT.ipynb` in VS Code or Jupyter.
6. Run the notebook cells in order, from data loading and cleaning through model evaluation and report generation.
7. Check that the expected CSV reports, matrix file, plots, and presentation are present.

The notebook uses a fixed random state for the decomposition and clustering experiments where specified. Runtime and exact results can vary across software versions, hardware, and numerical implementations.

## 13. Limitations and Future Improvements

- Duplicate transactions were identified but not removed.
- The non-product filtering rule covers only explicitly specified stock codes.
- Product eligibility and the customer–product matrix are defined using training data, so recommendations are limited to the eligible training catalogue.
- Hit Rate@10 does not measure recommendation ranking quality beyond whether at least one relevant product appears in the top 10.
- The recommendation approach uses aggregate product similarity and does not incorporate product descriptions, customer demographics, or explicit feedback.
- The customer clusters are based on a two-dimensional representation, which may omit useful purchasing information.
- Segment 4 is very small and should not be treated as a robust general customer category without further validation.
- Additional evaluation across multiple temporal splits, alternative recommendation metrics, and different clustering settings would help assess the stability of the results.

## 14. Conclusion

This project demonstrates a reproducible workflow for cleaning retail transaction data, building a sparse customer–product matrix, applying dimensionality reduction, evaluating recommendations, and exploring customer segments.

The matrix contains 5,484 customers and 4,537 products, with 98.32% sparsity. The SVD-based recommender achieved a Hit Rate@10 of 20.3346%, compared with 17.4072% for the best-seller baseline, on the implemented held-out evaluation. PCA was also evaluated as a comparison, and four customer clusters were explored using a two-component SVD projection.

Overall, the results provide a practical starting point for personalised product recommendations and customer behaviour analysis, while leaving room for stronger validation and richer recommendation methods.