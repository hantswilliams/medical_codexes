# Quality Measure Code Extractor Assignment

## Real-World Context

This assignment simulates work you'd do as a clinical quality analyst at a health insurance plan, hospital system, or quality improvement organization. Healthcare providers must report on standardized quality measures (like HEDIS) to demonstrate they're providing evidence-based care. Your job is to build automated systems that extract the right patient populations and measure compliance using medical codes from claims or EHR data.

The skills you'll develop:
- Healthcare quality measure knowledge
- Complex data filtering and cohort building
- Working with multiple code systems simultaneously
- Clinical decision logic implementation
- Quality reporting and analytics

## Scenario
You're a clinical quality analyst at HealthFirst Insurance, a regional health plan. Your team must report annual HEDIS (Healthcare Effectiveness Data and Information Set) scores to NCQA for accreditation. Instead of manual chart review, you're building automated tools to identify eligible patients and measure compliance using claims data with medical codes.

## Assignment Overview
Build Python tools that extract patient cohorts and measure compliance for 5 key HEDIS quality measures. Your system should process claims data, apply eligibility criteria using medical codes, and generate compliance reports that match NCQA specifications.

## Target HEDIS Measures
Focus on these high-impact HEDIS measures with clear code specifications:

1. **CDC-A: Comprehensive Diabetes Care - HbA1c Testing**
2. **COL: Colorectal Cancer Screening**  
3. **BCS: Breast Cancer Screening**
4. **CBP: Controlling High Blood Pressure**
5. **AMM: Antidepressant Medication Management**

## Repository Setup
1. Create a **public** GitHub repository named `hedis-quality-extractor`
2. **Important**: Repository must be public for submission
3. Structure:
   ```
   hedis-quality-extractor/
   ├── input/
   ├── measures/
   │   ├── diabetes_care.py
   │   ├── colorectal_screening.py
   │   ├── breast_screening.py
   │   ├── blood_pressure.py
   │   └── depression_mgmt.py
   ├── extractors/
   │   ├── cohort_builder.py
   │   ├── compliance_calculator.py
   │   └── report_generator.py
   ├── reference_data/
   │   ├── hedis_code_lists.json
   │   └── measure_specifications.json
   ├── output/
   │   ├── cohorts/
   │   ├── compliance_reports/
   │   └── summary_statistics/
   ├── utils/
   │   └── measure_utils.py
   ├── test_data/
   ├── requirements.txt
   └── README.md
   ```

4. **Important**: Create `.gitignore` to exclude PHI:
   ```gitignore
   # Patient data (contains PHI)
   input/
   real_claims_data/
   *.csv
   *.xlsx
   patient_files/
   
   # Keep synthetic test data
   # test_data/ - keep these
   # reference_data/ - keep these
   
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

### 1. HEDIS Measure Specifications
Create detailed measure definitions in `reference_data/hedis_code_lists.json`:

```json
{
  "CDC_HbA1c_Testing": {
    "name": "Comprehensive Diabetes Care - HbA1c Testing",
    "description": "Diabetic patients aged 18-75 who had HbA1c test",
    "denominator": {
      "age_range": [18, 75],
      "diagnosis_codes": {
        "ICD10CM": ["E08", "E09", "E10", "E11", "E13"],
        "description": "Diabetes mellitus codes"
      },
      "continuous_enrollment": 365,
      "exclusions": {
        "ICD10CM": ["Z51.11", "Z51.12"],
        "description": "Hospice care"
      }
    },
    "numerator": {
      "lab_tests": {
        "CPT": ["83036", "83037"],
        "LOINC": ["4548-4", "17856-6", "4549-2"],
        "description": "HbA1c laboratory tests"
      }
    },
    "measurement_period": "annual",
    "target_rate": 0.90
  },
  
  "COL_Screening": {
    "name": "Colorectal Cancer Screening", 
    "description": "Adults aged 50-75 with colorectal cancer screening",
    "denominator": {
      "age_range": [50, 75],
      "continuous_enrollment": 365,
      "exclusions": {
        "ICD10CM": ["C18", "C19", "C20", "C21", "Z90.4"],
        "description": "Colorectal cancer history or colectomy"
      }
    },
    "numerator": {
      "procedures": {
        "CPT": ["45378", "45380", "45381", "45384", "45385"],
        "HCPCS": ["G0105", "G0121"],
        "description": "Colonoscopy and sigmoidoscopy"
      },
      "lab_tests": {
        "CPT": ["81528", "82270"],
        "description": "FIT and FIT-DNA tests"
      }
    },
    "lookback_period": 1095,
    "target_rate": 0.75
  }
}
```

### 2. Individual Measure Processors
For **each** HEDIS measure, create a processor that:

#### Patient Cohort Identification
- Apply age and enrollment criteria
- Filter by required diagnosis codes
- Exclude patients with exclusion criteria
- Handle different code formats (ICD-10, CPT, HCPCS, LOINC)

#### Compliance Assessment
- Search for numerator events (procedures, lab tests, prescriptions)
- Apply appropriate lookback periods
- Handle multiple qualifying event types
- Track compliance dates and frequencies

#### Data Quality Validation
- Verify code format validity
- Check for required data elements
- Flag incomplete or suspicious records
- Generate data quality reports

### 3. Cohort Builder Module
Create `extractors/cohort_builder.py` with functions:

```python
class HEDISCohortBuilder:
    def __init__(self, measure_specs):
        self.specs = measure_specs
        
    def build_denominator(self, claims_data, measure_name):
        """Build eligible patient cohort for measure"""
        spec = self.specs[measure_name]
        cohort = []
        
        for patient in claims_data:
            if self.meets_age_criteria(patient, spec['denominator']['age_range']):
                if self.has_qualifying_diagnosis(patient, spec['denominator']['diagnosis_codes']):
                    if not self.has_exclusions(patient, spec['denominator']['exclusions']):
                        cohort.append(patient)
        
        return cohort
    
    def calculate_compliance(self, cohort, measure_name):
        """Calculate numerator compliance for cohort"""
        spec = self.specs[measure_name]
        compliant_patients = []
        
        for patient in cohort:
            if self.has_numerator_events(patient, spec['numerator']):
                compliant_patients.append(patient)
        
        return compliant_patients
        
    def generate_measure_report(self, measure_name, cohort, compliant):
        """Generate compliance report for measure"""
        return {
            'measure': measure_name,
            'denominator_count': len(cohort),
            'numerator_count': len(compliant),
            'compliance_rate': len(compliant) / len(cohort) if cohort else 0,
            'target_rate': self.specs[measure_name]['target_rate']
        }
```

### 4. Reporting and Analytics
Generate comprehensive quality reports:

#### Measure-Level Reports
- Denominator and numerator counts
- Compliance rates vs. targets
- Trend analysis over time
- Statistical significance testing

#### Patient-Level Reports
- Individual patient compliance status
- Gap-in-care identification
- Next due dates for screenings
- Care opportunity alerts

#### Population Analytics
- Compliance by demographics (age, gender, geography)
- Provider performance comparisons
- Risk-adjusted outcomes
- Benchmark comparisons

---

## Sample Input Data Structure

Create synthetic test data in `test_data/sample_claims.csv`:

```csv
patient_id,age,gender,enrollment_start,claim_date,code,code_system,description
P001,45,M,2023-01-01,2023-03-15,E11.9,ICD10CM,Type 2 diabetes
P001,45,M,2023-01-01,2023-04-20,83036,CPT,HbA1c test
P002,62,F,2023-01-01,2023-05-10,Z12.11,ICD10CM,Colon cancer screening
P002,62,F,2023-01-01,2023-05-10,45378,CPT,Colonoscopy
P003,58,F,2023-01-01,2023-07-08,Z12.31,ICD10CM,Mammography screening
P003,58,F,2023-01-01,2023-07-08,77057,CPT,Bilateral mammography
```

---

## Expected Outputs

### Cohort Files (`output/cohorts/`)
```csv
measure,patient_id,denominator_eligible,numerator_compliant,gap_in_care
CDC_HbA1c_Testing,P001,Yes,Yes,No
CDC_HbA1c_Testing,P004,Yes,No,Yes
COL_Screening,P002,Yes,Yes,No
```

### Compliance Reports (`output/compliance_reports/`)
```json
{
  "reporting_period": "2023",
  "measures": [
    {
      "measure_id": "CDC_HbA1c_Testing",
      "measure_name": "Comprehensive Diabetes Care - HbA1c Testing",
      "denominator": 245,
      "numerator": 198,
      "rate": 0.808,
      "target": 0.90,
      "performance": "Below Target"
    }
  ],
  "overall_summary": {
    "total_measures": 5,
    "measures_meeting_target": 3,
    "overall_quality_score": 82.4
  }
}
```

---

## Optional Extensions

### Level 1: Advanced Analytics (Bonus)
- Risk stratification by patient characteristics
- Provider-level performance comparisons
- Time-series trend analysis
- Statistical process control charts

### Level 2: Integration Features (Bonus)
- Export to NCQA reporting formats
- Dashboard visualization (matplotlib/plotly)
- Automated email reports
- Database connectivity for large datasets

### Level 3: Clinical Decision Support (Bonus)
- Care gap alerts and reminders
- Next service due date calculations
- Risk-based care recommendations
- Population health management tools

---

## Real HEDIS Code References

Students should research actual HEDIS specifications from:
- **NCQA HEDIS Technical Specifications**: Available publicly
- **CMS Quality Measure Database**: qpp.cms.gov
- **AHRQ Quality Indicators**: qualityindicators.ahrq.gov

---

## Deliverables

### Required Files
- 5 HEDIS measure processors (one per measure)
- Cohort building and compliance calculation modules
- Comprehensive measure specifications (JSON)
- Quality reporting and analytics tools
- Synthetic test data and validation results

### Expected Libraries
- `pandas` - Data processing and analysis
- `json` - Configuration and reporting
- `datetime` - Date calculations and lookbacks
- `statistics` - Rate calculations and analytics
- Standard library: `csv`, `pathlib`, `logging`

---

## Submission

**Submit your public GitHub repository URL through Brightspace.**

Make sure your repository:
- Is **public** and accessible
- Contains all measure processors and test data
- Includes sample compliance reports and analytics
- Has comprehensive documentation with HEDIS measure explanations
- Demonstrates accurate cohort identification and compliance calculation

---

## Industry Impact

Quality measure reporting is a $billions industry requirement:
- **HEDIS scores** determine health plan star ratings and bonuses
- **Hospital quality scores** affect Medicare reimbursement rates
- **Provider quality metrics** influence patient choice and contracts
- **Population health management** depends on accurate measure calculation

Your automation tools directly impact healthcare quality improvement efforts and organizational performance in value-based care contracts.