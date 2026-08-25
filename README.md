# Flight_project_deploye
I built an incremental data ingestion pipeline for airline data using AWS services.

The main goal of the project was to process new daily flight data without processing the old data again. I used Amazon S3 to store the airport reference data and daily flight files. When a new CSV file was uploaded to S3, Amazon EventBridge detected the file upload and triggered the Step Functions workflow. The EventBridge rule was configured to listen for S3 Object Created events for CSV files.

Step Functions was used to control the complete workflow. First, it started the AWS Glue Crawler and checked its status until the crawler finished. After that, it started the Glue ETL job.

In the Glue job, I read the flight data and airport reference data from the Glue Data Catalog. I joined the flight data with airport data to get departure and arrival airport details such as city, state, and airport name. I also applied the required column mappings before loading the processed data into Amazon Redshift.

The final processed data was stored in Redshift in a structured table containing carrier, departure and arrival details, cities, states, and flight delays.

### 2. Pipeline Flow

The pipeline works like this:

**S3 → EventBridge → Step Functions → Glue Crawler → Glue Data Catalog → Glue ETL → Redshift**

![Architecture Diagram]("https://github.com/Rudraprasad112/Flight_project_deploye/blob/main/Architecture_diagram.png")


At the end of the workflow, SNS sends a notification about whether the Glue job was successful or failed. The Step Functions configuration contains separate success and failure notification states.

### 3. Problems I Faced

**1. Duplicate processing**

One problem was that the same data could be processed again when files were uploaded or processed more than once. This can increase processing time and create duplicate records.

**2. Processing only new data**

The main challenge was to make the pipeline incremental instead of processing the complete dataset every time. Processing all the old flight data again would not be efficient.

**3. Pipeline failures**

Another problem was that different steps of the pipeline could fail. For example, the crawler or Glue ETL job could fail, and it was important to know when something went wrong.

**4. Data transformation**

The raw flight data only had airport IDs. I needed to join it with the airport reference data to get useful information such as departure airport, arrival airport, city, and state. My Glue job performs these joins and mappings before loading the data into Redshift.

### 4. Solutions

For incremental processing, I used **AWS Glue Job Bookmarking** so that the ETL process could keep track of previously processed data and focus on new data.

I used **Step Functions** to control the order of the pipeline. It first runs the crawler, checks whether it is still running, and then starts the Glue ETL job after the crawler is completed.

For failures, I added a failure path in Step Functions. If the Glue job fails, the workflow goes to the failure notification step. If it completes successfully, it sends a success notification through SNS.

For data transformation, I joined the flight data with the airport dimension data using the airport IDs. Then I renamed and mapped the columns before writing the final data to Redshift.

### 5. What I Learned

Through this project, I learned how different AWS services can work together to build a complete data pipeline. I got a better understanding of incremental data processing, ETL, data cataloging, workflow orchestration, data transformation, and loading data into a data warehouse.

The project also helped me understand why monitoring and failure notifications are important in a data pipeline.

