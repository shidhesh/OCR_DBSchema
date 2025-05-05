# OCR_DBSchema
# PDF OCR to PostgreSQL Pipeline

A comprehensive solution for extracting structured data from PDF documents using OCR and storing it in a PostgreSQL database according to a predefined schema.

## Overview

This project consists of two main components:

1. **OCR Extraction Module**: Converts PDF documents into machine-readable text using OpenAI's Vision API, preserving document structure and table layouts.
2. **Database Integration Module**: Transforms the extracted OCR text into structured JSON data based on a PostgreSQL schema and inserts it into a database.

## Features

- High-quality OCR extraction with table structure preservation
- Parallel processing of PDF pages for improved performance
- Intelligent data extraction and JSON generation using GPT-4.1
- PostgreSQL database integration with proper schema validation
- Comprehensive error handling and logging

## Requirements

- Python 3.8+
- OpenAI API key
- PostgreSQL database
- Required Python packages (see Installation section)

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/pdf-ocr-to-postgres.git
   cd pdf-ocr-to-postgres
   ```

2. Install required packages:
   ```
   pip install -r requirements.txt
   ```

3. Create a `.env` file in the project root with the following variables:
   ```
   OPENAI_API_KEY=your_openai_api_key
   DB_HOST=your_db_host
   DB_USER=your_db_username
   DB_PASSWORD=your_db_password
   DB_NAME=your_db_name
   DB_PORT=your_db_port
   ```

## Database Schema

The project uses the following PostgreSQL schema:

```sql
CREATE TABLE batches (
    batch_id      text PRIMARY KEY,
    doc_no        text,
    title         text,
    effective_dt  date
);

CREATE TABLE steps (
    step_id       bigserial PRIMARY KEY,
    batch_id      text REFERENCES batches,
    step_no       text,
    step_title    text
);

CREATE TABLE facts (
    fact_id       bigserial PRIMARY KEY,
    step_id       bigint REFERENCES steps,
    category      text,
    kv            jsonb
);

CREATE TABLE materials (
    mat_id        bigserial PRIMARY KEY,
    step_id       bigint REFERENCES steps,
    kv            jsonb
);

CREATE TABLE signoffs (
    sign_id       bigserial PRIMARY KEY,
    step_id       bigint REFERENCES steps,
    role          text,
    initials      text,
    signed_ts     timestamp
);
```

## Usage

### OCR Module

```python
from ocr_module import process_pdf

# Extract text from PDF
output_file = process_pdf("path/to/your/document.pdf")
print(f"OCR extraction completed. Results saved to {output_file}")
```

### Database Integration Module

```python
from db_module import generate_json_from_data, insert_into_database

# Define your schema
schema_definition = """
CREATE TABLE batches (
    batch_id      text PRIMARY KEY,
    ...
);
...
"""

# Read OCR data
with open("ocr_output.txt", "r", encoding="utf-8") as f:
    ocr_data = f.read()

# Process data and insert into database
json_data = generate_json_from_data(ocr_data, schema_definition)
insert_into_database(json_data)
```

## How It Works

1. **PDF Processing**:
   - PDF pages are extracted as high-resolution images
   - Each page is processed in parallel for efficiency
   - Images are sent to OpenAI's Vision API for OCR extraction
   - OCR results are saved as individual text files and combined

2. **Data Extraction and Transformation**:
   - OCR text is analyzed by GPT-4.1 to identify structured data
   - Data is transformed into a JSON format matching the database schema
   - JSON includes batches, steps, facts, materials, and signoffs

3. **Database Integration**:
   - JSON data is validated against the schema
   - Data is inserted into the PostgreSQL database
   - Foreign key relationships are preserved
   - Duplicate entries are handled appropriately

## Configuration Options

### OCR Module

- `MODEL`: OpenAI model to use for vision processing (default: "gpt-4.1")
- `dpi`: Resolution for PDF page extraction (default: 300)

### Database Module

- `MODEL`: OpenAI model for JSON generation (default: "gpt-4.1")
- Database connection parameters (stored in `.env` file)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Author

Sidheshwar Shelke

---
