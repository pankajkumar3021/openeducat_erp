# OpenEduCat Student India Module

This module extends the OpenEduCat student model with India-specific customizations.

## Overview

The `openeducat_student_india` module customizes the student model for Indian educational institutions by:
- Adding India-specific fields: Aadhar Number, Religion, and Caste
- Hiding unnecessary fields from the UI (Visa Info, Nationality) while preserving data in the database
- Providing proper validation for all new fields

## Features

### New Fields

1. **Aadhar Number** (Optional)
   - Format: 12-digit unique number
   - Validation: Exact 12 digits, numeric only
   - Constraint: Unique per student (prevents duplicates)
   - Stored in database for identification and compliance

2. **Religion** (Optional)
   - Selection options: Hindu, Muslim, Christian, Sikh, Buddhist, Jain, Other
   - Used for institutional records and demographic tracking

3. **Caste** (Optional)
   - Selection options: General, SC (Scheduled Caste), ST (Scheduled Tribe), OBC-NCL, OBC-CL
   - Used for scholarship eligibility, reservations, and affirmative action tracking

### Hidden Fields

The following fields are hidden from the student form UI but remain in the database for backward compatibility:
- **Visa Info** - No longer required for Indian students
- **Nationality** - Assumed Indian for all students in this configuration

## Installation

1. Place the module in your Odoo addons directory
2. Install the module through Odoo UI or command line:
   ```bash
   odoo -d database_name -u openeducat_student_india
   ```

## Usage

After installation:
1. Open any student record
2. You'll see the new "India Specific Information" section with the new fields
3. Fill in Aadhar number (12 digits), Religion, and Caste as needed
4. The form will validate Aadhar format automatically

## Validation Rules

- **Aadhar Number**: Must be exactly 12 numeric digits if provided. System prevents duplicate Aadhar numbers.
- **Religion & Caste**: Free selection from predefined lists based on Indian demographics

## Data Migration

This module preserves all existing student data:
- Existing visa_info and nationality values remain in the database
- These fields are simply hidden from forms and list views
- If needed in the future, the fields can be made visible again by removing the view customizations

## Testing

Run the module tests when Odoo is running:
```bash
odoo -d database_name -i openeducat_student_india --test-tags=openeducat_student_india
```

All tests verify:
- Aadhar validation (format, uniqueness, optional)
- Religion and Caste selection options
- Backward compatibility with existing fields

## Dependencies

- `openeducat_core` >= 18.0

## License

LGPL-3.0 (Same as OpenEduCat)

## Author

OpenEduCat Inc
