# ZeptoFresh: 15-Minute Delivery Risk Prediction EDA Phase

### (a) Data Quality Diagnosis
Based on the EDA of 110,000 records, four critical data quality issues were identified:

1.  **Impossible Values**: 214 rows have `delivery_time_mins = 0`.
    * **Classification**: Data Entry Error.
    * **Treatment**: Remove these records as they represent physical impossibilities in a delivery context.
2.  **Extreme Outliers**: A single bakery order is valued at Rs. 2.95 Lakh.
    * **Classification**: Outlier (Anomaly).
    * **Treatment**: Cap the value at the 99th percentile (Winsorization) to prevent it from skewing model weights.
3.  **Invalid Numerical Data**: `prep_time_mins` shows a minimum value of -6.
    * **Classification**: Data Entry Error / Structural Issue.
    * **Treatment**: Replace negative values with the median preparation time for that specific `order_category`.
4.  **Missing Values**: 9,800 null values exist in `customer_rating`.
    * **Classification**: Missing Value.
    * **Treatment**: Impute with a constant (e.g., 0) or a "Not Rated" flag to preserve data volume without inventing fake satisfaction scores.

---

### (b) Distribution Analysis
* **Observation**: The Mean (18.4) is greater than the Median (14.2).
* **Shape**: This indicates a **Positively Skewed (Right-Skewed)** distribution.

* **Transformation**: Apply a **Log Transformation**.
* **Reasoning**: Log transforms compress the long right tail, reducing the impact of outliers and helping the model converge more effectively by approximating a normal distribution.

---

### (c) Correlation Interpretation
* **Logical Flaw**: The Product Manager assumes **Causation** (Late deliveries cause refunds) from **Correlation** ($r=+0.74$). 
* **Meaning of Correlation**: Correlation only measures the strength and direction of a linear relationship; it does not prove that one variable causes the other.
* **Possible Confounders**:
    1.  **Rain (rain_flag)**: Heavy rain increases delivery time and may damage food quality, causing refunds independent of time.
    2.  **Order Volume (items_count)**: Large orders take longer to prep and have a higher likelihood of missing items, triggering refunds.

---

### (d) Bimodal Pattern in Tier-1 Cities
* **Operational Reasons**: In cities like Mumbai and Bangalore, the peaks at 12-14 mins and 28-32 mins likely represent **Peak vs. Off-Peak traffic** or **Proximity vs. Remote delivery zones**.
* **Modeling Mistake**: If ignored, the model will attempt to predict a single "mean" (approx. 22 mins), which is the point of lowest frequency (the valley). This results in a model that is consistently wrong for both fast and slow deliveries.

---

### (e) Business Trade-Off
* **Prioritize**: **Recall**.
* **Justification**: VP Operations Kavya Sharma notes that 30+ min deliveries lead to significant **customer churn**. The long-term cost of losing a customer (High Recall need) is greater than the short-term operational cost of "unnecessary rider reallocations" (Low Precision risk).

---

### (f) Advanced Feature Engineering
Recommended features to improve the "late delivery risk prediction model":

1.  **Delivery_Efficiency**: `rider_distance_km / delivery_time_mins`
2.  **Rain_Delay_Index**: `rain_flag * rider_distance_km`
3.  **Preparation_Efficiency**: `prep_time_mins / items_count`
