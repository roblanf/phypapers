# Microsoft Power Automate instructions

So, dlvr.it has massively constrained the free option. In a hunt for another free option, I thought that Microsoft Power Automate may suffice. Many academics will have access to Microsoft Power Automate through an institutional subscription to Office365, so here's a set of instructions for getting going.

They're a little complicated, but the principle is simple. 

First, we use PowerAutomate to check each RSS feed periodically. 

Then, we filter out just the new papers from each feed.

Finally, we post the new papers over the next time period, evenly spaced.

# Set up your Power Automate Flow for PubMed

> NB I hate power automate, and you may come to hate it too. It's like programming without access to anything useful. It's worse than the lego drag and drop programming thing that my kids and I use on the iPad. 

NEVERHTELESS Here we're going to make a 'Flow'. Take a deep breath...

### 1. Create a Scheduled Flow
1. Go to [Power Automate](https://flow.microsoft.com).
2. Click on "Create" > "Scheduled cloud flow."
3. Name your flow "literature_bot_phypapers" or whatever the hell you like
4. Set it to run every Day and click "Create."

> NB: Pubmed gets updated once every 24 hours, and the rest of this flow assumes you only check it once every 24 hours. If you check it more often you'll get a lot of duplicate posts.

### 2. Initialise your variables

We'll set the variables that different people will want to change right at the top. This will make it easier to adapt this to different RSS feeds and/or people.

1. Click on "+" and "Add an Action", then search for the "Initialize variable" action and select it. Set it up as follows:
   - **Name:** `BlueskyUsername` (e.g. `phypapers.bsky.social`)
   - **Type:** String
   - **Value:** Enter your Bluesky username.

2. Add another "Initialize variable" action and set it up as follows:
   - **Name:** `BlueskyAPIPassword` (should be something with alphanumeric characters in the form `xxxx-xxxx-xxxx-xxxx`)
   - **Type:** String
   - **Value:** Enter your Bluesky API password.

2. Add another "Initialize variable" action and set it up as follows:
   - **Name:** `RssURL`
   - **Type:** String
   - **Value:** Enter the URL for your RSS feed (e.g. mine is `https://pubmed.ncbi.nlm.nih.gov/rss/search/1pSbSzklLaRDgrBBecLaHXjj_NtDB256CbB-lTk3MQA9gZRkc4/?limit=100&utm_campaign=pubmed-2&fc=20240525000654`)


### 3. Add RSS Action to Fetch RSS Feed
1. Click on "+" and "Add an Action"
2. Search for "RSS" and select "List all feed items."
3. Configure the action:
   - **Feed URL:** click the lightning bolt and select the variable `RssUrl` which you set earlier

### 4. Only keep papers added to the feed in the last 24 hours
1. Click on "+" and "Add an Action"
2. Search for "Filter array" and select it.
3. Configure the action:
   - **Name:** Call it `FilterArray`
   - **From:** Select `body` from the "List all feed items" action.
   - **Condition:**
      - In the left box, click the `fx` and paste this into the text box: `formatDateTime(item()?['publishDate'], 'yyyy-MM-dd')`
      - Choose `is greater or equal to` for the operator.
      - In the right box, click the `fx` and paste this into the text box: `formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')`

This keeps only the papers in the RSS feed that have been added to it in the last 24 hours, which stops us double posting (the feed will always have 100 items, but not all of them will necessarily be new each day).

### 5. Figure out how frequently we should post

Let's aim to post everything we've got within 23 hours.

1. Add a "Compose" action after filtering the RSS feed.
   - **Name:** `PostCount`
   - **Inputs:** click the blue `fx` and enter this in the text box: `length(body('FilterArray'))`
2. Add a "Compose" action after getting the post count.
   - **Name:** `MinutesBetweenPosts`
   - **Inputs:** click the blue `fx` and enter this in the text box: `div(1380, outputs('PostCount'))`

This will allow us to trickle out our posts over a ~23 hour period.


### 6. Authenticate with Bluesky API
1. Add an "HTTP" action and call it `GetAccessToken`
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

2. Add a "Parse JSON" action.
   - **Content:** click the lightning bolt and choose `body` of `GetAccessToken`
   - **Schema:**
     ```json
     {
       "type": "object",
       "properties": {
         "accessJwt": { "type": "string" },
         "refreshJwt": { "type": "string" }
       }
     }
     ```

3. Add an "Initialize variable" action and call it `AccessToken`
   - **Type:** String
   - **Value:** use the lightning bolt and select `Body acessJWT` from Parse JSON

4. Add an "Initialize variable" action and call it `RefreshToken`
   - **Type:** String
   - **Value:** use the lightning bolt and select `Body refreshJWT` from Parse JSON

We need these tokens later to post to Bluesky

### 7. Loop Through All Papers, extract the basics of each paper
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


### 8. Make the text for the post
1. Inside the "PostToBluesky" loop, add a "Compose" action after the `ShortTitle` and `CleanLink` actions.
   - **Name:** `PostContent`
   - **Inputs:**
     ```json
     "@{concat(outputs('ShortTitle'),' ',outputs('CleanLink'))}"
     ```

### 9. Refresh Access Token in Each Loop Iteration

Access tokens don't last for long, so we need to refresh it each time we post

1. Inside the "PostToBluesky" loop, add an "HTTP" action.
   - **Name:** `RefreshAccessToken`
   - **Method:** POST
   - **URI:** `https://public.api.bsky.app/xrpc/com.atproto.server.refreshSession`
   - **Headers:**
     - Accept: application/json
     - Authorization: `Bearer @{variables('RefreshToken')}`

2. Add a "Parse JSON" action.
   - **Name:** `ParseRefreshResponse`
   - **Content:** click the lightning bolt and choose `body` of `RefreshAccessToken`
   - **Schema:**
     ```json
     {
       "type": "object",
       "properties": {
         "accessJwt": { "type": "string" }
       }
     }
     ```

### 10. Post to Bluesky

1. Inside the "PostToBluesky" loop, add an "HTTP" action after the `PostContent` action.
   - **Name:** `PostToBlueskyAPI`
   - **Method:** POST
   - **URI:** `https://bsky.social/xrpc/com.atproto.repo.createRecord`
   - **Headers:**
     - Content-Type: application/json
     - Authorization: `Bearer @{variables('AccessToken')}`
   - **Body:**
     ```json
     {
       "collection": "app.bsky.feed.post",
       "repo": "@{variables('BlueskyUsername')}",
       "record": {
         "$type": "app.bsky.feed.post",
         "text": "@{outputs('PostContent')}",
         "facets": [
           {
             "index": {
               "byteStart": @{add(length(outputs('ShortTitle')), 1)},
               "byteEnd": @{length(outputs('PostContent'))}
             },
             "uri": "@{outputs('CleanLink')}"
           }
         ],
         "createdAt": "@{utcNow()}"
       }
     }
     ```
3. Add a "Set variable" action to update the access token.
   - **Name:** `AccessToken`
   - **Value:** click the lightning bolt and choose `body accessJWT` of `ParseRefreshResponse` 

### 11. Wait a bit until you post again

1. Add a 'Delay` action
2. Select the blue lightning bolt and choose the `Outputs` of the `MinutesBetweenPosts` variable

This will make the bot wait, so the papers trickle out over ~23 hours.

> If you thought these instructions were long and tiresome, I cannot tell you how much longer and tiresomer they were to figure out!

# Set up your Power Automate Flow for BioRxiv

This should be easier. 

First, save a copy of your first Flow, and call it something different. 

Now let's edit what we need to. The BioRxiv feed pushes only 30 papers at a time, and you can't get papers that match your search. So we'll need to do it a little bit differently - checking every subject feed once a day, removing duplicates, only posting papers with the words we want that appeared in the last day. 

Here are the steps:

### 1. Initialize a List of All Feeds

1. Remove the `RssUrl` variable, and in it's place...
2. Add an "Initialize variable" action.
   - **Name:** `FeedURLs`
   - **Type:** Array
   - **Value:** 
     ```json
     [
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=animal_behavior",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=biochemistry",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=bioinformatics",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=biophysics",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=cancer_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=cell_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=developmental_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=ecology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=evolutionary_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=genetics",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=genomics",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=immunology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=microbiology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=molecular_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=neuroscience",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=paleontology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=pathology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=pharmacology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=physiology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=plant_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=scientific_communication_and_education",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=synthetic_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=systems_biology",
       "http://connect.biorxiv.org/biorxiv_xml.php?subject=zoology"
     ]
     ```

### 2. Get them all

1. Above the `List all RSS Feed Items`, add an `Initialize Variable`, call it `AllPapers`, choose `Array` and the value should be `[]` (we'll use this to store all the papers we get)
2. Above the `List all RSS Feed Items`, add an `Apply to Each` and call it `LoopThroughFeeds`
3. Drag the `List all RSS Feed Items` into the loop
4. Drag the `FilterArray` into the loop
5. In the `List all RSS Feed Items`, delte the existing `RSS Feed URL`, click the blue `fx` and add this: `items('LoopThroughFeeds')`
6. Now we add this to our list of all papers, ignoring duplicates. Inside the `Apply to each` loop, add a "Set variable" action.
   - **Name:** `AllPapers`
   - **Value:** 
     ```json
     union(variables('OldAllPapers'), body('FilterArray'))
     ```

OK, our `AllPapers` list now contains all the papers from bioRxiv, now we need to select only those we want.

### 3. Filter papers by search term

We're going to add our list of search terms at the top of the script where it's easy to edit. Then filter the bioRxiv papers lower down.

1. Add an "Initialize variable" action at the top of the script, I put mine just under the list of URLs.
   - **Name:** `Keywords`
   - **Type:** Array
   - **Value:**
     ```json
     [
       "phylogenetics", 
       "phylogenomics", 
       "phylogenetic analysis", 
       "phylogenomic analysis" 
     ]
     ```
     
2. Directly after the `LoopThroughFeeds` loop, add an "Initialize variable" action.
   - **Name:** `FilteredPapers`
   - **Type:** Array
   - **Value:** `[]`

3. Inside this loop, add another `Apply to each` loop for `AllPapers`.
   - **Name:** `LoopThroughPapers`

4. Add an `Apply to each` loop for the `Keywords` array.
   - **Name:** `LoopThroughKeywords`

5. Inside the `LoopThroughKeywords` loop, add a "Condition" action.
   - **Condition:**
     - **Left:** (use the blue `fx` to paste this into the box)
       ```json
       or(
          contains(items('LoopThroughPapers')['title'], items('LoopThroughKeywords')), 
          contains(items('LoopThroughPapers')['summary'], items('LoopThroughKeywords'))
       )
       ```

6. If the condition is true, add an "Append to array variable" action.
   - **Name:** `FilteredPapers`
   - **Value:** `items('LoopThroughPapers')`
