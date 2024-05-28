# Microsoft Power Automate instructions

First, we use PowerAutomate to check each RSS feed periodically. 

Then, we filter out just the new papers, and for bioRxiv those that match our search terms, from each feed and remove any duplicates.

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

3. Set up the list of bioRxiv RSS feeds (we keep these separate because we have to manually search for relevant papers):
   - Add an "Initialize variable" action.
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
4. Set up the list of search terms (we'll only keep bioRxiv papers that have these terms in the title/abstract):
   - Add an "Initialize variable" action.
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

These are my search terms. Obviously you'll (probably...) want different ones.

5. Set up the list of other RSS feeds (ones where you can get things that already match your search terms):
   - Add an "Initialize variable" action.
   - **Name:** `OtherFeedURLs`
   - **Type:** Array
   - **Value:** 
     ```json
     [
       "https://pubmed.ncbi.nlm.nih.gov/rss/search/1nMk785v90DQLMmnCRvxhmwnyIvgek4pzThcVTJIYNlwhOfkkg/?limit=100&utm_campaign=pubmed-2&fc=20240527034611",
       "https://export.arxiv.org/api/query?search_query=all:phylogen*&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending"
     ]
     ```

Note that you can add as many feeds as you like here, as long as everything in those feeds is what you want to post.


### 2. Authenticate with Bluesky API
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

### 3. Get the bioRxiv papers and filter them

We only want papers from the last 24 hours, and that match our search terms. 

1. Add an `Initialize Variable`, call it `AllPapers`, choose `Array` and the value should be `[]` (we'll use this to store all the papers we get)
2. Add an `Initialize Variable`, call it `OldAllPapers`, choose `Array` and the value should be `[]` (we'll use this as a workaround to update our `AllPapers` list one feed at a time; one more reason to hate Power Automate)
3. Add an `Apply to Each` action and call it `LoopThroughFeeds`
4. Add a `Scope` action, and call it `FetchAndFilterFeed` (this is so we don't fall over if one of the bioRxiv feeds doesn't work, which is *often*)
5. Add a `Scope` action, and call it `ErrorHandler`
   - Go to the settings of the "ErrorHandler" scope, and expand the drop down menu
   - Check the boxes for "has failed" and "has timed out", and uncheck the others.
6. Add a `List all RSS Feed Items` action into the `FetchAndFilterFeed` loop
   - Use the lightning bolt to set the `RSS Feed URL` to `Current Item` from `LoopThroughFeeds` (i.e. we're just getting each URL in turn)
7. Add a `Filter Array` action, call `FilterArray`, we'll use this to get papers from the last day only (to avoid duplicates day to day):
   - Use the lightning bolt to set the `From` to `body` from `List all RSS Feed Items`
   - On the left side of the query, use the blue `fx` to add this code: `formatDateTime(item()?['publishDate'], 'yyyy-MM-dd')`
   - On the right side of the query, use the blue `fx` to add this code: `formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')`
   - Set the operator to `is greater or equal to`
     


###########
###########
###########


4. Drag the `FilterArray` into the loop
5. In the `List all RSS Feed Items`, delte the existing `RSS Feed URL`, click the blue `fx` and add this: `items('LoopThroughFeeds')`
6. Now we add this to our list of all papers, ignoring duplicates. Inside the `Apply to each` loop, add a "Set variable" action.
   - **Name:** `AllPapers`
   - **Value:** 
     ```json
     union(variables('OldAllPapers'), body('FilterArray'))
     ```

OK, our `AllPapers` list now contains all the papers from bioRxiv, now we need to select only those we want.







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







### 3. Handle feeds that fail

Since we're checking a lot of feeds, if one fails everything falls over. So we need to be robust to that. Here's how.

1. **Create a Scope for RSS Feed Fetching:**
   - Add a "Scope" action to contain the RSS feed fetching and filtering actions.
   - Name this scope "FetchAndFilterFeed".

2. **Move RSS Fetching and Filtering Actions into the Scope:**
   - Move the "List all RSS Feed Items" and "FilterArray" actions into the "FetchAndFilterFeed" scope. Do this by copy/pasting them and deleting the old ones.

3. **Move the other actions into the Scope:**
   - Move the `UpdateOldAllPapers` and `UpdateAllPapers` actions into the scope. You should be able to drag and drop these.
   
5. **Configure Error Handling Scope:**
   - After the first scope, add another "Scope" action named "ErrorHandler".
   - Inside the "ErrorHandler" scope, add a "Compose" action to log the error.
     - **Name:** `ErrorLog`
     - **Inputs:** `Something went wrong with fetching or filtering the feed.`
   - Add any other actions you want to take in case of an error, such as sending a notification.

6. **Configure the Run After Settings:**
   - Go to the settings of the "ErrorHandler" scope, and expand the drop down menu
   - Check the boxes for "has failed" and "has timed out", and uncheck the others.

### 4. Filter papers by search term

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
          contains(toLower(items('LoopThroughPapers')['title']), toLower(items('LoopThroughKeywords'))),
          contains(toLower(items('LoopThroughPapers')['summary']), toLower(items('LoopThroughKeywords')))
       )
       ```
     - **Operator:** `is equal to`
     - **Right:** `true`

6. If the condition is true, add an "Append to array variable" action.
   - **Name:** `FilteredPapers`
   - **Value:** `items('LoopThroughPapers')`

### 5. Count the papers we need to post

1. Edit the PostCount to have the following code: `length(variables('FilteredPapers'))`


### 3. Edit the Bluesky posting loop

1. Click on the `PostToBluesky` loop, remove the output, and replace it with `FilteredPapers`
2.  
