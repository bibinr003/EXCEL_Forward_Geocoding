# EXCEL_Forward_Geocoding

Here's a systematic approach for geocoding

```markdown
# Excel Geocoder
A Python script for geocoding addresses from Excel files using the Geoapify API.

## Features
- Process addresses from Excel files
- Handle rate limiting automatically
- Support for large datasets
- Error handling and logging
- NDJSON output format

## Requirements
- Python 3.7+
- pandas
- requests
- Geoapify API key

## Usage
```bash
python excel_geocoder.py --file your_addresses.xlsx --column "Address" --output results.ndjson
```

## Installation
```bash
pip install -r requirements.txt
```

## Example
```python
from excel_geocoder import ExcelGeoCoder

# Initialize with your API key
geocoder = ExcelGeoCoder(api_key="YOUR_API_KEY")

# Read and process addresses
addresses = geocoder.read_excel("addresses.xlsx", "Address")
results = geocoder.batch_geocode(addresses)
geocoder.save_results(results, "output.ndjson")
```
```
