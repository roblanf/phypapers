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

### 3. Initialise an Array of All Titles we already have
1. In the flow, click on "+" and "Add an Action"
2. Search for "Initialize variable" and select it.
3. Configure the action:
   - **Name:** `ExistingTitles`
   - **Type:** Array
   - **Value:** leave empty

### 4. Fill the Array of All Titles we already have
1. Click on "+" and "Add an Action."
2. Search for "Apply to each" and select it.
3. Configure the action:
   - **Select an output from previous steps:** click the lightning bolt, then `value` from "List rows present in a table."
4. Inside the loop, click on "+" and "Add an Action."
5. Search for "Append to array variable" and select it.
6. Configure the action:
   - **Name:** `ExistingTitles`
   - **Value:** click the lightning bolt, then choose `Title` from Excel rows.


### 5. Add RSS Action to Fetch RSS Feed
1. In the flow, click on "+" and "Add an Action"
2. Search for "RSS" and select "List all feed items."
3. Configure the action:
   - **Feed URL:** Enter the RSS feed URL, e.g. my pubmed search is `https://pubmed.ncbi.nlm.nih.gov/rss/search/1pSbSzklLaRDgrBBecLaHXjj_NtDB256CbB-lTk3MQA9gZRkc4/?limit=100&utm_campaign=pubmed-2&fc=20240525000654`

### 6. Initialize Array Variable for New RSS Items
1. In the flow, click on "+" and "Add an Action"
2. Search for "Variable" and select "Initialize variable."
3. Configure the action:
   - **Name:** `NewRSSItems`
   - **Type:** Array

### 6. Initialize a Variable to Track Duplicates
1. In the flow, click on "+" and "Add an Action."
2. Search for "Variable" and select "Initialize variable."
3. Configure the action:
   - **Name:** `DuplicateFound`
   - **Type:** Boolean
   - **Value:** `false`

### 7. Process the RSS items
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

1. In the flow, click on "+" and "Add an Action."
2. Search for "Condition" and select it.
3. Configure the action:
   - **Name:** `IsDuplicate`
   - **First "Choose a Value":** choose the lightning bolt, and choose "ExistingTitles"
   - Select 'contains' as the logical operator 
   - **Second "Choose a Value":** choose the lightning bolt, and select the "outputs" from "Extract Title" 
4. In the "True" branch of the condition:
   1. Click on "+" and "Add an Action."
   2. Search for "Variable" and select "Set variable."
   3. Configure the action:
      - **Name:** `DuplicateFound`
      - **Value:** `true`

### 9. Add Condition to Check DuplicateFound and Take Action

1. **Add Condition to Check `DuplicateFound`:**
   1. After the "Apply to each" loop that checks for duplicates, click on "+" and "Add an Action." (make sure this is outside the loop!!)
   2. Search for "Condition" and select it.
   3. Configure the condition:
      - **First "Choose a Value":** Select the `DuplicateFound` variable.
      - **Condition:** `is equal to`
      - **Second "Choose a Value":** `true`

2. In the "False" branch of the condition, click on "+" and "Add an Action."
   1. Search for "Excel" and select "Add a row into a table."
   2. Configure the action to add the new row:
      - **Location:** OneDrive for Business
      - **Document Library:** OneDrive
      - **File:** Select the "literature_bot.xlsx" Excel file.
      - **Table:** Select the table (e.g., "Table1").
      - Select `Show All` under advanced parameters.
      - **Title:** Select the output from the "Compose" action that extracts the title.
      - **Link:** Select the output from the "Compose" action that extracts the link.
      - **Posted:** Enter `FALSE`.

3. In the "True" branch of the condition, click on "+" and "Add an Action."
   2. Search for "Variable" and select "Set variable."
   3. Configure the action:
      - **Name:** `DuplicateFound`
      - **Value:** `false`
