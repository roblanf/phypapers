# Microsoft Power Automate instructions

So, dlvr.it has massively constrained the free option. In a hunt for another free option, I thought that Microsoft Power Automate may suffice. Many academics will have access to Microsoft Power Automate through an institutional subscription to Office365, so here's a set of instructions for getting going.

They're a little complicated, but the principle is simple. 

First, we use PowerAutomate to check each RSS feed periodically. 

Then, we filter out just the new papers from each feed.

Finally, we post the new papers over the next time period, evenly spaced.

# Set up your Power Automate Flow

> NB I hate power automate, and you may come to hate it too. It's like programming without access to anything useful. It's worse than the lego drag and drop programming thing that my kids and I use on the iPad. 

NEVERHTELESS Here we're going to make a 'Flow'. Take a deep breath...

### 1. Create a Scheduled Flow
1. Go to [Power Automate](https://flow.microsoft.com).
2. Click on "Create" > "Scheduled cloud flow."
3. Name your flow "literature_bot_phypapers" or whatever the hell you like
4. Set it to run every Day and click "Create."

> NB: Pubmed gets updated once every 24 hours, and the rest of this flow assumes you only check it once every 24 hours. If you check it more often you'll get a lot of duplicate posts.

### 2. Add RSS Action to Fetch RSS Feed
1. In the flow, click on "+" and "Add an Action"
2. Search for "RSS" and select "List all feed items."
3. Configure the action:
   - **Feed URL:** Enter the RSS feed URL, e.g. my pubmed search is `https://pubmed.ncbi.nlm.nih.gov/rss/search/1pSbSzklLaRDgrBBecLaHXjj_NtDB256CbB-lTk3MQA9gZRkc4/?limit=100&utm_campaign=pubmed-2&fc=20240525000654`

### 3. Only keep papers added to the feed in the last 24 hours
1. In the flow, click on "+" and "Add an Action."
2. Search for "Filter array" and select it.
3. Configure the action:
   - **Name:** Call it `FilterArray`
   - **From:** Select `body` from the "List all feed items" action.
   - **Condition:**
      - In the left box, click the `fx` and paste this into the text box: `formatDateTime(item()?['publishDate'], 'yyyy-MM-dd')`
      - Choose `is greater or equal to` for the operator.
      - In the right box, click the `fx` and paste this into the text box: `formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')`

OK, now we have our list of new things, and we can go ahead and post to Bluesky.

### 4. Figure out how frequently we should post

Let's aim to post everything we've got within 23 hours.

1. Add a "Compose" action after filtering the RSS feed.
   - **Name:** `PostCount`
   - **Inputs:** click the blue `fx` and enter this in the text box: `length(body('FilterArray'))`
2. Add a "Compose" action after getting the post count.
   - **Name:** `MinutesBetweenPosts`
   - **Inputs:** click the blue `fx` and enter this in the text box: `div(1380, outputs('PostCount'))`

This will allow us to trickle out our posts over a ~23 hour period.

### 5. Initialize Variables for Bluesky Credentials
1. Add a "Initialize variable" action.
   - **Name:** `BlueskyUsername` (e.g. `phypapers.bsky.social`)
   - **Type:** String
   - **Value:** Enter your Bluesky username.

2. Add another "Initialize variable" action.
   - **Name:** `BlueskyAPIPassword` (should be something with alphanumeric characters in the form `xxxx-xxxx-xxxx-xxxx`)
   - **Type:** String
   - **Value:** Enter your Bluesky API password.

### 6. Loop Through All Papers
1. Add an "Apply to each" action.
   - **Value:** use the lightning bolt to select the FilterArray `body`
   - **Name:** `PostToBluesky`

2. Inside the "Apply to each" action, add a "Compose" action.
   - **Name:** `CurrentPaper`
   - **Inputs:** use the lightning bolt to select the PostToBluesky `Current Item`

3. Next, add a "Compose" action to get the title.
   - **Name:** `Title`
   - **Inputs:** select the blue `fx` and in the code box put `item()?['title']`

4. Next, add a "Compose" action to strip HTML tags from the title
   - **Name:** `Title`
   - **Inputs:** select the blue `fx` and in the code box put `join(xpath(xml(concat('<root>', outputs('Title'), '</root>')), '//text()'), '')`

5. Next, add a "Compose" action to truncate the title if it's longer than 260 characters
   - **Name:** `Title`
   - **Inputs:** select the blue `fx` and in the code box put `if(greater(length(outputs('CleanTitle')), 260), substring(outputs('CleanTitle'), 0, 260), outputs('CleanTitle'))`

6. Next, add a "Compose" action to get the link.
   - **Name:** `Link`
   - **Inputs:** select the blue `fx` and in the code box put `item()?['primaryLink']`

7. Next, add a "Compose" action to take the crud off the link.
   - **Name:** `CleanLink`
   - **Inputs:** select the blue `fx` and in the code box put `split(outputs('Link'), '?')[0]`


### 7. Build the Post Content
1. Inside the "PostToBluesky" loop, add a "Compose" action after the `ShortTitle` and `CleanLink` actions.
   - **Name:** `PostContent`
   - **Inputs:**
     ```json
     "@{concat(outputs('ShortTitle'),' ',outputs('CleanLink'))}"
     ```

### 8. Authenticate with Bluesky API
1. Inside the "PostToBluesky" loop, add an "HTTP" action.
   - **Method:** POST
   - **URI:** `https://bsky.social/xrpc/com.atproto.server.createSession`
   - **Headers:** 
     - Content-Type: application/json
   - **Body:** 
     ```json
     {
       "identifier": "@{variables('BlueskyUsername')}",
       "password": "@{variables('BlueskyAPIPassword')}"
     }
     ```
