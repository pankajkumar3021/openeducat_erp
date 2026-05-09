# OpenEduCat Student India Customization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a new module `openeducat_student_india` that customizes the student model for Indian educational institutions by adding Aadhar number, religion, and caste fields while hiding visa and nationality fields from the UI.

**Architecture:** A new module that inherits from `op.student` model and extends it with India-specific fields. Old fields (visa_info, nationality) remain in the database but are hidden from forms/views. New fields have proper validation (Aadhar 12-digit format and uniqueness). Form views are inherited and modified to reorganize fields into a new "India Specific Information" section.

**Tech Stack:** Odoo 18.0, Python 3.x, XML for views, pytest for testing

---

## File Structure

```
openeducat_student_india/
├── __init__.py                    # Module initialization
├── __manifest__.py                # Module metadata
├── models/
│   ├── __init__.py               # Models package init
│   └── student.py                # Extended student model
├── views/
│   └── student_views.xml         # Form and list view customizations
└── tests/
    ├── __init__.py               # Tests package init
    └── test_student.py           # Student model tests
```

---

## Task 1: Initialize Module Structure & Manifest

**Files:**
- Create: `openeducat_student_india/__init__.py`
- Create: `openeducat_student_india/__manifest__.py`
- Create: `openeducat_student_india/models/__init__.py`
- Create: `openeducat_student_india/views/__init__.py`
- Create: `openeducat_student_india/tests/__init__.py`

- [ ] **Step 1: Create `__manifest__.py`**

```python
# openeducat_student_india/__manifest__.py

{
    'name': 'OpenEduCat Student India',
    'version': '18.0.1.0.0',
    'category': 'Education',
    'license': 'LGPL-3',
    'author': 'OpenEduCat Inc',
    'website': 'https://www.openeducat.org',
    'depends': ['openeducat_core'],
    'data': [
        'views/student_views.xml',
    ],
    'installable': True,
    'auto_install': False,
}
```

- [ ] **Step 2: Create `openeducat_student_india/__init__.py`**

```python
# openeducat_student_india/__init__.py

from . import models
```

- [ ] **Step 3: Create `openeducat_student_india/models/__init__.py`**

```python
# openeducat_student_india/models/__init__.py

from . import student
```

- [ ] **Step 4: Create `openeducat_student_india/views/__init__.py`**

```python
# openeducat_student_india/views/__init__.py
```

(empty file)

- [ ] **Step 5: Create `openeducat_student_india/tests/__init__.py`**

```python
# openeducat_student_india/tests/__init__.py
```

(empty file)

- [ ] **Step 6: Commit module structure**

```bash
git add openeducat_student_india/__init__.py \
        openeducat_student_india/__manifest__.py \
        openeducat_student_india/models/__init__.py \
        openeducat_student_india/views/__init__.py \
        openeducat_student_india/tests/__init__.py
git commit -m "[ADD] openeducat_student_india: initialize module structure"
```

---

## Task 2: Implement Extended Student Model

**Files:**
- Create: `openeducat_student_india/models/student.py`

- [ ] **Step 1: Create the inherited OpStudent model**

```python
# openeducat_student_india/models/student.py

from odoo import fields, models, api
from odoo.exceptions import ValidationError


class OpStudent(models.Model):
    _inherit = "op.student"

    # India-specific fields
    aadhar_number = fields.Char(
        'Aadhar Number',
        size=12,
        help='12-digit unique Aadhar identification number'
    )
    religion = fields.Selection(
        [
            ('hindu', 'Hindu'),
            ('muslim', 'Muslim'),
            ('christian', 'Christian'),
            ('sikh', 'Sikh'),
            ('buddhist', 'Buddhist'),
            ('jain', 'Jain'),
            ('other', 'Other'),
        ],
        string='Religion'
    )
    caste = fields.Selection(
        [
            ('general', 'General'),
            ('sc', 'SC'),
            ('st', 'ST'),
            ('obc_ncl', 'OBC-NCL'),
            ('obc_cl', 'OBC-CL'),
        ],
        string='Caste'
    )

    _sql_constraints = [
        ('unique_aadhar_number',
         'unique(aadhar_number)',
         'Aadhar Number must be unique!')
    ]

    @api.constrains('aadhar_number')
    def _check_aadhar_format(self):
        """Validate Aadhar number format: must be 12 digits"""
        for record in self:
            if record.aadhar_number:
                # Remove any spaces
                aadhar = record.aadhar_number.replace(' ', '')
                
                # Check if it's exactly 12 digits
                if not aadhar.isdigit() or len(aadhar) != 12:
                    raise ValidationError(
                        'Aadhar Number must be exactly 12 digits'
                    )
```

- [ ] **Step 2: Commit the student model**

```bash
git add openeducat_student_india/models/student.py
git commit -m "[ADD] openeducat_student_india: extend student model with India-specific fields"
```

---

## Task 3: Create Form View Customizations

**Files:**
- Create: `openeducat_student_india/views/student_views.xml`

- [ ] **Step 1: Create form view inheritance**

```xml
<!-- openeducat_student_india/views/student_views.xml -->

<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Student Form View Customization -->
        <record id="view_op_student_form_india" model="ir.ui.view">
            <field name="name">op.student.form.india</field>
            <field name="model">op.student</field>
            <field name="inherit_id" ref="openeducat_core.view_op_student_form"/>
            <field name="arch" type="xml">
                <!-- Hide visa_info and nationality fields -->
                <xpath expr="//field[@name='visa_info']" position="replace"/>
                <xpath expr="//field[@name='nationality']" position="replace"/>

                <!-- Add India-specific fields section after personal info -->
                <xpath expr="//field[@name='emergency_contact']" position="after">
                    <group string="India Specific Information" colspan="2">
                        <field name="aadhar_number" 
                               placeholder="12-digit Aadhar ID"/>
                        <field name="religion"/>
                        <field name="caste"/>
                    </group>
                </xpath>
            </field>
        </record>

        <!-- Student List View Customization -->
        <record id="view_op_student_tree_india" model="ir.ui.view">
            <field name="name">op.student.tree.india</field>
            <field name="model">op.student</field>
            <field name="inherit_id" ref="openeducat_core.view_op_student_tree"/>
            <field name="arch" type="xml">
                <!-- Hide visa_info and nationality from list view -->
                <xpath expr="//field[@name='visa_info']" position="replace"/>
                <xpath expr="//field[@name='nationality']" position="replace"/>
            </field>
        </record>
    </data>
</odoo>
```

- [ ] **Step 2: Commit form views**

```bash
git add openeducat_student_india/views/student_views.xml
git commit -m "[ADD] openeducat_student_india: customize student forms to show India-specific fields"
```

---

## Task 4: Write Unit Tests for Model Validation

**Files:**
- Create: `openeducat_student_india/tests/test_student.py`

- [ ] **Step 1: Create test file with test data setup**

```python
# openeducat_student_india/tests/test_student.py

from odoo.tests.common import TransactionCase
from odoo.exceptions import ValidationError


class TestOpStudentIndia(TransactionCase):
    """Test cases for India-specific student customization"""

    def setUp(self):
        super().setUp()
        # Create a test partner (student requires a partner)
        self.partner = self.env['res.partner'].create({
            'name': 'Test Student Partner',
            'email': 'test.student@example.com',
        })

    def test_aadhar_valid_format(self):
        """Test that valid Aadhar numbers are accepted"""
        student = self.env['op.student'].create({
            'first_name': 'Rajesh',
            'last_name': 'Kumar',
            'gender': 'm',
            'partner_id': self.partner.id,
            'aadhar_number': '123456789012',
            'religion': 'hindu',
            'caste': 'general',
        })
        self.assertEqual(student.aadhar_number, '123456789012')
        self.assertEqual(student.religion, 'hindu')
        self.assertEqual(student.caste, 'general')

    def test_aadhar_invalid_format_too_short(self):
        """Test that Aadhar numbers with less than 12 digits are rejected"""
        with self.assertRaises(ValidationError) as context:
            self.env['op.student'].create({
                'first_name': 'Rajesh',
                'last_name': 'Kumar',
                'gender': 'm',
                'partner_id': self.partner.id,
                'aadhar_number': '12345678901',  # Only 11 digits
            })
        self.assertIn('12 digits', str(context.exception))

    def test_aadhar_invalid_format_too_long(self):
        """Test that Aadhar numbers with more than 12 digits are rejected"""
        with self.assertRaises(ValidationError) as context:
            self.env['op.student'].create({
                'first_name': 'Rajesh',
                'last_name': 'Kumar',
                'gender': 'm',
                'partner_id': self.partner.id,
                'aadhar_number': '1234567890123',  # 13 digits
            })
        self.assertIn('12 digits', str(context.exception))

    def test_aadhar_invalid_format_non_numeric(self):
        """Test that Aadhar numbers with non-numeric characters are rejected"""
        with self.assertRaises(ValidationError) as context:
            self.env['op.student'].create({
                'first_name': 'Rajesh',
                'last_name': 'Kumar',
                'gender': 'm',
                'partner_id': self.partner.id,
                'aadhar_number': '123456789ABC',  # Contains letters
            })
        self.assertIn('12 digits', str(context.exception))

    def test_aadhar_unique_constraint(self):
        """Test that duplicate Aadhar numbers are rejected"""
        # Create first student
        self.env['op.student'].create({
            'first_name': 'Rajesh',
            'last_name': 'Kumar',
            'gender': 'm',
            'partner_id': self.partner.id,
            'aadhar_number': '123456789012',
        })

        # Create second partner for second student
        partner2 = self.env['res.partner'].create({
            'name': 'Test Student 2',
            'email': 'test2@example.com',
        })

        # Try to create second student with same Aadhar
        with self.assertRaises(Exception) as context:  # IntegrityError wrapped
            self.env['op.student'].create({
                'first_name': 'Priya',
                'last_name': 'Singh',
                'gender': 'f',
                'partner_id': partner2.id,
                'aadhar_number': '123456789012',  # Duplicate
            })
        self.assertIn('unique', str(context.exception).lower())

    def test_aadhar_optional(self):
        """Test that Aadhar number is optional"""
        student = self.env['op.student'].create({
            'first_name': 'Priya',
            'last_name': 'Singh',
            'gender': 'f',
            'partner_id': self.partner.id,
            # No aadhar_number provided
        })
        self.assertEqual(student.aadhar_number, False)

    def test_religion_selection_options(self):
        """Test that religion field accepts valid options"""
        religions = ['hindu', 'muslim', 'christian', 'sikh', 'buddhist', 'jain', 'other']
        for religion in religions:
            student = self.env['op.student'].create({
                'first_name': f'Student_{religion}',
                'last_name': 'Test',
                'gender': 'm',
                'partner_id': self.partner.id,
                'religion': religion,
            })
            self.assertEqual(student.religion, religion)

    def test_caste_selection_options(self):
        """Test that caste field accepts valid options"""
        castes = ['general', 'sc', 'st', 'obc_ncl', 'obc_cl']
        for caste in castes:
            partner = self.env['res.partner'].create({
                'name': f'Partner_{caste}',
                'email': f'test_{caste}@example.com',
            })
            student = self.env['op.student'].create({
                'first_name': f'Student_{caste}',
                'last_name': 'Test',
                'gender': 'm',
                'partner_id': partner.id,
                'caste': caste,
            })
            self.assertEqual(student.caste, caste)

    def test_visa_field_still_exists_in_db(self):
        """Test that visa_info field still exists for backward compatibility"""
        # This test verifies that the field exists in the database
        # even though it's hidden from the UI
        self.assertTrue(hasattr(self.env['op.student'], 'visa_info'))

    def test_nationality_field_still_exists_in_db(self):
        """Test that nationality field still exists for backward compatibility"""
        # This test verifies that the field exists in the database
        # even though it's hidden from the UI
        self.assertTrue(hasattr(self.env['op.student'], 'nationality'))
```

- [ ] **Step 2: Commit the test file**

```bash
git add openeducat_student_india/tests/test_student.py
git commit -m "[ADD] openeducat_student_india: add comprehensive unit tests for India-specific fields"
```

---

## Task 5: Verify Tests Pass

**Files:**
- Test: `openeducat_student_india/tests/test_student.py`

- [ ] **Step 1: Run all tests for the module**

```bash
cd /home/pkumar02/projects/openeducat_erp
python -m pytest openeducat_student_india/tests/test_student.py -v
```

Expected output:
```
test_student.py::TestOpStudentIndia::test_aadhar_valid_format PASSED
test_student.py::TestOpStudentIndia::test_aadhar_invalid_format_too_short PASSED
test_student.py::TestOpStudentIndia::test_aadhar_invalid_format_too_long PASSED
test_student.py::TestOpStudentIndia::test_aadhar_invalid_format_non_numeric PASSED
test_student.py::TestOpStudentIndia::test_aadhar_unique_constraint PASSED
test_student.py::TestOpStudentIndia::test_aadhar_optional PASSED
test_student.py::TestOpStudentIndia::test_religion_selection_options PASSED
test_student.py::TestOpStudentIndia::test_caste_selection_options PASSED
test_student.py::TestOpStudentIndia::test_visa_field_still_exists_in_db PASSED
test_student.py::TestOpStudentIndia::test_nationality_field_still_exists_in_db PASSED

===== 10 passed in X.XXs =====
```

- [ ] **Step 2: Verify all tests pass**

If all 10 tests pass, proceed to next task. If any fail, debug and re-run.

- [ ] **Step 3: Commit test results confirmation**

```bash
git add openeducat_student_india/tests/test_student.py
git commit -m "[TEST] openeducat_student_india: all unit tests passing"
```

---

## Task 6: Create Module Documentation

**Files:**
- Create: `openeducat_student_india/README.md`

- [ ] **Step 1: Create README documentation**

```markdown
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

Run the module tests:
```bash
python -m pytest openeducat_student_india/tests/test_student.py -v
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
```

- [ ] **Step 2: Commit documentation**

```bash
git add openeducat_student_india/README.md
git commit -m "[DOC] openeducat_student_india: add module README with features and usage"
```

---

## Task 7: Create __init__.py for Tests Module

**Files:**
- Create: `openeducat_student_india/tests/__init__.py`

- [ ] **Step 1: Initialize tests package**

```python
# openeducat_student_india/tests/__init__.py

from . import test_student
```

- [ ] **Step 2: Commit tests init**

```bash
git add openeducat_student_india/tests/__init__.py
git commit -m "[ADD] openeducat_student_india: initialize tests package"
```

---

## Task 8: Final Verification & Summary

**Files:**
- All module files created and tested

- [ ] **Step 1: Verify complete module structure**

```bash
cd /home/pkumar02/projects/openeducat_erp
find openeducat_student_india -type f | sort
```

Expected structure:
```
openeducat_student_india/__init__.py
openeducat_student_india/__manifest__.py
openeducat_student_india/models/__init__.py
openeducat_student_india/models/student.py
openeducat_student_india/views/__init__.py
openeducat_student_india/views/student_views.xml
openeducat_student_india/tests/__init__.py
openeducat_student_india/tests/test_student.py
openeducat_student_india/README.md
```

- [ ] **Step 2: Verify module can be imported**

```bash
cd /home/pkumar02/projects/openeducat_erp
python -c "import openeducat_student_india; print('Module imports successfully')"
```

Expected: `Module imports successfully`

- [ ] **Step 3: Run tests one final time**

```bash
cd /home/pkumar02/projects/openeducat_erp
python -m pytest openeducat_student_india/tests/ -v --tb=short
```

Expected: All 10 tests pass

- [ ] **Step 4: Create final summary commit**

```bash
git log --oneline | head -10
```

Verify that these commits appear:
- `[TEST] openeducat_student_india: all unit tests passing`
- `[DOC] openeducat_student_india: add module README`
- `[ADD] openeducat_student_india: add comprehensive unit tests`
- `[ADD] openeducat_student_india: customize student forms`
- `[ADD] openeducat_student_india: extend student model`
- `[ADD] openeducat_student_india: initialize module structure`

---

## Summary

This implementation plan creates a complete, tested `openeducat_student_india` module with:

✅ **Module Structure** - Complete Odoo module with proper initialization
✅ **Model Extensions** - Student model extended with aadhar_number, religion, caste fields
✅ **Validation** - Aadhar format validation (12 digits) and uniqueness constraint
✅ **Form Customization** - Forms show new fields, hide visa_info and nationality
✅ **Backward Compatibility** - Old fields preserved in DB, not deleted
✅ **Comprehensive Tests** - 10 test cases covering all validation scenarios
✅ **Documentation** - Complete README with usage instructions

**Total Implementation Time:** ~30-40 minutes
**Complexity:** Low-Medium (straightforward model extension, form inheritance)
**Risk Level:** Low (no changes to core model, isolated in new module)
