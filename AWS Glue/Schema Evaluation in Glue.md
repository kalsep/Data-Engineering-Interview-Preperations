# Handling Schema Changes in AWS Glue: A Guide to Building Robust ETL Pipelines


When you’re working with data at scale, it’s inevitable that the structure of your data will evolve over time. Whether it’s new fields being added, existing fields changing types, or entire schemas shifting, schema changes are a common challenge in data engineering.

AWS Glue, Amazon’s managed ETL (Extract, Transform, Load) service, provides several tools and strategies for handling schema evolution, making it easier to adapt to these changes without disrupting your data pipelines. In this post, we’ll walk you through how to handle schema changes in AWS Glue, covering automatic schema detection, Glue Crawlers, DynamicFrames, and best practices for ensuring your ETL jobs run smoothly even when the data schema changes.

1. DynamicFrames: The Glue Abstraction for Schema Flexibility
In AWS Glue, DynamicFrames are a powerful abstraction that simplifies working with semi-structured data. Unlike traditional Spark DataFrames, which require a fixed schema, DynamicFrames allow you to process data without needing to define a schema upfront. This is particularly useful when dealing with data that evolves over time.

## Automatic Schema Evolution with DynamicFrames

One of the standout features of DynamicFrames is their ability to handle schema evolution automatically. When you read data from sources like Amazon S3 or a database, Glue will infer the schema the first time it reads the data. In subsequent reads, Glue can adapt to changes in the schema — whether that’s new columns being added or existing columns being dropped.

For example, if you’re reading JSON files and a new field appears in the dataset, Glue can detect this and incorporate the new field into the processing pipeline without manual intervention.

Resolving Schema Conflicts

In some cases, schema changes can lead to conflicts. Perhaps a new column has been added, or a column’s data type has changed, which could break the existing ETL logic. To manage these conflicts, AWS Glue provides the resolveChoice()method, which allows you to specify how to handle mismatches in column names or types.

Here’s an example of how you can use resolveChoice() in a Glue job to handle schema changes:

from awsglue.dynamicframe import DynamicFrame
# Create a DynamicFrame from a DataFrame
dynamic_frame = DynamicFrame.fromDF(spark_df, glueContext, "dynamic_frame")
# Resolve conflicts (e.g., cast a column to integer)
dynamic_frame_resolved = dynamic_frame.resolveChoice(specs=[('column_name', 'cast:integer')])
# Continue with further processing...
This flexibility allows your ETL jobs to process evolving datasets without breaking.

2. Glue Catalog and Schema Evolution
The AWS Glue Data Catalog acts as a central repository for metadata. When the structure of your data changes (such as new fields appearing or field types changing), the Glue Catalog can be updated to reflect the new schema.

Automatic Schema Detection with Glue Crawlers

AWS Glue Crawlers can automatically detect schema changes when they scan your data sources (e.g., Amazon S3 buckets or relational databases). By running crawlers periodically, you can ensure that your Glue Data Catalog is up to date with the latest schema.

When schema changes are detected, the crawler can update the table definition in the Glue Data Catalog. For instance, if a new column is added to the dataset, the crawler will update the metadata to include this new field. You can configure crawlers to “update the table definition” when schema changes occur.

To enable this, simply create or configure a Glue Crawler and schedule it to run on your data source. The crawler will then ensure that the Glue Catalog is updated with any changes to the schema.

Get Devendra’s stories in your inbox
Join Medium for free to get updates from this writer.

Enter your email
Subscribe

Remember me for faster sign in

How to Configure Crawlers for Schema Detection:

Create a Crawler: Define the data source (like S3 or a database) and the target Glue database.
Schedule the Crawler: Set the crawler to run periodically (daily, weekly, etc.) to scan for schema changes.
Update Tables Automatically: Configure the crawler to “update the table definition” when schema changes are detected.
Manually Managing Schema Changes

In some cases, you may want more control over schema changes. You can manually update the schema in the Glue Data Catalog using the AWS Glue Console, AWS CLI, or AWS SDKs. For instance, if a new column needs to be added to an existing table, you can manually update the table’s schema in the Glue Data Catalog.

Here’s an example of how you can manually update the schema using the Glue API:

import boto3
# Create a Glue client
glue = boto3.client('glue')
# Define the updated schema (adding a new column)
updated_columns = [
    {'Name': 'existing_column1', 'Type': 'string'},
    {'Name': 'existing_column2', 'Type': 'int'},
    {'Name': 'new_column', 'Type': 'string'}  # New column added
]
# Update the table schema in the Glue Catalog
response = glue.update_table(
    DatabaseName='my_database',
    TableInput={
        'Name': 'my_table',
        'StorageDescriptor': {
            'Columns': updated_columns
        }
    }
)
print(response)
This manual approach gives you the flexibility to control the schema update process.

3. Glue ETL Jobs and Schema Changes
When building ETL jobs in AWS Glue, schema changes are often inevitable. To handle these changes in your job scripts, you need to account for potential issues such as missing fields, mismatched column types, or unexpected null values.

Adapting to Schema Changes in ETL Jobs

Let’s say you’re working with a dataset that has been evolving over time, and a new column has been added. In your ETL job, you can adapt to these changes by using Glue’s DynamicFrame methods.

For example, if a column is missing in a new batch of data, you can drop null values to ensure that the ETL process continues smoothly:

# Drop null fields if schema changes have introduced missing columns
dynamic_frame_clean = dynamic_frame.dropNullFields()
# Continue with further transformations
data_frame = dynamic_frame_clean.toDF()
# Write to a destination (e.g., S3, Redshift)
This ensures that your job can handle variations in the schema without causing errors or inconsistent results.

4. Best Practices for Handling Schema Evolution in AWS Glue
While AWS Glue offers powerful tools to manage schema evolution, it’s important to follow some best practices to ensure your data pipelines remain robust:

Version Control: Keep track of schema changes by versioning your data or using Glue’s built-in versioning for tables in the Glue Data Catalog. This helps maintain a history of schema changes for auditing and debugging purposes.
Testing: Always test your ETL jobs with different schema versions to ensure they can handle changes without failing. You can do this by running jobs on smaller test datasets with different schema variations.
Error Handling: Implement error handling in your ETL scripts to catch issues that arise from unexpected schema changes, such as missing or extra fields.
Frequent Crawler Runs: If you’re relying on Glue Crawlers for automatic schema updates, schedule them to run frequently to catch schema changes early.