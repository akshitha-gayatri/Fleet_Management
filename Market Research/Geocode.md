# Geocoding Employee Addresses
## 11 February 2025

## Overview
This application provides a simple and efficient way to geocode addresses using the `Nominatim` geocoding service from the `geopy` library. Users can geocode single addresses, batch process multiple addresses, or read addresses from an Excel file for geocoding.

## Features
- **Single Address Geocoding:** Enter a single address and get its coordinates.
- **Batch Address Geocoding:** Enter multiple addresses for batch processing.
- **Excel File Geocoding:** Read addresses from an Excel file and save the geocoded results to a new Excel file.
- **Error Handling:** Includes retry logic and exponential backoff for geocoding failures.

## Prerequisites
Ensure the following dependencies are installed:

- Python 3.6+
- `geopy`
- `pandas`
- `openpyxl` (for Excel file operations)

Install the required packages using:
```bash
pip install geopy pandas openpyxl
```

## Usage Instructions
Upon running the program, you will see the following menu:

```
Select an option:
1. Geocode a single address
2. Batch geocode multiple addresses
3. Geocode addresses from an Excel file
4. Exit
Enter your choice: 
```

### 1. Geocode a Single Address
- Select option `1`.
- Enter the address when prompted.
- The program will display the coordinates if geocoding is successful.

### 2. Batch Geocode Multiple Addresses
- Select option `2`.
- Enter addresses one by one. Type `done` to finish.
- The program will display the geocoded results.

### 3. Geocode Addresses from an Excel File
- Select option `3`.
- Provide the following inputs when prompted:
  - Path to the input Excel file.
  - Name of the column containing addresses.
  - Path to save the output Excel file.
- The program will save the geocoded results to the specified output file.

### 4. Exit
- Select option `4` to exit the program.

## Example Outputs
### Single Address Geocoding
```
Enter the address to geocode: Coordinates for IIIT Hyderabad, Gachibowli, Hyderabad, Telangana, India
Coordinates for IIIT Hyderabad, Gachibowli, Hyderabad, Telangana, India: (17.4454957, 78.34854697544472)
```

### Batch Geocoding
```
Enter addresses one by one. Type 'done' when finished.
Address: India Gate, New Delhi, India
Address: Taj Mahal, Agra, India
Address: Gateway of India, Mumbai, India
Address: done

Batch geocoding results:
                            address  latitude  longitude  success
0     India Gate, New Delhi, India  28.612933  77.229493     True
1           Taj Mahal, Agra, India  27.175189  78.042066     True
2  Gateway of India, Mumbai, India  18.921966  72.834566     True
```

## Error Handling
- Retry logic with exponential backoff for `GeocoderTimedOut` and `GeocoderServiceError` exceptions.
- Cache mechanism to avoid repeated geocoding of the same address.

## Code Structure
### `EmployeeGeocoder` Class
- **`__init__`:** Initializes the geocoder with a user agent and cache.
- **`geocode_address`:** Geocodes a single address with retry logic.
- **`batch_geocode`:** Geocodes a list of addresses and returns a DataFrame.
- **`geocode_from_excel`:** Reads addresses from an Excel file, geocodes them, and saves the results.

### `main()` Function
- Provides a user-friendly terminal menu for selecting geocoding options.

## Full Code
```python
from geopy.geocoders import Nominatim
from geopy.exc import GeocoderTimedOut, GeocoderServiceError
import pandas as pd
import time
from typing import List, Dict, Optional, Tuple

class EmployeeGeocoder:
    def __init__(self, user_agent: str = "employee_geocoder"):
        """
        Initialize the geocoder with a specific user agent.
        
        Args:
            user_agent (str): Identifier for the geocoding service
        """
        self.geolocator = Nominatim(user_agent=user_agent)
        self.cache: Dict[str, Optional[Tuple[float, float]]] = {}
    
    def geocode_address(self, address: str, max_retries: int = 3) -> Optional[Tuple[float, float]]:
        """
        Geocode a single address with retry logic.
        
        Args:
            address (str): Address to geocode
            max_retries (int): Maximum number of retry attempts
            
        Returns:
            Optional[Tuple[float, float]]: (latitude, longitude) if successful, None otherwise
        """
        if address in self.cache:
            return self.cache[address]
            
        for attempt in range(max_retries):
            try:
                location = self.geolocator.geocode(address)
                if location:
                    result = (location.latitude, location.longitude)
                    self.cache[address] = result
                    return result
                time.sleep(1)  # Respect rate limits
            except (GeocoderTimedOut, GeocoderServiceError):
                if attempt == max_retries - 1:
                    print(f"Failed to geocode address after {max_retries} attempts: {address}")
                time.sleep(2 ** attempt)  # Exponential backoff
        
        self.cache[address] = None
        return None
    
    def batch_geocode(self, addresses: List[str]) -> pd.DataFrame:
        """
        Geocode a list of addresses and return results as a DataFrame.
        
        Args:
            addresses (List[str]): List of addresses to geocode
            
        Returns:
            pd.DataFrame: DataFrame with original addresses and their coordinates
        """
        results = []
        for address in addresses:
            coords = self.geocode_address(address)
            results.append({
                'address': address,
                'latitude': coords[0] if coords else None,
                'longitude': coords[1] if coords else None,
                'success': coords is not None
            })
        
        return pd.DataFrame(results)
    
    def geocode_from_excel(self, input_file: str, address_column: str, output_file: str) -> None:
        """
        Read addresses from Excel file, geocode them, and save results.
        
        Args:
            input_file (str): Path to input Excel file
            address_column (str): Name of the column containing addresses
            output_file (str): Path to save the output Excel file
        """
        df = pd.read_excel(input_file)
        addresses = df[address_column].tolist()
        results_df = self.batch_geocode(addresses)
        
        # Merge results with original data
        final_df = pd.concat([df, results_df.drop('address', axis=1)], axis=1)
        final_df.to_excel(output_file, index=False)
        print(f"Geocoding complete. Results saved to {output_file}")

# Example usage
if __name__ == "__main__":
    # Initialize geocoder
    geocoder = EmployeeGeocoder()
    
    while True:
        print("Select an option:")
        print("1. Geocode a single address")
        print("2. Batch geocode multiple addresses")
        print("3. Geocode addresses from an Excel file")
        print("4. Exit")
        choice = input("Enter your choice: ")
        
        if choice == "1":
            address = input("Enter the address to geocode: ")
            coords = geocoder.geocode_address(address)
            if coords:
                print(f"Coordinates for {address}: {coords}")
            else:
                print("Failed to geocode the address.")
        
        elif choice == "2":
            addresses = []
            print("Enter addresses one by one. Type 'done' when finished.")
            while True:
                address = input("Address: ")
                if address.lower() == 'done':
                    break
                addresses.append(address)
            results = geocoder.batch_geocode(addresses)
            print("\nBatch geocoding results:")
            print(results)
        
        elif choice == "3":
            input_file = input("Enter the input Excel file path: ")
            address_column = input("Enter the address column name: ")
            output_file = input("Enter the output Excel file path: ")
            geocoder.geocode_from_excel(input_file, address_column, output_file)
        
        elif choice == "4":
            print("Exiting the program.")
            break
        
        else:
            print("Invalid choice. Please try again.")
```

