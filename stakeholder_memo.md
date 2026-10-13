# Stakeholder Memo: Customers Like You

**Project:** Product Recommendations and Customer Segmentation  
**Dataset:** UCI Online Retail II  
**Purpose:** Evaluate whether purchase patterns can support more relevant product recommendations and customer segmentation.

## Executive Summary

This project explored how historical customer purchases can be used to recommend products and identify groups of customers with similar purchasing patterns.

A recommendation model based on Truncated Singular Value Decomposition (SVD) achieved a Hit Rate@10 of **20.33%**, compared with **17.41%** for a best-seller baseline. This represents an improvement of approximately **2.93 percentage points**, or **16.8% relative to the baseline**, on the historical evaluation dataset.

The results suggest that customer purchase patterns contain useful information beyond overall product popularity. However, the evaluation is based on historical transactions and does not establish that the recommendations will increase sales in a live business environment.

## 1. Data and Approach

The analysis combined both sheets of the Online Retail II workbook, containing **1,067,371 original transaction rows**. After cleaning, **802,965 rows** remained.

The data was divided chronologically into training and test periods. A sparse customer–product matrix was constructed using training transactions only, with **5,484 customers**, **4,537 products**, and **98.32% sparsity**.

Truncated SVD was selected as the primary method because it can represent the sparse purchasing matrix in a lower-dimensional factor space. Product similarities were used to generate recommendations from each customer's previous purchases. PCA was also fitted for comparison.

## 2. Key Findings

### Product recommendations

The model was evaluated on **1,913 eligible customers** using held-out purchases.

| Evaluation metric | Result |
|---|---:|
| SVD Hit Rate@10 | 20.33% |
| Best-seller baseline Hit Rate@10 | 17.41% |
| Improvement | 2.93 percentage points |
| Relative improvement | Approximately 16.8% |
| Scoring time | 28.97 seconds |

Hit Rate@10 measures the percentage of evaluated customers for whom at least one eligible held-out product appears in the top ten recommendations.

The SVD model outperformed the best-seller baseline in this experiment. This is encouraging evidence that recommendations based on purchasing similarities may be more relevant than recommending only globally popular products.

### Customer segmentation

The analysis projected customers into two SVD components and used KMeans to create four segments.

| Segment | Customers |
|---|---:|
| Segment 1 | 1,114 |
| Segment 2 | 4,005 |
| Segment 3 | 353 |
| Segment 4 | 12 |

The segments were described using their leading products and countries. The second segment is the largest, while the fourth contains only 12 customers. The small segment should be investigated further before being used to make business decisions.

These segments summarize observed purchasing patterns; they should not be interpreted as established demographic profiles.

## 3. Business Implications

The findings suggest three potential opportunities:

1. **Personalized product discovery:** Test whether recommendations based on related purchasing patterns help customers discover products beyond the best sellers.
2. **Targeted merchandising:** Explore the leading products within each customer segment when planning product displays or promotional campaigns.
3. **Further experimentation:** Evaluate whether personalized recommendations improve customer engagement and commercial outcomes compared with a popularity-based approach.

## 4. Limitations and Risks

- The evaluation uses historical transactions rather than a live customer experiment.
- A Hit Rate@10 improvement does not directly demonstrate increased revenue, conversion, or customer satisfaction.
- Products absent from the training catalogue cannot be recommended in this evaluation setup.
- The smallest customer segment may be unstable and should not be generalized without further analysis.
- Scoring time was measured in the current local environment and may differ in deployment.

## 5. Recommended Next Steps

1. Run a controlled pilot comparing SVD recommendations with the best-seller baseline.
2. Track recommendation clicks, conversion rate, revenue per session, and product coverage.
3. Review the smallest customer segment and assess whether the chosen number of clusters is appropriate.
4. Re-evaluate the model periodically as customer purchasing patterns change.

## Conclusion

The SVD model showed a promising improvement over the best-seller baseline on historical held-out purchases. The customer segmentation analysis also provides a starting point for exploring differences in purchasing behavior.

The recommended next step is a controlled pilot with clear business metrics. Commercial value should be confirmed through further evaluation before deploying the recommendations broadly.