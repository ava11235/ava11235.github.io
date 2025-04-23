# Bias Detection with Amazon SageMaker Clarify 

![image-1745435703244](https://github.com/user-attachments/assets/6974d068-f5ae-448d-9dae-16ac863f0309)

Let's explore bias detection using a banking marketing dataset 📊

## What is SageMaker Clarify? 
Amazon SageMaker Clarify is a powerful tool that helps:

- Detect bias in data and models
- Explain model predictions
- Promote transparency in machine learning

### Key Capabilities:

Bias Detection:

- Pre-training data bias analysis
- Post-training model bias evaluation
- Multiple bias metrics calculation
- Model Explainability:

- Feature importance
- SHAP values for explainability
- Global and local explanations

Integration:

- Seamless integration with SageMaker
- Works with various data types
- Supports multiple model types

When to Use Clarify:

- Before training: Detect data bias
- After training: Evaluate model fairness
- During deployment: Monitor for bias
- For compliance: Document fairness metrics

### Example usage of SageMaker Clarify for data bias detection 🎯
We'll analyze a banking marketing campaign dataset stored in the /tmp/ directory (remember, this is temporary storage just for our analysis session!):

```python
# Download and prepare data
!curl -o bank-additional.zip https://sagemaker-sample-data-us-west-2.s3-us-west-2.amazonaws.com/autopilot/direct_marketing/bank-additional.zip
!unzip -o bank-additional.zip -d /tmp/
local_data_path = '/tmp/bank-additional/bank-additional-full.csv'
df = pd.read_csv(local_data_path)
```
```python
df.columns
```
![image](https://github.com/user-attachments/assets/991acb97-6646-4a11-a8db-d4febe90b634)

```python
df.head()
```
![image](https://github.com/user-attachments/assets/0a9dc7ab-2f6c-4991-9b32-01103d12d174)



📊 Initial Data Exploration
Let's visualize our data relationships:
```python
# Create pairplot for key variables
sns.pairplot(df[['age','campaign', 'pdays']])
```
![image](https://github.com/user-attachments/assets/e6e5eb20-3e10-4da4-8268-3c8731559682)

Remember: 
- Diagonal shows distributions
- Off-diagonal shows relationships between variables
- 'pdays' represents days since last contact (999 = never contacted)

📈 Campaign Success Visualization
```python
# Visualize campaign outcomes
sns.countplot(data=df, x='y')
```
![image](https://github.com/user-attachments/assets/019a97ef-b0e1-43de-b503-46589a193f6f)

🔍 Detailed Bias Analysis

Let's set up our bias detection:
```python
# Create age buckets
df['age_disc'] = pd.cut(df.age, bins=3, labels=['young', 'middle', 'old'])
facet_column = report.FacetColumn('age_disc')
label_column = report.LabelColumn(name='y', series=df['y'], positive_label_values=['yes'])

# Run bias report with education as confounding variable
report.bias_report(df, facet_column, label_column, 
                  stage_type=report.StageType.PRE_TRAINING, 
                  group_variable=df['education'])
```
🔍 Notes: While we used other tools for visualization (seaborn, matplotlib), clarify is used for the report (from smclarify.bias import report)

![image](https://github.com/user-attachments/assets/aacbf3e0-4cad-4784-8d3f-53ad9e12a075)

🎯 Key Bias Metrics Explained:
1. CDDL (Conditional Demographic Disparity in Labels)
   - Middle age: 0.007 (minimal bias)
   - Young: 0.028 (slight bias)
   - Old: -0.035 (significant disadvantage)

2. CI (Class Imbalance)
   - Middle: 0.390
   - Young: -0.372 (underrepresented)
   - Old: 0.982 (severe imbalance!)

3. DPL (Difference in Positive Proportions)
   - Middle: 0.005
   - Young: 0.010
   - Old: -0.381 (much lower success rate)

💡  AWS ML Engineer exam tip: Know Your Metrics

CDDL (Conditional Demographic Disparity in Labels)
- Measures bias while accounting for confounding variables (like education)
- Shows if disparities persist after controlling for other factors
- Range: -1 to +1, where 0 indicates no conditional bias
- Positive values indicate favorable bias towards the facet group
- Negative values indicate unfavorable bias against the facet group

CI (Class Imbalance)
-  representation imbalance between groups
- Range: -1 to +1 (0 = perfect balance)
- Positive values indicate overrepresentation, negative values indicate underrepresentation

DPL (Difference in Positive Proportions in Labels)
- Compares the rate of positive outcomes between groups
- Shows if certain groups are more/less likely to get positive outcomes

JS (Jensen-Shannon Divergence)
- Measures similarity between probability distributions
- Range: 0 to 1 (0 = identical distributions)

KL (Kullback-Leibler Divergence)
- Measures how one probability distribution differs from another
- Larger values indicate greater differences
- KS (Kolmogorov-Smirnov Distance)

Maximum difference between cumulative distributions
- Range: 0 to 1 (0 = no difference)

TVD (Total Variation Distance)

- Measures maximum difference in probabilities between groups
- Range: 0 to 1 (0 = identical distributions)

LP (L-p Norm)
- Measures the magnitude of differences between distributions
- Larger values indicate greater disparity

📊 Campaign Contact Analysis
```python
# Analyze contact patterns
age_group_stats = df.groupby('age_group').agg({
    'campaign': ['count', 'mean', 'sum']
}).round(2)
```
![image](https://github.com/user-attachments/assets/59c38a2c-6ef1-4162-b834-f373600a050a)

Key findings:
- Ages 50-60: highest average contacts (2.69)
- Ages 70-80: lowest average contacts (1.94)
- Clear age-based contact strategy differences

🛠️ Mitigation Strategies:
1. Data Collection & Sampling:
   - Balance age group representation
   - Implement stratified sampling
   - Increase older age group data

2. Marketing Campaign Adjustments:
   - Age-appropriate strategies
   - Standardize contact frequencies
   - Monitor contact rates

3. Process Modifications:
   - Regular bias monitoring
   - Clear decision criteria
   - Document justifications

4. Training & Awareness:
   - Staff training on age bias
   - Policy development
   - Regular reviews



2. Understanding Confounding Variables:
   - Why we used education as a group variable
   - How it helps isolate true age bias
   - Impact on bias interpretation

3. Practical Application:
   - How to use SageMaker Clarify
   - Interpreting bias reports
   - Implementing mitigation strategies

🎓 Why This Matters:
The AWS ML Engineer exam specifically tests:
- Bias identification using AWS tools
- Pre-training metric understanding
- Mitigation strategy knowledge
- Real-world application

Remember: Understanding bias isn't just about passing the exam - it's about building fair, ethical ML solutions that work for everyone! 🌟

🔬 Try It Yourself! Want to get hands-on experience with this example? Check out this SageMaker Clarify notebook:
[Bias_metrics_usage_marketing.ipynb](https://github.com/aws/amazon-sagemaker-clarify/blob/master/examples/Bias_metrics_usage_marketing.ipynb)

