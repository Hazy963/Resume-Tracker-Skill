# Job Application Extractor

A configurable skill for extracting job application details from saved HTML job listings and writing consolidated YAML output. The output structure is defined by a YAML configuration file, so the skill can follow your chosen schema.

## Workflow diagrams

### End-to-end application workflow

![End-to-end application workflow](docs/images/application-workflow.png)

### Job information extraction workflow

![Job information extraction workflow](docs/images/extraction-workflow.png)

## Configure folders

Set these paths before processing. They are configurable and should not be hardcoded elsewhere in the workflow.

```yaml
INPUT_FOLDER: "JobTracker/JobListings"
CONFIG_FILE: "JobTracker/Config/extraction.yaml"
OUTPUT_FOLDER: "JobTracker/Output"
```

- `INPUT_FOLDER` contains saved HTML job listings.
- `CONFIG_FILE` defines the required YAML output structure.
- `OUTPUT_FOLDER` receives the generated `jobs.yaml` file.

## Configure the output schema

The skill reads the configuration before processing and preserves its top-level property, field names, capitalization, and data types. The following is the example schema:

```yaml
Jobs:
  - CompanyName: ""
    Address: ""
    Email: null
    OnlineFormApplying: true
    ApplyingDate: "dd-mm-yyyy"
    Status: ""
    Notes: ""
    JobTitle: ""
    Link: ""
```

Every job record must include all configured properties. Use `null` for unavailable nullable values, empty strings for unavailable string values, and booleans for boolean values. Do not add, remove, or rename fields.

## How extraction works

The skill scans the input folder recursively, processes supported HTML files, and creates one record for each unique listing. It uses page content, HTML metadata, and structured job data where available. It can use internet research or the installed Playwright skill to verify missing information when appropriate.

Key extraction rules:

- **Application email:** Include an address only when the listing explicitly identifies it for receiving applications. A specific application email sets `OnlineFormApplying` to `false`.
- **Application method:** Without a specific application email, `OnlineFormApplying` is `true` by default, including for Indeed, Xing, and StepStone listings.
- **Application date:** Use the saved HTML file’s filesystem creation date, converted to local time and formatted as `dd-mm-yyyy`. Do not substitute the publication date, current date, or modification date. Report the date as unavailable if no reliable creation timestamp exists.
- **Status:** Use `Applied` only when submission can be established. Otherwise, leave it empty.
- **Link:** Prefer the original listing URL in the HTML metadata or canonical link; verify it when needed.
- **Duplicates:** Check the original listing URL first, then company name and job title if the URL is unavailable.
- **Missing details:** Never guess or fabricate information. Preserve richer existing values when updating output.

The original HTML files are not modified or deleted. The skill does not submit applications, interact with application submission forms, or modify websites.

## Output

The result is saved as:

```text
OUTPUT_FOLDER/jobs.yaml
```

It contains one record per unique listing and follows the configured schema. The YAML must be syntactically valid, and existing valid records should be preserved when updating the output.

## View your applications in the dashboard

The repository includes a predefined dashboard that can load one or more exported YAML files from the output folder. Load the files to view their entries together in the dashboard.

The dashboard provides an overview of the loaded files and applications, including status and application-method breakdowns, entries per file, applications per day, and summaries by city and company. You can search, filter by status, sort, and switch between card and table views.

![Job application dashboard showing loaded YAML files and application summaries](docs/images/dashboard-preview.png)

## Processing report

After processing, report the number of files discovered and processed, incomplete records, duplicates, files that could not be processed, missing or uncertain properties, and any files that required Playwright or internet research.