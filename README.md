# 🧹 Data Cleaning Toolbox 📊
Welcome to the Data Cleaning Toolbox! A curated collection of useful and reusable functions to streamline the process of cleaning and preparing data for analysis or machine learning.

### Usual libraries
``` python
import csv
import pandas as pd
import numpy as np
import regex as re
import difflib
import logging
from googletrans import Translator
```

### Set up logging (for a Jupyter notebook, we will print logs directly to output)
``` python
# Configure logging
def setup_logger():
    logger = logging.getLogger("Data Cleaner")  # Create a logger object
    if not logger.hasHandlers():  # Avoid adding handlers multiple times
        logger.setLevel(logging.DEBUG)  # Set the minimum logging level

        # Create handlers
        file_handler = logging.FileHandler("breadcrumbs.log")  # Logs to a file
        console_handler = logging.StreamHandler()  # Logs to the VSCode terminal

        # Set logging level for handlers
        file_handler.setLevel(logging.DEBUG)
        console_handler.setLevel(logging.INFO)

        # Create formatters
        formatter = logging.Formatter(
            "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
            datefmt="%Y-%m-%d %H:%M:%S",
        )
        file_handler.setFormatter(formatter)
        console_handler.setFormatter(formatter)

        # Add handlers to the logger
        logger.addHandler(file_handler)
        logger.addHandler(console_handler)

    return logger

# Initialize the logger
logger = setup_logger()

# Log a message
logger.info("Logging configured successfully")
```
Logging Methods:

- logging.info: Logs informational messages.
- logging.error: Logs error messages.
- logging.exception: Logs error messages with a traceback.

Log Levels:
- DEBUG: Detailed information for debugging.
- INFO: General information about program execution.
- WARNING: An indication that something unexpected happened but the program can still continue.
- ERROR: A serious issue that prevents part of the program from functioning.
- CRITICAL: A very serious error indicating that the program may not continue.

### Read a CSV file and apply the result to a dataframe named 'df'
``` python
def read_csv(file_path, sep=';', header=0, engine=None):
    try:
        df = pd.read_csv(file_path, sep=sep, header=header, engine=engine)
        return df
    except Exception as e:
        print(f'Error reading csv: {e}')
        return None
```
### Translating all column headers to eanglish using googletrans
``` python
def translate_headers(file_path, output_path, file_type='csv'):
    """
    Translates the column headers of a given CSV or Excel file to English.
    
    :param file_path: Path to the input file (CSV or Excel)
    :param output_path: Path to save the translated file
    :param file_type: Type of file ('csv' or 'excel')
    """
    translator = Translator()
    
    # Load the file
    if file_type == 'csv':
        df = pd.read_csv(file_path)
    elif file_type == 'excel':
        df = pd.read_excel(file_path)
    else:
        raise ValueError("Unsupported file type. Use 'csv' or 'excel'.")
    
    # Translate headers
    translated_headers = {col: translator.translate(col, dest='en').text for col in df.columns}
    df.rename(columns=translated_headers, inplace=True)
    
    # Save the translated file
    if file_type == 'csv':
        df.to_csv(output_path, index=False)
    else:
        df.to_excel(output_path, index=False)
    
    print(f"File with translated headers saved to {output_path}")
```

### Standardizing column headers
standardize column headers by lowercasing all characters, removing all spaces and underscrores
``` python
def standardize_column_headers(df):
    df.columns = df.columns.str.lower().str.replace(' ', '').str.replace('_', '')
    return df
```
### Filtering necessary columns by namelist
``` python
def filter_dataframe(df, columns_to_keep):
    """
    Filters a DataFrame to retain only specified columns, with error handling and logging.

    Parameters:
        df (pd.DataFrame): The input DataFrame.
        columns_to_keep (list): List of column names to retain.

    Returns:
        pd.DataFrame: A filtered DataFrame with only the specified columns.
    """
    try:
        # Validate columns to keep
        valid_columns = [col for col in columns_to_keep if col in df.columns]
        
        if not valid_columns:
            logging.warning("None of the specified columns are in the DataFrame. Returning an empty DataFrame.")
            return pd.DataFrame()  # Return an empty DataFrame if no valid columns

        # Log any missing columns
        missing_columns = set(columns_to_keep) - set(valid_columns)
        if missing_columns:
            logging.warning(f"The following columns were not found in the DataFrame: {missing_columns}")

        # Keep only the valid columns
        filtered_df = df[valid_columns]
        logging.info(f"Successfully filtered DataFrame. Retained columns: {valid_columns}")
        return filtered_df

    except Exception as e:
        logging.error(f"An error occurred while filtering the DataFrame: {e}")
        raise  # Re-raise the exception after logging
```

### Validating email addresses
``` python
def validate_emails(df, column):
    try:
        email_pattern = r'^[a-zA-Z0-9._%+-]+@([^\d@]+\.[a-zA-Z]{2,})$'
        valid_domains = ['gmail.com', 'hotmail.com', 'yahoo.com', 'outlook.com', 'aol.com', 'icloud.com','naver.com','naver.net','hanmail.net']
        
        df[column] = df[column].str.replace(' ', '', regex=False)

        def clean_and_match_email(email):
            if isinstance(email, str) and '@' in email:
                username, domain = email.split('@', 1)
                domain = re.sub(r'[^a-zA-Z0-9.-]', '', domain).strip().replace(' ', '').lower()
                corrected_domain = difflib.get_close_matches(domain, valid_domains, n=1, cutoff=0.8)
                if corrected_domain:
                    domain = corrected_domain[0]
                return username + '@' + domain
            return email
        
        df[column] = df[column].apply(clean_and_match_email)
        invalid_emails = df[~df[column].str.match(email_pattern, na=False)]

        # Replace invalid emails with an empty string
        df.loc[~df['email'].str.match(email_pattern, na=False), column] = ""
        print(f'Found {len(invalid_emails)} invalid email addresses.')
        return df
    except Exception as e:
        print(f'Error validating emails: {e}')
        return df
```
### Cleaning and standardizing names
``` python
def clean_and_standardize_names(df, column_first_name):
    try:

        def retain_language_characters(text):
            """
            Retains all language-specific characters and removes everything else.
            This includes Unicode letters from all scripts and spaces.
            """
            if isinstance(text, str):
                # Use regex to match Unicode letters (all scripts) and spaces
                language_pattern = re.compile(r'[^\p{L}\s]', flags=re.UNICODE)
                return language_pattern.sub('', text).strip()
            return text

        df[column_first_name] = df[column_first_name].apply(retain_language_characters)
        df[column_first_name] = df[column_first_name].str.replace(r'\d+', '', regex=True).str.strip()

        def is_invalid_first_name(name: str) -> bool:
            if pd.isna(name) or not isinstance(name, str): return True
            if any(char.isdigit() for char in name) or any(char in set('~!@#$%^&*()_+=[]}{|\\:;,.<>?') for char in name):
                return True
            return False

        invalid_first_names = df[df[column_first_name].apply(is_invalid_first_name)]
        
        def clean_first_name(name):
            return '' if is_invalid_first_name(name) else name.title().strip()

        df[column_first_name] = df[column_first_name].apply(clean_first_name)
        return df
    except Exception as e:
        print(f'Error standardizing names: {e}')
        return df
```

### Validating phone numbers
``` python
def validate_phone_numbers(df, column):
    try:
        df[column] = df[column].astype(str).str.replace(r'\D', '', regex=True)
        
        def is_invalid_number(number: str) -> bool:
            if not (6 <= len(number) <= 16) or bool(re.search(r'[^0-9+]', number)):
                return True
            return False

        invalid_numbers = df[df[column].apply(is_invalid_number)]
        df.loc[df[column].apply(is_invalid_number), column] = ''

        print(f'Found {len(invalid_numbers)} invalid phone numbers.')
        return df
    except Exception as e:
        print(f'Error validating phone numbers: {e}')
        return df
```

### Removing duplicates
``` python
def remove_duplicates(df):
    try:
        initial_count = len(df)
        df = df.drop_duplicates()
        final_count = len(df)
        print(f'Removed {initial_count - final_count} duplicate rows.')
        return df
    except Exception as e:
        print(f'Error removing duplicates: {e}')
        return df
```
