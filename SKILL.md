---
name: job-application-extractor
description: Extract job application information from saved HTML job listings using a configurable YAML schema, optional Playwright browsing, and web research.
---

# Job Application Extractor

## 1. Directory Configuration

Configure the following paths before processing:

```yaml
INPUT_FOLDER: "JobTracker/JobListings"
CONFIG_FILE: "JobTracker/Config/extraction.yaml"
OUTPUT_FOLDER: "JobTracker/Output"
```

These paths are configurable and must not be hardcoded elsewhere in the workflow.

- `INPUT_FOLDER`: Contains saved HTML job listings.
- `CONFIG_FILE`: Contains the required YAML output structure.
- `OUTPUT_FOLDER`: Destination for the extracted YAML file.

## 2. Objective

Process every saved job listing HTML file in the input directory.

Extract all properties defined in the configuration YAML, verify missing information when necessary, and generate one consolidated YAML file containing all job listings.

The extraction must work across different job websites without relying exclusively on website-specific HTML selectors.

## 3. Input and Output Schema

Read `CONFIG_FILE` before processing.

The expected structure is:

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

### Schema Rules

- Preserve the exact top-level property `Jobs`.
- Preserve all property names and their capitalization.
- Do not add, remove, or rename properties.
- Preserve the configured data types.
- Every job must contain all configured properties.
- Use `null` for unavailable nullable values.
- Use empty strings for unavailable string values.
- Use boolean values for `OnlineFormApplying`.

The configuration YAML is the source of truth for the output structure.

## 4. Discover and Process HTML Files

1. Scan `INPUT_FOLDER` recursively.
2. Discover every supported HTML file.
3. Process every discovered file.
4. Do not skip files because their filenames or website structures are unfamiliar.
5. Do not modify or delete original HTML files.
6. Record the source filename internally for traceability.

## 5. Extract Job Information

For every HTML file, extract the following properties according to the configuration:

### CompanyName
Identify the company offering the position.

### Address
Find the company's address when available.

Distinguish between the company's address and the job location.

### Email
Extract only an email address explicitly intended for receiving job applications.

Follow the special email rules in Section 6.

### OnlineFormApplying
Determine the application method using the rules in Section 6.

### ApplyingDate
Determine the date using the file creation date according to Section 7.

### Status
For newly discovered applications, use:

`Applied`

Do not assume an application was submitted if the source only represents an unsubmitted job listing. If submission cannot be established, use an empty string.

### Notes
Include useful additional information that does not belong in the other configured properties.

Examples:
- Remote work availability.
- Salary information.
- Special application requirements.
- Important uncertainties.

Do not add unsupported assumptions.

### JobTitle
Extract the actual position title, not the company name or a general page heading.

### Link
Use the original job listing URL.

Prefer the URL contained in the saved HTML metadata or canonical link. If unavailable, use the original URL recorded with the saved file or verify it through research.

## 6. OnlineFormApplying and Email Rules

This section defines how to determine the application method.

### Rule 1: Supported job platforms

If the original job listing belongs to one of these platforms:

- Indeed
- Xing
- StepStone

Set:

`OnlineFormApplying: true`

This is the default classification for these platforms.

### Rule 2: Specific application email

Inspect the listing for an email address explicitly designated for receiving job applications.

Examples of relevant wording:

- Bewerbung an:
- Bewerbungen bitte an:
- Senden Sie Ihre Bewerbung an:
- Bewerbung per E-Mail an:
- Please send your application to:
- Apply via email:

If a specific application email is found:

1. Extract the email address into `Email`.
2. Set `OnlineFormApplying: false`.

### Email Restrictions

Do not consider the following as application emails unless the listing explicitly identifies them as application contacts:

- General company contact addresses.
- Customer support addresses.
- Sales addresses.
- General HR department addresses.
- Addresses found in unrelated website sections or footers.

The email must be explicitly associated with submitting an application.

### Rule Priority

A specific application email always takes priority over the platform classification.

Examples:

| Listing source | Specific application email | OnlineFormApplying |
|---|---|---|
| Indeed | No | true |
| Xing | No | true |
| StepStone | No | true |
| Company website | No | true |
| Company website | Yes | false |
| Indeed | Yes | false |
| Xing | Yes | false |
| StepStone | Yes | false |

If no specific application email is found:

- `Email: null`
- Apply the platform/default classification.

For other websites without a specific application email, use `OnlineFormApplying: true` as the default.

## 7. Application Date Rules

The application date must be based on the saved HTML file's filesystem creation date.

Requirements:

1. Retrieve the file creation/birth timestamp.
2. Convert it to the local date.
3. Format it as `dd-mm-yyyy`.
4. Do not use the job publication date.
5. Do not use the current date.
6. Do not silently substitute the modification date.

If the filesystem does not provide a reliable creation timestamp, leave the date unavailable and report the issue.

Important: The file creation date is an approximation of the application date based on the user's saving convention. It does not independently prove when an application was submitted.

## 8. Internet Research and Playwright Skill

The agent may use internet research and another installed skill named `playwright`.

### When to use Playwright

Use Playwright when:

- The saved HTML is incomplete.
- Important information is dynamically loaded.
- The original job listing URL is available but information is missing.
- Additional information needs verification.

### Playwright Instructions

1. Check whether the `playwright` skill is installed and available.
2. Read its SKILL.md instructions before using it.
3. Follow the Playwright skill's documented workflow.
4. Open the original job listing URL whenever possible.
5. Retrieve the rendered page content.
6. Extract missing or additional information.
7. Do not submit applications.
8. Do not interact with application submission forms.
9. Do not modify website data.
10. If Playwright is unavailable, continue with the saved HTML and other permitted research methods.

### Internet Research

Internet research may be used to:

- Identify the company.
- Verify company addresses.
- Find missing job titles or listing information.
- Verify the original job listing URL.
- Find explicitly designated application email addresses.

Prefer:
1. The original job listing.
2. The official company website.
3. Reliable secondary sources.

Do not replace verified listing information with unrelated or outdated information.

Never invent missing information.

## 9. Extraction and Validation

For every processed HTML file:

1. Extract the available page content.
2. Identify the requested properties semantically.
3. Use HTML metadata and structured job data where available.
4. Use Playwright or internet research when necessary.
5. Apply the application method rules.
6. Determine the application date.
7. Validate the extracted values against the configuration.
8. Produce one record matching the configured schema.

### Duplicate Detection

Detect duplicate listings using:

1. Original job listing URL.
2. Company name and job title when the URL is unavailable.

Do not create duplicate records for the same position.

### Missing Information

- Never guess or fabricate information.
- Use `null` for missing nullable fields.
- Use empty strings for missing string fields.
- Use the configured default for boolean fields when applicable.
- Report important uncertainties.

## 10. Output Generation

Save the final result to:

`OUTPUT_FOLDER/jobs.yaml`

The output must follow the exact configured structure.

Example:

```yaml
Jobs:
  - CompanyName: "Bosch"
    Address: "Stuttgart, Germany"
    Email: null
    OnlineFormApplying: true
    ApplyingDate: "12-11-2026"
    Status: "Applied"
    Notes: ""
    JobTitle: "C# Software Developer"
    Link: "https://example.com/jobs/123"
```

### Output Rules

- Preserve the exact schema.
- Include one record per unique listing.
- Do not add extra properties.
- Preserve existing valid records when updating the output.
- Do not overwrite richer existing information with missing values.
- Keep the YAML syntactically valid.
- Do not modify original HTML files.

## 11. Processing Report

After processing all files, report:

- Total HTML files discovered.
- Successfully processed files.
- Incomplete records.
- Duplicate listings.
- Files that could not be processed.
- Missing or uncertain properties.
- Files for which Playwright or internet research was required.

## 12. General Rules

- Process all files recursively.
- Use the YAML configuration as the source of truth.
- Keep extraction independent of individual website structures.
- Prefer verified information over assumptions.
- Use Playwright as an optional supplementary tool.
- Never submit applications or modify websites.
- Never invent missing information.
- Keep the original HTML files unchanged.