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
import pandas as pd
import requests
import json
from itertools import batched
from concurrent.futures import ThreadPoolExecutor, wait, ALL_COMPLETED
from time import sleep

class ExcelGeoCoder:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.geoapify.com/v1/geocode/search"
        self.REQUESTS_PER_SECOND = 5  # Geoapify's rate limit
        
    def read_excel(self, file_path, address_column):
        """Read Excel file and extract address column"""
        try:
            df = pd.read_excel(file_path)
            if address_column not in df.columns:
                raise ValueError(f"Column '{address_column}' not found in Excel file")
            return df[address_column].dropna().tolist()
        except Exception as e:
            raise Exception(f"Error reading Excel file: {str(e)}")
    
    def geocode_address(self, address):
        """Geocode a single address"""
        params = {
            'format': 'json',
            'text': address,
            'limit': 1,
            'apiKey': self.api_key
        }
        
        try:
            response = requests.get(self.base_url, params=params)
            if response.status_code == 200:
                data = response.json()
                if len(data['results']) > 0:
                    return data['results'][0]
            return {"error": "Not found"}
        except Exception as e:
            return {"error": str(e)}
    
    def batch_geocode(self, addresses):
        """Process addresses in batches respecting rate limits"""
        results = []
        batches = list(batched(addresses, self.REQUESTS_PER_SECOND))
        
        for batch in batches:
            tasks = []
            with ThreadPoolExecutor(max_workers=10) as executor:
                tasks.extend([
                    executor.submit(self.geocode_address, address)
                    for address in batch
                ])
                sleep(1)  # Respect rate limit
                wait(tasks, return_when=ALL_COMPLETED)
                results.extend([task.result() for task in tasks])
        
        return results
    
    def save_results(self, results, output_file):
        """Save results to JSON file"""
        with open(output_file, 'w') as f:
            for result in results:
                f.write(json.dumps(result) + '\n')

def main():
    # Configuration
    api_key = "YOUR_GEOAPIFY_API_KEY"  # Replace with your Geoapify API key
    excel_file = "addresses.xlsx"      # Your Excel file
    address_column = "Address"         # Your address column name
    output_file = "geocoded_results.ndjson"
    
    # Initialize geocoder
    geocoder = ExcelGeoCoder(api_key)
    
    try:
        # Read addresses from Excel
        print("Reading addresses from Excel...")
        addresses = geocoder.read_excel(excel_file, address_column)
        
        # Process addresses
        print(f"Processing {len(addresses)} addresses...")
        results = geocoder.batch_geocode(addresses)
        
        # Save results
        print("Saving results...")
        geocoder.save_results(results, output_file)
        
        print("Geocoding completed successfully!")
        
    except Exception as e:
        print(f"Error: {str(e)}")

if __name__ == "__main__":
    main()
