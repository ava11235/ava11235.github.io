# 🛠️ Feature Engineering with Amazon SageMaker Data Wrangler 

## Summary
Feature engineering is critical for machine learning success, yet it traditionally consumes 60-70% of data scientists' time. Amazon SageMaker Data Wrangler transforms this process through its visual interface, allowing you to create features without writing code. This tutorial demonstrates how Data Wrangler streamlines feature creation, from basic transformations to complex derived features that improve model performance.

### 1. Setup and Dataset Creation 📁

First, let's create a dummy e-commerce dataset:

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# Set seed for reproducibility ✨
np.random.seed(42)

# Generate 1000 sample records 📝
n_samples = 1000

# Create base data 🗄️
data = {
    'customer_id': np.arange(1, n_samples + 1),
    'age': np.random.randint(18, 75, n_samples),
    'gender': np.random.choice(['M', 'F', 'Other'], n_samples),
    'location': np.random.choice(['Urban', 'Suburban', 'Rural'], n_samples),
    'membership_days': np.random.randint(1, 1000, n_samples),
    'previous_purchases': np.random.randint(0, 50, n_samples),
    'last_purchase_amount': np.random.uniform(10, 500, n_samples).round(2),
    'items_viewed': np.random.randint(0, 100, n_samples),
    'cart_abandonment_rate': np.random.uniform(0, 1, n_samples).round(2),
    'purchase_made': np.random.choice([0, 1], n_samples, p=[0.7, 0.3])
}

# Add some missing values 
for col in ['age', 'last_purchase_amount', 'items_viewed']:
    mask = np.random.choice([True, False], n_samples, p=[0.05, 0.95])
    data[col] = np.where(mask, np.nan, data[col])

# Create DataFrame 🔢
ecommerce_df = pd.DataFrame(data)

# Save to CSV 💾
ecommerce_df.to_csv('ecommerce_data.csv', index=False)
print("Dataset created and saved as 'ecommerce_data.csv'")
```

Upload this CSV file to an S3 bucket for use with Data Wrangler: 🪣

```python
import boto3
import sagemaker

session = boto3.Session()
region = session.region_name
sagemaker_session = sagemaker.Session()
default_bucket = sagemaker_session.default_bucket()
prefix = 'data-wrangler-tutorial'

s3_client = boto3.client('s3')
s3_client.upload_file(
    'ecommerce_data.csv', 
    default_bucket, 
    f'{prefix}/ecommerce_data.csv'
)

print(f"File uploaded to s3://{default_bucket}/{prefix}/ecommerce_data.csv")
```

![image](https://github.com/user-attachments/assets/665d57b7-ca36-41a5-bd5e-0517438fbf22)

### 2. Launch SageMaker Data Wrangler 🚀

1. Open Amazon SageMaker Studio 🖥️
2. From the Studio launcher, click on "New data flow" or navigate to File > New > Flow [[3]](https://aws.amazon.com/sagemaker/data-wrangler/)

### 3. Import the Dataset 📥

1. In the Data Wrangler interface, select "Import data" 📤
2. Choose "Amazon S3" as the data source 🪣
3. Navigate to your bucket and select the ecommerce_data.csv file 📄
4. Click "Import" to load the dataset into Data Wrangler ✅

### 4. Data Exploration and Analysis 🔍

1. In the Data Flow view, click on the dataset node 📊
2. Select "Data types" to verify column types are correctly assigned ✓
3. Use "Data quality and insights" to generate a report on your data: 📈
   * Check for missing values ❓
   * View distribution of features 📊
   * Identify correlations between features [[4]](https://aws.amazon.com/blogs/machine-learning/sagemaker-data-wrangler-now-auto-generates-feature-level-visualizations/)

### 5. Feature Engineering Steps 🛠️

#### Step 1: Handle Missing Values ❓➡️✅

1. Click "+ Add step" on your data node ➕
2. Select "Handle missing values" 🔧
3. Choose "Impute" for:
   * age: Use "Mean" imputation 📊
   * last_purchase_amount: Use "Median" imputation 💰
   * items_viewed: Use "Custom" with value 0 👁️

#### Step 2: Create Customer Segments 👥

1. Click "+ Add step" again ➕
2. Select "Custom formula" ✏️
3. Enter formula to create spending segments:

   ```
   case(
     last_purchase_amount > 300, 'High Spender',
     last_purchase_amount > 100, 'Medium Spender',
     'Low Spender'
   )
   ```


5. Name the new column spending_segment 💸

#### Step 3: One-Hot Encode Categorical Variables 🔄

1. Add another step ➕
2. Select "Encode categorical" 🏷️
3. Choose "One-hot encode" 0️⃣1️⃣
4. Select columns: gender, location, and spending_segment 📋

#### Step 4: Create Interaction Features ⚡

1. Add another step ➕
2. Select "Custom formula" ✏️
3. Create an engagement score: `(previous_purchases * 0.4) + (items_viewed * 0.3) + ((1 - cart_abandonment_rate) * 0.3)` 🧮
4. Name the new column engagement_score 📈

### 6. Analyze Model Quality 📊

1. Select "Analysis" on your final transformation node 🔍
2. Choose "Quick model" analysis 🏃‍♂️
3. Set purchase_made as the target column 🎯
4. Select "Classification" as the model type 🌲
5. Review feature importance and model performance metrics [[5]](https://aws.amazon.com/blogs/machine-learning/prepare-and-visualize-time-series-datasets-in-amazon-sagemaker-data-wrangler/)

### 7. Export the Transformed Data 📤

1. Click "+ Add destination" on your final node ➡️
2. Choose one of the following options:
   * Export to S3 🪣
   * Export to SageMaker Feature Store 🏪
   * Export to SageMaker Pipelines 🔄
   * Export to SageMaker Autopilot 🤖 [[6]](https://aws.amazon.com/sagemaker/data-wrangler/)
3. For this tutorial, select "Amazon S3" 🪣
4. Specify your S3 location and file format (e.g., CSV or Parquet) 📄
5. Click "Export" to start the processing job 🚀

### 8. Clean Up 🧹

To avoid incurring unnecessary charges, make sure to:

1. Shut down the SageMaker Data Wrangler application 🛑
2. Delete any resources created during this tutorial [[7]](https://aws.amazon.com/sagemaker/data-wrangler/)

## Key Benefits Demonstrated ✨

* 🧙‍♂️ No-code data preparation: Transformed raw data into ML-ready features without writing code
* 👁️ Visual data exploration: Quickly understood data quality issues and distributions
* ⚡ Efficient feature engineering: Created new features and transformed existing ones in minutes
* 🔍 Model validation: Used Quick Model to validate feature importance before model building
* 🔄 Streamlined workflow: Prepared data for ML model training in a single interface [[8]](https://aws.amazon.com/sagemaker/data-wrangler/)

