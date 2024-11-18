
# Data Ingestion

## Purpose
This document provides an overview of a data ingestion process that enables seamless data transfer between different storage systems (Azure, AWS, SFTP, and Databases) using a JSON-based configuration file. The ingestion framework dynamically reads the JSON file to process data from defined sources to destinations with minimal manual effort.

## Supported Sources
- **Azure Blob Storage**
- **AWS S3**
- **Relational Databases** (e.g., MySQL, Postgresql, SSMS)
- **SFTP Servers**

## Supported Destinations
- **Azure Blob Storage**
- **AWS S3**

## How It Works
The ingestion process involves defining a JSON configuration file to specify the source and destination parameters. The ingestion framework reads this JSON file and automates the transfer, ensuring data is processed and stored in the desired location and format.

## Key Features
- Flexibility to specify multiple tables in relational database sources.
- Automatic handling of file formats with Delta as the default for the destination.
- Dynamic path assignment to destination for each table in the source.

## Mandatory Keys in JSON

### Source Keys
#### For Azure Blob, AWS S3, and SFTP:
- **type**: The type of source storage (e.g., azure_blob, aws_s3, sftp).
- **path**: Path to the source data.
- **format**: File format of the source data (csv, parquet, delta, etc.).
- **options**: Additional settings for connection or processing (e.g., credentials, header inclusion).

#### For Databases:
- **type**: Set to `database`.
- **url**: JDBC URL of the database.
- **tables**: List of table names to ingest.
- **properties**: Connection properties, including:
  - **user**: Database username.
  - **password**: Database password.
  - **driver**: JDBC driver class (e.g., com.mysql.cj.jdbc.Driver).

### Destination Keys
#### For Azure Blob and AWS S3:
- **type**: The type of destination storage (e.g., azure_blob, aws_s3).
- **path**: List of paths to store the data. Each source table requires a corresponding destination path.
- **format**: File format for the destination data. Default is delta.
- **options**: Additional settings for writing data (e.g., mode for overwrite/append).

## JSON Configuration File Example

### Azure Blob to AWS S3
Transfer data in Delta format from Azure Blob Storage to AWS S3 with a Parquet output.

```json
{
  "source": {
    "type": "azure_blob",
    "path": ["/mnt/liquormarts/liquormart/gold/look_up"],
    "format": "delta",
    "options": {
      "header": "true"
    }
  },
  "destination": {
    "type": "aws_s3",
    "path": ["/mnt/s3-bucket/Azure"],
    "format": "parquet",
    "options": {
      "mode": "overwrite"
    }
  }
}
```

### Database to Azure Blob
Extract data from two tables in a MySQL database and store them in Parquet format on Azure Blob Storage. For each table in the source, a corresponding path must be specified in the destination.

```json
{
  "source": {
    "type": "database",
    "url": "jdbc:mysql://<host>:3306/CompanyDB",
    "tables": ["Employees", "Departments"],
    "properties": {
      "user": "admin",
      "password": "your_password",
      "driver": "com.mysql.cj.jdbc.Driver"
    }
  },
  "destination": {
    "type": "azure_blob",
    "path": [
      "/mnt/liquormarts/liquormart/DB/Emp",
      "/mnt/liquormarts/liquormart/DB/Dep"
    ],
    "format": "parquet",
    "options": {
      "header": "true",
      "mode": "overwrite"
    }
  }
}
```

### SFTP to AWS S3
Fetches a CSV file from an SFTP server and uploads it to AWS S3 in Delta format.

```json
{
  "source": {
    "type": "sftp",
    "path": ["/SFTP/liquormart.csv"],
    "format": "csv",
    "options": {
      "host": "us-east-1.sftpcloud.io",
      "port": 22,
      "username": "user_name",
      "password": "pass"
    }
  },
  "destination": {
    "type": "aws_s3",
    "path": ["/mnt/s3-bucket/SFTP"],
    "format": "delta",
    "options": {
      "mode": "overwrite"
    }
  }
}
```

## How to Use

### 1. Prepare the JSON File:
- Create a JSON file as per the structure described above.
- Define:
  - The source type, paths, and format.
  - The destination type, paths, and format.
- For multiple database tables, ensure each table has a corresponding destination path.

### 2. Place the JSON File in Databricks:
- Save the JSON file to the Databricks workspace or DBFS.

### 3. Install Prerequisites:
- Ensure the following libraries are installed in the Databricks cluster:
  - **MySQL JDBC Driver**: Add “com.mysql.cj.jdbc.Driver”.
  - **Paramiko for SFTP**: Add “paramiko==2.11.0”.

### 4. Run the `ingest_data` Function:

- Import the `ingest_data` function from `ingestion.py`.
- Call the function with the JSON file:

```python
from ingestion import ingest_data

# Load the JSON configuration
config_path = "/dbfs/<path-to-json-file>.json"

# Execute the ingestion process
ingest_data(config_path)
```

### 5. Validate Output:
- Check the logs in Databricks for any errors or warnings.
- Verify the data in the destination location.

## Key Points
- For database sources, ensure each table in the `tables` section has a corresponding path in the destination's `path` list.
- The default file format for the destination is Delta. You can override it by specifying another format.
- The `options` section must include required settings such as connection details or write modes.
