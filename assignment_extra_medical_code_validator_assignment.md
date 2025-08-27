# Medical Code Validator & Formatter Assignment

## Real-World Context

This assignment simulates work you'd do as a quality assurance analyst at a medical billing company or EHR vendor. Healthcare organizations receive medical codes in various formats from different systems, often with inconsistent formatting, missing components, or invalid structures. Your job is to build validation and formatting tools to ensure data quality before codes enter billing or clinical systems.

The skills you'll develop:
- Data validation and quality assurance
- Regular expressions for pattern matching
- Medical coding standards knowledge
- Error handling and reporting
- Production-ready validation tools

## Scenario
You're a QA analyst at MedTech Solutions, a company that processes medical billing data for healthcare providers. Your team receives medical codes from various sources (hospitals, clinics, third-party systems) in different formats. Before this data enters the billing system, it must be validated and standardized to prevent costly claim rejections and compliance issues.

## Assignment Overview
Build Python validators and formatters for the major medical code types. Your tools should detect invalid codes, standardize formatting, and generate quality reports that help billing staff identify and fix data issues.

## Target Code Systems
Focus on these commonly used medical codes:

1. **ICD-10-CM** - Diagnosis codes (format: A00.0 to Z99.89)
2. **ICD-10-PCS** - Procedure codes (format: 7-character alphanumeric)
3. **CPT** - Current Procedural Terminology (format: 5-digit numeric)
4. **HCPCS Level II** - Healthcare procedure codes (format: 1 letter + 4 digits)
5. **NPI** - National Provider Identifier (format: 10-digit with checksum)

## Repository Setup
1. Create a **public** GitHub repository named `medical-code-validator`
2. **Important**: Repository must be public for submission
3. Structure:
   ```
   medical-code-validator/
   ├── input/
   ├── validators/
   │   ├── icd10cm_validator.py
   │   ├── icd10pcs_validator.py
   │   ├── cpt_validator.py
   │   ├── hcpcs_validator.py
   │   └── npi_validator.py
   ├── formatters/
   │   ├── code_formatter.py
   │   └── batch_processor.py
   ├── reports/
   ├── utils/
   │   └── validation_utils.py
   ├── test_data/
   ├── requirements.txt
   └── README.md
   ```

4. **Important**: Create `.gitignore` to exclude sensitive data:
   ```gitignore
   # Input data files (may contain PHI)
   input/
   *.xlsx
   *.csv
   test_files/
   
   # Keep sample/demo files
   # test_data/ - keep these
   # reports/ - keep these
   
   # Python
   __pycache__/
   *.pyc
   .env
   venv/
   
   # IDE
   .vscode/
   .idea/
   ```

---

## Core Requirements

### 1. Individual Code Validators
For **each** code system, create a validator that:

#### Code Format Validation
- Check if code matches the expected format/structure
- Validate character types (numeric vs. alphanumeric)
- Verify proper length and positioning
- Handle different input variations (with/without periods, spaces)

#### Business Rule Validation  
- **ICD-10-CM**: Valid category ranges, proper decimal placement
- **ICD-10-PCS**: Valid character values for each position
- **CPT**: Numeric range validation, modifier code handling
- **HCPCS**: Valid prefix letters, numeric suffix ranges  
- **NPI**: Luhn algorithm checksum validation

#### Error Detection & Reporting
- Identify specific validation failures
- Generate descriptive error messages
- Categorize error types for reporting
- Track validation statistics

### 2. Code Formatting Module
Create `formatters/code_formatter.py` with functions:

#### Standardization Functions
- `format_icd10cm(code)`: Add/remove periods, proper case
- `format_cpt(code)`: Remove extra characters, pad with zeros
- `format_npi(code)`: Remove hyphens/spaces, validate length
- `clean_whitespace(code)`: Remove extra spaces, normalize

#### Batch Processing
- Process lists of codes from CSV/Excel files
- Apply formatting rules consistently
- Preserve original codes for comparison
- Generate before/after reports

### 3. Validation Reports
Generate comprehensive quality reports:

#### Summary Statistics
- Total codes processed
- Validation pass/fail rates by code type
- Most common error types
- Processing time and throughput

#### Detailed Error Reports
- Line-by-line validation results
- Specific error descriptions
- Suggested corrections where possible
- Codes flagged for manual review

#### Export Formats
- CSV reports for spreadsheet analysis
- JSON for system integration
- HTML reports for stakeholder review. 

---

## Sample Code Structure

```python
# icd10cm_validator.py
import re
from utils.validation_utils import ValidationResult

class ICD10CMValidator:
    def __init__(self):
        self.pattern = re.compile(r'^[A-TV-Z][0-9][0-9AB]\.?[0-9A-TV-Z]{0,4}$')
        self.valid_categories = ['A00', 'B99', 'C00', 'D89']  # etc.
    
    def validate(self, code):
        """Validate a single ICD-10-CM code"""
        result = ValidationResult(code)
        
        # Format check
        if not self.pattern.match(code.upper().replace('.', '')):
            result.add_error("Invalid format")
            return result
            
        # Business rule checks
        category = code[:3]
        if category not in self.valid_categories:
            result.add_warning("Category not in current year range")
            
        return result
    
    def format(self, code):
        """Standardize ICD-10-CM code formatting"""
        clean_code = code.upper().replace(' ', '').replace('.', '')
        if len(clean_code) > 3:
            return f"{clean_code[:3]}.{clean_code[3:]}"
        return clean_code

# batch_processor.py
def process_code_file(filepath, validators):
    """Process a file containing mixed medical codes"""
    results = []
    
    with open(filepath, 'r') as file:
        for line_num, line in enumerate(file, 1):
            code = line.strip()
            
            # Auto-detect code type and validate
            code_type = detect_code_type(code)
            validator = validators.get(code_type)
            
            if validator:
                result = validator.validate(code)
                result.line_number = line_num
                results.append(result)
    
    return results
```

---

## Optional Extensions

### Level 1: Advanced Validation (Bonus)
- Code combination validation (diagnosis + procedure compatibility)
- Date range validation (code effective dates)
- Modifier code validation for CPT codes
- Cross-code system validation

### Level 2: Interactive Tools (Bonus)
- Command-line interface for single code validation
- Web-based validation tool (Flask/FastAPI)
- Real-time validation API endpoints
- Batch processing with progress indicators

### Level 3: Integration Features (Bonus)
- Database connectivity for reference data
- Email reports for validation results
- Scheduled validation jobs
- Integration with common EHR formats

---

## Test Data Examples

Create sample files in `test_data/` directory:

**mixed_codes_sample.csv**
```csv
code,expected_type,description
M79.3,ICD10CM,Joint pain
99213,CPT,Office visit
A0428,HCPCS,Ambulance service
1234567893,NPI,Provider ID
0016070,ICD10PCS,Procedure code
```

**validation_test_cases.csv**
```csv
code,code_type,should_pass,expected_error
M79.3,ICD10CM,true,
M79,ICD10CM,true,
M999.99,ICD10CM,false,Invalid category
99213,CPT,true,
99999,CPT,false,Invalid range
```

---

## Deliverables

### Required Files
- 5 individual validator classes (one per code system)
- Batch processing and formatting modules
- Validation utilities and helper functions
- Sample test data and validation reports
- Complete documentation and usage examples

### Expected Libraries
- `re` - Regular expression pattern matching
- `pandas` - Data processing (optional but recommended)
- `json` - Report generation
- `csv` - File processing
- `logging` - Error tracking
- `argparse` - Command-line interface (bonus)

### Sample Outputs
- Validation summary reports
- Error detail files
- Formatted code exports
- Performance metrics
- Test case results

---

## Submission

**Submit your public GitHub repository URL through Brightspace.**

Make sure your repository:
- Is **public** and accessible
- Contains all validator modules and test cases
- Includes sample validation reports
- Has comprehensive README with usage examples
- Demonstrates validation accuracy with test data

---

## Industry Impact

This type of tool is critical in healthcare IT. Invalid medical codes lead to:
- **Claim denials** costing billions annually
- **Compliance violations** with hefty penalties  
- **Revenue cycle delays** affecting cash flow
- **Data quality issues** impacting patient care

Your validation tools directly address these real business problems, making this assignment highly relevant to healthcare technology careers.