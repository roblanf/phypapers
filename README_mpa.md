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

### 3. Get the bioRxiv papers from the last 24 hours

We only want papers from the last 24 hours 

1. Add an `Initialize Variable`, call it `AllPapers`, choose `Array` and the value should be `[]` (we'll use this to store all the papers we get)
2. Add an `Initialize Variable`, call it `OldAllPapers`, choose `Array` and the value should be `[]` (we'll use this as a workaround to update our `AllPapers` list one feed at a time; one more reason to hate Power Automate)
3. Add an `Apply to Each` action and call it `LoopThroughFeeds`, set the input to the `FeedURLs` variable
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
8. Add a `Set Variable` action, call it `UpdateOldAllPapers`, choose `OldAllPapers` from the dropdown, and use the lightning bolt to set the value to the `AllPapers` variable
9. Add a `Set Variable` action, call it `UpdateAllPapers`, choose `AllPapers` from the dropdown, and use the blue `fx` to enter the following code: union(variables('OldAllPapers'), body('FilterArray')). This adds the new papers from the feed we're working on to our `AllPapers` list (without duplicates).    

### 4. Filter the bioRxiv papers against our search terms
     
1. Directly after the `LoopThroughFeeds` loop, add an "Initialize variable" action.
   - **Name:** `FilteredPapers`
   - **Type:** Array
   - **Value:** `[]`

2. Inside this loop, add another `Apply to each` loop for `AllPapers`.
   - **Name:** `LoopThroughPapers`

3. Add an `Apply to each` loop for the `Keywords` array.
   - **Name:** `LoopThroughKeywords`

4. Inside the `LoopThroughKeywords` loop, add a "Condition" action.
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

5. If the condition is true, add an "Append to array variable" action.
   - **Name:** `FilteredPapers`
   - **Value:** `items('LoopThroughPapers')`

### 5. Get the rest of the RSS feeds

These are the ones with search terms built in, so we don't need to do as much work here.

1. Add an `Initialize Variable`, call it `OldFilteredPapers`, choose `Array` and the value should be `[]`
2. Add an `Apply to Each` action, set it to loop over the `OtherFeedURLs` variable
3. Add a `Scope` action, and call it `FetchAndFilterFeed2` 
5. Add a `Scope` action, and call it `ErrorHandler2`
   - Go to the settings of the "ErrorHandler" scope, and expand the drop down menu
   - Check the boxes for "has failed" and "has timed out", and uncheck the others.
6. Add a `List all RSS Feed Items` action into the `FetchAndFilterFeed` loop, call it `List all RSS Feed Items2`
   - Use the lightning bolt to set the `RSS Feed URL` to `Current Item` from `LoopThroughOtherRSSFeeds` (i.e. we're just getting each URL in turn)
7. Add a `Filter Array` action, call `FilterArray2`, we'll use this to get papers from the last day only (to avoid duplicates day to day):
   - Use the lightning bolt to set the `From` to `body` from `List all RSS Feed Items2`
   - On the left side of the query, use the blue `fx` to add this code: `formatDateTime(item()?['publishDate'], 'yyyy-MM-dd')`
   - On the right side of the query, use the blue `fx` to add this code: `formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')`
   - Set the operator to `is greater or equal to`
8. Add a `Compose` action, call it `RecentPapers`, use the lightning bolt to choose the 'body' of `FilterArray2`
9. Add a `Compose` action, call it `AddNewPapersToFilteredPapers`, use the blue `fx` to enter the following code `union(variables('FilteredPapers'), outputs('RecentPapers'))`
10. Add a `Set Variable` action, call it `SetFilteredPapers`, choose `FilteredPapers` from the dropdown, and use the lightning bolt to choose the `output` of `AddNewPapersToFilteredPapers`. 

### 5. Remove duplicates, and figure out how often to post

1. Add a "Compose" action and call it `RemoveDuplicates`. Use the blue `fx` to enter the following code: `union(variables('FilteredPapers'), variables('FilteredPapers'))`
2. Add a Set Variable Action and call it `Set Filtered Papers`, set the Name to `FilteredPapers`, use the lightning bolt to set the Value to the `output` of `RemoveDuplicates`
3. Add a "Compose" action, call it `PostCount`, and use the blue `fx` to enter teh code `length(variables('FilteredPapers'))`
4. Add a "Compose" action, call it `MinutesBetweenPosts` and use the blue `fx` and enter the code `div(1380, outputs('PostCount'))`. This will allow us to space the posts out over ~23 hours.


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

4. Next, add a "Compose" action to strip HTML tags and newline characters from the title
   - **Name:** `Title`
   - **Inputs:** select the blue `fx` and in the code box put the following code:
```json
replace(
    join(
        xpath(
            xml(
                concat(
                    '<root>',
                    replace(replace(replace(outputs('Title'), '&', '&amp;'), '<', '&lt;'), '>', '&gt;'),
                    '</root>'
                )
            ),
            '//text()'
        ),
        ''
    ),
    '\n',
    ''
)
```
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
               "byteStart": @{sub(outputs('GetPostLength`, outputs('GetLinkLength')))},
               "byteEnd": @{length(outputs('GetPostLength`))}
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









### 5. Count the papers we need to post

1. Edit the PostCount to have the following code: `length(variables('FilteredPapers'))`


### 3. Edit the Bluesky posting loop

1. Click on the `PostToBluesky` loop, remove the output, and replace it with `FilteredPapers`
2.  
