# Job Application Extractor

A configurable skill for extracting job application details from saved HTML listings, generating schema-compliant YAML, and reporting processing results including incomplete or duplicate listings, uncertainties, and any research or Playwright used.

# How This will work

- By Appling you need to save every job listing html page using browser extension or "ctrl+s"
- Configure this folder of html pages as your skill "INPUT_FOLDER"
- Config Folder is provided in the repo: you can add or remove properties that you need or not need.
- Changing this will change what will be extracted
- Provide an hardcoed path for your "OUTPUT_FOLDER".
- The reports of this skill will be creatd in "yaml" format
- This reports can be indviaualley or together presented in the in this repo attached html file "Resume-Tracker-Dashboard.html".

The repository includes a predefined dashboard that can load one or more exported YAML files from the output folder. Load the files to view their entries together in the dashboard.

The dashboard provides an overview of the loaded files and applications, including status and application-method breakdowns, entries per file, applications per day, and summaries by city and company. You can search, filter by status, sort, and switch between card and table views.

<img width="2544" height="1261" alt="image" src="https://github.com/user-attachments/assets/f5bd240f-82a8-4811-a9ec-19bdde8542b4" />



## Workflow diagrams

### End-to-end application workflow

<img width="1371" height="850" alt="image" src="https://github.com/user-attachments/assets/6cecfb92-d962-421c-a337-3e9caa054313" />


### Job information extraction workflow (Breakdown)

<img width="698" height="913" alt="image" src="https://github.com/user-attachments/assets/1ee54e23-12a2-4c10-bb37-059843e880e6" />


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

## Processing report

After processing, report the number of files discovered and processed, incomplete records, duplicates, files that could not be processed, missing or uncertain properties, and any files that required Playwright or internet research.
