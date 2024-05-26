# Microsoft Power Automate instructions

So, dlvr.it has massively constrained the free option. Many academics will have access to Microsoft Power Automate through an institutional subscription to Office365, so here's a set of instructions for getting going.

They're a little complicated, but the principle is simple. 

We set up an excel sheet to store all the papers for our literature bot, recording the title, the DOI, and whether that paper has been posted yet.

Then, we use PowerAutomate to populate that sheet from multiple RSS feeds, adding new papers every hour.

Finally, we use PowerAutomate to post un-posted papers in a sensible way to a Bluesky account.

# Set up the Excel Sheet

1. Log in to Office365
2. Set up a blank Excel Workbook
3. Make three columns:
  * `Title`
  * `DOI`
  * `Posted`
4. At the top left, name it `literature_bot`
5. Select the three columns
6. Click "Insert" then "Table"
7. Leave "My Table Has Headers" checked, and click "OK"

# Set up your Power Automate Flow

I hate power automate, and you may come to hate it too. Here we're going to make a 'Flow', which first gets everything from our spreasheet, then gets the RSS feed, and then adds anything new from the RSS feed to the spreadsheet. Take a deep breath...

### 1. Create a Scheduled Flow
1. Go to [Power Automate](https://flow.microsoft.com).
2. Click on "Create" > "Scheduled cloud flow."
3. Name your flow "literature_bot"
4. Set it to run every hour and click "Create."

### 2. Add Excel Action to List Rows
1. In the flow, click on "+" and "Add an Action"
2. Search for "Excel" and select "List rows present in a table."
3. Configure the action:
   - **Location:** OneDrive for Business
   - **Document Library:** OneDrive
   - **File:** Select the "literature_bot.xlsx" Excel file you created.
   - **Table:** Select the table (for me it's called "Table1")

### 3. Add HTTP Action to Fetch RSS Feed
1. Click on "New step."
2. Search for "HTTP" and select "HTTP" action.
3. Configure the action:
   - **Method:** GET
   - **URI:** Enter the RSS feed URL.

### 4. Parse RSS Feed
1. Click on "New step."
2. Search for "XML" and select "XML - Parse XML."
3. Configure the action:
   - **Content:** Select the body from the HTTP request.
   - **Schema:** Use a sample schema or leave it for now; we will adjust it later.

### 5. Initialize Array Variable for New RSS Items
1. Click on "New step."
2. Search for "Variable" and select "Initialize variable."
3. Configure the action:
   - **Name:** NewRSSItems
   - **Type:** Array

### 6. Apply to Each RSS Item
1. Click on "New step."
2. Search for "Control" and select "Apply to each."
3. Configure the action:
   - **Select an output from previous steps:** Select the parsed XML items.

### 7. Check for Existing DOIs
1. Inside the "Apply to each" action, add a "Con

