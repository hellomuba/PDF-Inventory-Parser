# PDF Inventory Parser (Jupyter Notebook)

A Jupyter notebook-based tool for extracting structured inventory data from PDF documents.

## Project Overview

This project uses Jupyter notebooks to parse PDF files containing inventory information and convert them into structured JSON data. It extracts:
- Owner information (name, address, telephone)
- Inventory items with their descriptions, purchase dates, serial numbers, and values

## Requirements

- Python 3.8+
- Jupyter Notebook/Lab
- Required libraries:
  - PyPDF2 (or similar PDF extraction library)
  - datetime
  - re
  - json

## Installation

```bash
git clone https://github.com/yourusername/pdf-inventory-parser.git
cd pdf-inventory-parser
pip install -r requirements.txt
jupyter notebook
```

## Usage

1. Open `inventory_parser.ipynb` in Jupyter
2. Update the `pdf_path` variable with the path to your PDF file
3. Run all cells in sequence
4. The structured data will be saved as `extracted_inventory.json`

## Project Files

- `inventory_parser.ipynb` - Main notebook containing all extraction and parsing code
- `requirements.txt` - Required Python dependencies
- `sample/` - Sample PDF files for testing
- `output/` - Directory where JSON output is saved

## Notebook Structure

The notebook is divided into these main sections:
1. **Setup** - Imports and initialization
2. **Data Models** - Classes for Owner and Inventory data
3. **PDF Processing** - Functions to extract and clean text from PDFs
4. **Date Parsing** - Functions to handle different date formats
5. **Data Extraction** - Functions to identify and extract structured data
6. **Run Pipeline** - End-to-end process execution
7. **Output Results** - Data visualization and JSON export

## Output Format

```json
{
  "owner_name": "Owner Name",
  "owner_address": "Owner Address",
  "owner_telephone": "Owner Phone Number",
  "data": [
    {
      "purchase_date": "2025-04-25T20:50:51",
      "serial_number": "12345",
      "description": "Item Description",
      "source_style_area": "Source Style Area",
      "value": "500"
    }
  ]
}
```

## License

MIT
