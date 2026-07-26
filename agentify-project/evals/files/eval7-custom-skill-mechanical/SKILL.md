---
name: csv-helper
description: Parses and validates CSV exports from the billing pipeline, checking column headers and row counts before they're loaded downstream.
---

# CSV helper

1. Read the CSV file.
2. Validate the header row matches the expected schema.
3. Report any rows with missing required fields.
