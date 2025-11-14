# RPA Job Search Process - Documentation

## 1. Introduction to RPA

**Robotic Process Automation (RPA)** is a technology that uses software robots (bots) to automate repetitive, rule-based
tasks that humans previously performed. RPA bots can interact with applications, websites, and systems in the
same way a human would—by clicking buttons, filling forms, extracting data, and performing other UI-based actions.

### Advantages of RPA:

- **Efficiency**: Bots can work 24/7 without breaks, significantly reducing processing time
- **Accuracy**: Eliminates human errors in repetitive tasks
- **Cost Reduction**: Frees up human resources for more value-added activities
- **Scalability**: Can easily scale to handle increased workloads
- **Non-Invasive**: Works with existing systems without requiring API integrations or system modifications
- **Quick Implementation**: Faster to deploy compared to traditional automation solutions

### Limitations of RPA:

- **Fragility**: Bots can break when UI elements change (e.g., website redesigns)
- **Limited Decision-Making**: Best suited for rule-based processes, not complex cognitive tasks
- **Maintenance Required**: Needs ongoing maintenance when target systems change
- **Not Suitable for All Processes**: Complex processes requiring judgment or creativity are not ideal
- **Initial Investment**: Requires setup time and resources, though ROI is typically positive

### RPA in Relation to This Project:

This job search automation project leverages the strengths of RPA in web scraping and data extraction. It automates the
repetitive task of checking job listings daily, extracting structured data, and delivering results via email. However,
the process is vulnerable to website structure changes, which is a common RPA limitation that requires monitoring and
maintenance.

## 2. Process Selection and Motivation

### Selected Process: Automated Job Search and Notification System

The chosen process automates the daily task of searching for job listings on Arbetsförmedlingen (Sweden's public
employment service), extracting relevant job information, and sending the results via email.

### Why This Process is Suitable for Automation:

1. **Highly Repetitive**: Job searching involves the same steps every day—navigating to a search page, applying filters,
   and reviewing results
2. **Rule-Based**: The process follows clear, predictable steps without requiring complex decision-making
3. **Time-Consuming**: Manual job searching can take 15–30 minutes daily, which adds up to significant time over weeks
4. **Structured Data**: Job listings are presented in a consistent format, making data extraction reliable
5. **Clear Value**: Automating this saves time and ensures consistent monitoring of job opportunities
6. **Low Complexity**: The process involves straightforward web interactions and data handling

### Process Analysis: Advantages and Disadvantages of Automation

#### Advantages:

- **Time Savings**: Automates a 15–30 minutes daily task, saving approximately 2–3 hours per week
- **Consistency**: Ensures job searches are performed at the same time daily without human forgetfulness
- **Comprehensive Coverage**: Can process all results systematically without missing opportunities
- **Immediate Notification**: Provides instant email notifications when new jobs are found
- **Data Organization**: Automatically structures job data into a CSV format for easy review
- **Scalability**: Can easily be extended to monitor multiple job search queries simultaneously

#### Disadvantages:

- **Website Dependency**: Vulnerable to changes in Arbetsförmedlingen's website structure
- **Limited Filtering Intelligence**: Cannot apply nuanced judgment about job relevance (e.g., company culture fit)
- **Maintenance Overhead**: Requires updates if the website changes its HTML structure
- **No Application Automation**: Only finds jobs; doesn't automate the application process
- **Potential for False Positives**: May extract jobs that don't perfectly match criteria if the website structure
  changes
- **Email Delivery Dependency**: Relies on email service configuration and may fail if credentials expire

## 3. Automation Flow Description

### High-Level Process Flow

The automation consists of five main workflows orchestrated by the `Main.xaml` entry point:

1. **Read Configuration** (`ReadJsonConfigWorkflow.xaml`)
2. **Web Scrape Jobs** (`WebscrapeJobsWorkflow.xaml`)
3. **Clean Job Data** (`CleanJobsTableWorkflow.xaml`)
4. **Save Results**
5. **Send Email** (`SendEmailWorkflow.xaml`)

### Detailed Workflow Steps

#### Step 1: Configuration Reading

- **Workflow**: `ReadJsonConfigWorkflow.xaml`
- **Purpose**: Loads configuration parameters from `config.json`
- **Process**:
    - Reads the JSON configuration file
    - Extracts three key parameters:
        - `recipientEmail`: Email address to receive job listings
        - `queryLink`: URL of the job search query on Arbetsförmedlingen
        - `outputFilePath`: Directory path where CSV files will be saved
- **Error Handling**: If the config file is missing or invalid, the workflow terminates with an error message

#### Step 2: Web Scraping

- **Workflow**: `WebscrapeJobsWorkflow.xaml`
- **Purpose**: Extracts job listings from the Arbetsförmedlingen website
- **Process**:
    - Opens Chrome browser
    - Navigates to the configured query link
    - Uses UiPath's data extraction capabilities to extract structured data from the job listing cards
    - Extracts the following fields for each job:
        - Job Title
        - Job Link (URL)
        - Company and City
        - Job Role
        - Published Date
    - Stores results in a DataTable structure
- **Technology**: Uses UiPath's NExtractData activity with structured data extraction

#### Step 3: Data Cleaning

- **Workflow**: `CleanJobsTableWorkflow.xaml`
- **Purpose**: Removes duplicates and cleans the extracted job data
- **Process**:
    - Receives the raw jobs DataTable
    - Removes duplicate entries (likely based on job link or title)
    - Performs data validation and cleaning
    - Returns a cleaned DataTable ready for export

#### Step 4: File Generation

- **Location**: Within `Main.xaml`
- **Purpose**: Converts cleaned data to CSV format and saves to disk
- **Process**:
    - Converts the cleaned DataTable to CSV text format
    - Generates a timestamped filename (format: `jobs-YYYY-MM-DD-HH-mm-ss.csv`)
    - Writes the CSV file to the configured output directory
    - Counts the number of jobs found

#### Step 5: Email Notification

- **Workflow**: `SendEmailWorkflow.xaml`
- **Purpose**: Sends an email with the job listings CSV as an attachment
- **Process**:
    - Reads the HTML email template (`email-body-template.html`)
    - Replaces the `{{JobCount}}` placeholder with the actual number of jobs found
    - Uses Gmail connection to send email via UiPath's GSuite activities
    - Attaches the generated CSV file
    - Sends it to the configured recipient email address
    - Subject line: "Inzane Bot: I Found Jobs. You're Welcome"
- **Technology**: Uses UiPath GSuite Activities with Gmail connection

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Main.xaml (Entry Point)                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │  ReadJsonConfigWorkflow.xaml          │
        │  - Read config.json                   │
        │  - Extract: email, queryLink, path    │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │  WebscrapeJobsWorkflow.xaml            │
        │  - Open Chrome                         │
        │  - Navigate to queryLink               │
        │  - Extract job listings                │
        │  - Return DataTable                    │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │  CleanJobsTableWorkflow.xaml           │
        │  - Clean data                          │
        │  - Return cleaned DataTable            │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │  Main.xaml (File Generation)           │
        │  - Convert to CSV                      │
        │  - Generate timestamped filename       │
        │  - Write to file system                │
        │  - Count jobs                          │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │  SendEmailWorkflow.xaml                │
        │  - Read email template                 │
        │  - Replace placeholders                 │
        │  - Attach CSV file                     │
        │  - Send via Gmail                      │
        └───────────────────────────────────────┘
```

## 4. Implementation Details

### Project Structure

```
rpa-job-search-process/
├── Main.xaml                          # Main entry point workflow
├── config.sample.json                 # Sample configuration file
├── email-body-template.html           # Email template with placeholders
├── project.json                       # UiPath project configuration
├── entry-points.json                  # Entry point definitions
└── Workflows/
    ├── ReadJsonConfigWorkflow.xaml    # Configuration reader
    ├── WebscrapeJobsWorkflow.xaml     # Web scraping logic
    ├── CleanJobsTableWorkflow.xaml    # Data cleaning
    └── SendEmailWorkflow.xaml         # Email sender
```

### Key Technologies and Activities Used

1. **Data Extraction**: `NExtractData` activity for structured web data extraction
2. **Browser Automation**: `NApplicationCard` and `NGoToUrl` for Chrome automation
3. **Data Processing**: `BuildDataTable`, `OutputDataTable` for data manipulation
4. **File Operations**: `ReadTextFile`, `WriteTextFile` for file handling
5. **Email**: `SendEmailConnections` from UiPath GSuite Activities
6. **Error Handling**: `TryCatch` blocks for robust error management
7. **JSON Processing**: `Newtonsoft.Json` for configuration parsing

### Configuration Requirements

The automation requires a `config.json` file with the following structure:

```json
{
  "recipientEmail": "your-email@example.com",
  "queryLink": "https://arbetsformedlingen.se/platsbanken/annonser?...",
  "outputFilePath": "C:/Users/your-username/Desktop"
}
```

### Gmail Connection Setup

The email functionality requires a Gmail connection configured in UiPath:

- Connection ID: `475abfec-75a1-4379-95e5-47612e9f9ded` (as specified in SendEmailWorkflow.xaml)
- Must be configured in UiPath Studio's Connection Manager
- Requires Gmail API access and OAuth2 authentication

## 5. Process Optimization and Complexity

### Optimizations Implemented

1. **Modular Workflow Design**: Each major step is separated into its own workflow file, enabling:
    - Reusability of components
    - Easier maintenance and debugging
    - Independent testing of each module
2. **Error Handling**: Try-Catch blocks ensure graceful failure with informative error messages
3. **Data Cleaning**: Deduplication prevents sending the same job multiple times
4. **Timestamped Files**: CSV files include timestamps to prevent overwriting and maintain history
5. **Conditional Email Sending**: Email is only sent if jobs are found (`jobsCount > 0`)
6. **Structured Data Extraction**: Uses UiPath's advanced extraction capabilities for reliable data capture

### Complexity Considerations

This automation demonstrates intermediate-level RPA complexity:

- **Multi-Workflow Orchestration**: Coordinates multiple workflows with data passing
- **Web Scraping with Dynamic Content**: Handles modern web applications with JavaScript-rendered content
- **Data Transformation**: Converts web data to structured format (DataTable → CSV)
- **External Service Integration**: Integrates with Gmail API for email delivery
- **Configuration Management**: Externalizes configuration for flexibility

## 6. Maintenance Considerations

### Regular Maintenance Tasks

1. **Monitor Website Changes**: Arbetsförmedlingen may update their website structure, requiring selector updates
2. **Gmail Connection**: OAuth tokens may expire and need renewal
3. **Configuration Updates**: Update query links if search criteria change
4. **Log Review**: Regularly check logs for errors or warnings

### Known Limitations

- Website structure changes will break the extraction selectors
- Gmail connection requires periodic re-authentication
- Network connectivity is required for web scraping and email sending
- Output directory must exist and be writable
