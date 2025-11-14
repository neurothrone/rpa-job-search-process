# RPA Job Search Process

A UiPath automation project that scrapes job listings from Arbetsförmedlingen (Sweden's public employment service) and
sends the results via email.

## Prerequisites

- UiPath Studio installed
- Google account with Gmail access
- Chrome browser installed
- Internet connection

## Setup Instructions

### 1. Create Configuration File

1. Copy the sample configuration file:
   ```
   Copy `config.sample.json` to `config.json`
   ```

2. Edit `config.json` and update the following values:
   ```json
   {
     "recipientEmail": "your-email@example.com",
     "queryLink": "https://arbetsformedlingen.se/platsbanken/annonser?p=4:apaJ_2ja_LuF&q=systemutvecklare%20cloud%20azure&l=3:PVZL_BQT_XtL;3:CCiR_sXa_BVW",
     "outputFilePath": "C:/Users/your-username/Desktop"
   }
   ```

   **Configuration Parameters:**
    - `recipientEmail`: The email address where job listings will be sent
    - `queryLink`: The full URL of your job search query on Arbetsförmedlingen. To get this:
        - Go to [Arbetsförmedlingen Platsbanken](https://arbetsformedlingen.se/platsbanken)
        - Perform your job search with desired filters (keywords, location, etc.)
        - Copy the complete URL from the browser address bar
    - `outputFilePath`: The directory path where CSV files will be saved (must exist and be writable)

### 2. Configure Gmail Connection

The email functionality requires a Gmail connection to be set up in UiPath:

1. Open the project in UiPath Studio
2. Navigate to `Workflows/SendEmailWorkflow.xaml`
3. Locate the `Send Email` activity (SendEmailConnections)
4. Click on the connection field or use UiPath's Connection Manager:
    - Go to **Home** tab → **Connections** (or use the Connection Manager)
    - Click **+ New Connection**
    - Select **Gmail** from the list of available connections
    - Follow the OAuth2 authentication flow:
        - Sign in with your Google account
        - Grant necessary permissions for Gmail API access
        - Complete the connection setup
5. The connection will be saved and can be reused for future runs

### 3. Verify Project Dependencies

Ensure the following UiPath packages are installed (should be included in `project.json`):

- UiPath.UIAutomation.Activities
- UiPath.GSuite.Activities
- UiPath.Excel.Activities
- UiPath.Web.Activities

If packages are missing, restore them via:

- **Manage Packages** → **Restore**

## Running the Automation

1. Open `Main.xaml` in UiPath Studio
2. Ensure `config.json` exists in the project root directory
3. Click **Run** (F5) or **Debug** (F6)

## Project Structure

```
rpa-job-search-process/
├── Main.xaml                          # Main entry point
├── config.json                        # Configuration file (create from config.sample.json)
├── config.sample.json                 # Sample configuration template
├── email-body-template.html           # Email body template
├── project.json                       # Project configuration
├── entry-points.json                  # Entry point definitions
└── Workflows/
    ├── ReadJsonConfigWorkflow.xaml    # Reads configuration
    ├── WebscrapeJobsWorkflow.xaml     # Scrapes job listings
    ├── CleanJobsTableWorkflow.xaml    # Cleans job data
    └── SendEmailWorkflow.xaml         # Sends email with results
```

## Workflow Overview

1. **Read Configuration**: Loads settings from `config.json`
2. **Web Scrape**: Extracts job listings from Arbetsförmedlingen
3. **Clean Data**: Removes duplicates and validates data
4. **Save Results**: Generates timestamped CSV file
5. **Send Email**: Emails the CSV file to the configured recipient

## Output

- **CSV File**: Generated in the specified output directory with format `jobs-YYYY-MM-DD-HH-mm-ss.csv`
- **Email**: Sent to the configured recipient with the CSV file attached
- **Log Messages**: Available in UiPath Studio's Output panel

## Troubleshooting

### Configuration File Not Found

- Ensure `config.json` exists in the project root directory
- Verify the file name is exactly `config.json` (not `config.json.txt`)

### Gmail Connection Issues

- Verify the Gmail connection is properly configured in UiPath
- Check that OAuth2 authentication completed successfully
- Ensure your Google account has Gmail API access enabled
- Try removing and recreating the Gmail connection

### Web Scraping Fails

- Verify the `queryLink` in `config.json` is valid and accessible
- Check that Arbetsförmedlingen website structure hasn't changed
- Ensure Chrome browser is installed and up to date
- Check internet connectivity

### Email Not Sent

- Verify Gmail connection is configured correctly
- Check that `recipientEmail` in config.json is a valid email address
- Ensure the CSV file was generated successfully (check output directory)
- Review UiPath logs for error messages

## Documentation

For detailed documentation about the automation process, design decisions, and analysis,
see [documentation](documentation.md).

## Visual Preview

### Generated CSV Output

The automation generates a CSV file containing all extracted job listings with structured data:

![CSV Data Preview](assets/csv-data-preview.png)

### Email Sent from Gmail

The automation sends an email via Gmail with the job listings CSV file attached:

![Gmail Send Preview](assets/gmail-send-preview.png)

### Email Received in Outlook

The recipient receives the email with the attached CSV file containing all job listings:

![Outlook Receive Preview](assets/outlook-receive-preview.png)

## License

This project is created for educational purposes as part of an RPA course assignment.
