# Simplifying Data Analytics: From CSV to Insights with AWS S3, Glue, and Athena


![s3-glue-athena](https://github.com/user-attachments/assets/efd6502d-9224-4a41-9816-c68afe65dd74)

The ability to quickly analyze data stored in CSV files is a common requirement. Here's how to create a streamlined workflow using AWS services to transform raw CSV data into queryable insights.

## Starting Point: Our Customer Data

We'll work with a customer dataset containing basic information like card IDs, personal details, and location information. The CSV file contains these columns: card_id, customer_id, lastname, firstname, email, address, birthday, and country.

## Step 1: Upload to Amazon S3

First, create an S3 bucket for your raw data:
1. Navigate to the S3 console
2. Create a new bucket (e.g., `my-customer-data-bucket`)
3. Upload your customers.csv file to a folder like `raw-data/customers/customers.csv`

## Step 2: Create and Run a Glue Crawler

The Glue Crawler automatically infers the schema from your CSV file:

1. Go to AWS Glue Console
2. Create a new crawler:
   - Name it `customer-data-crawler`
   - Select your S3 path (`s3://my-customer-data-bucket/raw-data/customers/`)
   - Create a new database (e.g., `customer_db`)
   - Schedule it to run on demand
3. Run the crawler

The crawler creates a table in your Glue Data Catalog, making the data queryable through Athena.

## Step 3: Configure Athena

Before running any queries, you must configure an S3 location for query results:

1. Open Athena console
2. Click on Settings
3. Set "Query result location" to an S3 bucket/path like:
   `s3://my-customer-data-bucket/athena-results/`

This step is crucial - Athena won't run queries without a configured results location.

## Step 4: Query Your Data

Now you can query your customer data using standard SQL. Here's a simple query to verify everything is working:

```sql
SELECT *
FROM customer_db.customers
LIMIT 5;
```

This query returns the first 5 rows from your customer data, allowing you to verify the schema and data quality.

## Benefits of This Workflow

- Zero infrastructure management
- Pay only for the queries you run
- Automatic schema inference
- SQL interface for CSV data
- Scalable to handle growing datasets

This workflow creates a foundation for more complex analytics. You can build on it by adding more data sources, creating views, or implementing more complex queries to derive business insights.

Remember to implement appropriate S3 bucket policies and IAM roles to secure your data, and consider using Glue jobs for any necessary data transformations before querying.
