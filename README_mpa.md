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
  * `Link`
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

### 3. Add RSS Action to Fetch RSS Feed
1. In the flow, click on "+" and "Add an Action"
2. Search for "RSS" and select "List all feed items."
3. Configure the action:
   - **Feed URL:** Enter the RSS feed URL, e.g. my pubmed search is `https://pubmed.ncbi.nlm.nih.gov/rss/search/1pSbSzklLaRDgrBBecLaHXjj_NtDB256CbB-lTk3MQA9gZRkc4/?limit=100&utm_campaign=pubmed-2&fc=20240525000654`

### 4. Initialize Array Variable for New RSS Items
1. In the flow, click on "+" and "Add an Action"
2. Search for "Variable" and select "Initialize variable."
3. Configure the action:
   - **Name:** `NewRSSItems`
   - **Type:** Array

### 5. Initialize a Variable to Track Duplicates

1. **Initialize a Variable to Track Duplicates:**
   1. In the flow, click on "+" and "Add an Action."
   2. Search for "Variable" and select "Initialize variable."
   3. Configure the action:
      - **Name:** `DuplicateFound`
      - **Type:** Boolean
      - **Value:** `false`


### 6. Process the RSS items
1. In the flow, click on "+" and "Add an Action"
2. Search for "Compose" in the "Data Operation" category and select it
3. Configure the action:
   - **Inputs:** Select `Item` from the "List all feed items" action.

You'll notice that it gets put inside a "For Each" loop. This is expected.

### 7. Extract Title and Link from Each RSS Item

1. **Extract Title:**
   1. Inside the "Apply to each" action, click on "+" and "Add an Action."
   2. Search for "Compose" in the "Data Operation" category and select it.
   3. Configure the action:
      - **Title:** Up the top where it says Compose, write `Extract title`
      - **Inputs:** Click in the box an click the blue 'fx'
      - Enter the following expression in the text box:
        ```plaintext
        item()?['title']
        ```
      - Click "Add" to save the expression.

2. **Extract Link:**
   1. Inside the "Apply to each" action, click on "+" and "Add an Action."
   2. Search for "Compose" in the "Data Operation" category and select it.
   3. Configure the action:
      - **Title:** Up the top where it says Compose, write `Extract Link`
      - **Inputs:** Click in the box an click the blue 'fx'
      - Enter the following expression in the text box:
        ```plaintext
        item()?['primaryLink']
        ```
      - Click "Add" to save the expression.


### 8. Check for Duplicate Links

1. **Check for Duplicate Links:**
   1. In the flow, click on "+" and "Add an Action."
   2. Search for "Condition" and select it.
   3. Configure the action:
      - **Name:** `IsDuplicate`
      - **First "Choose a Value":** choose the lightning bolt, scroll down to "List rows present in a table" and choose "Link" 
      - **Second "Choose a Value":** choose the lightning bolt, and select the "outputs" from "Extract Link" 

