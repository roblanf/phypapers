# Making twitterbots to post academic papers
##### using the building of https://twitter.com/phy_papers as an example

There are lots of reasons you might want to keep up with the literature, and twitter feeds are a useful way to do it. Casey Bergman set up flypapers (https://twitter.com/fly_papers), which is a twitterbot that tweets links to any new papers on Drosophila. This has proven useful to lots of people. There's no reason we shouldn't all be doing this - nobody has to follow accounts they're not interested in, and there may be accounts (like Fly_papers) that lots of people find useful. Who knows.

So, here are some very detailed instructions which, I hope, should mean that anyone can set up their own twitterbot to tweet papers on any given topic.

These notes are based on Casey Bergman's blog post here:

http://caseybergman.wordpress.com/2014/02/24/keeping-up-with-the-scientific-literature-using-twitterbots-the-flypapers-experiment/

## 1. Set up a twitter account

1. Decide on a name for your twitterbot. For consistency with the flypapers account, so that it's easier for acadmics to spot other 'papers' bots, I suggest we keep the same format: the "Full Name" should be something followed immediately by 'papers', and the username should be the same with an underscore between the two. E.g. 'flypapers' and 'fly_papers'. Or, for this example, 'phypapers' and 'phy_papers'.
2. Go here: https://twitter.com/
3. Follow the links to set up your new account
4. Sign in to your new account, and stay signed in until you're done with these instructions
5. Suggested (but not required): click on the cog symbol at the top right of the twitter account and click 'settings'. Then on the left click 'Email notifications'. Un-tick every single box. If you don't, then you're liable to get flooded by emails every time even the most trivially dull thing happens to your new twitter account. Un-ticking everything really works - twitter won't email you about anything. 
 
Note that you can't use the same email account for >1 account. So you should probably reserve your main email account for your own twitter account, and use a secondary email account to set up your twitterbot account.

## 2. Set up a pubmed search

1. Go here: http://www.ncbi.nlm.nih.gov/pubmed/
2. Type in your favourite search terms (this example: 'phylogen*') 
3. Click 'search'
4. Click the "RSS" link (with the orange symbol) just below the search box
5. Set "Number of items to be displayed" to 100
6. Click "Create RSS"
7. Click the orange "XML" box
8. Record the address in your browser's address bar (this example: http://www.ncbi.nlm.nih.gov/entrez/eutils/erss.cgi?rss_guid=1LU7YBGfC-yDnaYBUcYLivlvzqhgKnWemjak5n2h7PxDtGFoHp)

## 3. Set up your dlvr.it account

1. Go here: http://dlvr.it/
2. Follow the steps to sign up for a new account
3. You will need to confirm your account by email
4. Once your account is confirmed...
5. In the 'Step 1: add your feed', post the RSS feed URL from step 8, above. 
6. Choose the option "Post all items existing in feed now (up to maximum)."
7. Click OK
8. For "Step 2" Click the Twitter button, and authorise dlvr.it to post to your new twitter account.

That's it. Your new twitterbot is running. Give it a bit of time (a day or two perhaps) to catch up with itself before you tell everyone about it - it will take a while to post all the articles it finds on pubmed to start with. This is nice, because then when you do tell people about your project, they will see the kind of papers it posts.

For thoughts on other dlvr.it settings, see below.

## 4. Adding a feed from arxiv preprints

1. Edit this URL to include your search term: 'http://export.arxiv.org/api/query?search_query=all:[YOURSEARCHTERMHERE]&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending', e.g. for this example: 'http://export.arxiv.org/api/query?search_query=all:phylogen*&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending'
2. Log into dlvr.it
3. Under 'Routes' click your twitterbot. It might be called 'my new route' if you didn't rename it. 
4. In the 'Sources' box, click the '+add' rectangle
5. Click the square orange RSS symbol
6. Click 'add feed'
7. Paste the link from step 1 into the 'Feed URL' box
8. Set your other options as you see fit - some ideas are pasted below
9. Click 'Save source'

## 5. Adding a feed from bioRxiv preprints

This is a touch annoying, because you have to add each subject area separately. But it works.

1. For each subject area you might find interesting papers in, do the following (I'll use Evolutionary Biology as an example)
2. Click the link for that subject area on http://biorxiv.org/rss
3. Copy the URL (this example: http://hwmaint.biorxiv.highwire.org/cgi/collection/rss?coll_alias=evolutionary_biology)
4. Follow steps 2 to 8 in the above instructions, pasting the biorxiv link into the URL box
5. Under the 'Filters' tab, add your search terms (this example: 'phylogenetics, phylogenomics' and a few related terms)
6. Click 'save source'

Now just repeat for all subject areas of interest. (But note that you are limited to a total of 5 sources on the free version of dlvr.it)

## 6. Adding a feed from PeerJ PrePrints

This is easier than bioRxiv, but very similar.

1. Copy this link: https://peerj.com/preprints/index.rss2
2. Follow steps 2 to 8 in section 4, above, pasting the above peerj link into the URL box
3. Under the 'Filters' tab, add your search terms (this example: 'phylogenetics, phylogenomics' and a few related terms)
4. Click 'save source'

## A few thoughts

### Making things easier for people to ignore
It might be an idea to put a short prefix onto each different feed. On phy_papers I'm experimenting with adding prefixes before preprints to let people know what servers they're from, e.g. "arxiv" for those from arxiv. This might be helpful because typically arxiv papers are more math oriented than other papers, which will affect peoples interest (in both directions!). To do this, just edit your sources in dlvr.it (hover over the source and click the pencil).

### Update frequency
It's interesting to consider what settings might work best for your sources in dlvr.it: how often to collect data, and how often and how many tweets to post at a time. You don't want to obliterate your followers feeds with new papers, and in general twitter works best with frequent but small posts spread over the day (note that your followers could be in any timezone).

Here's my suggestion. PubMed is going to be your busiest feed, so let's look at that. First, go back to your search in pubmed. If it's a simple search, you'll be able to see a bar chart at top right, telling you how many papers per year you can expect. For 'phylogen*' there were up to about 14K papers each year for the last couple of years. That's up to about 40 papers per day. But there will be some variance in that, I imagine that there are peaks when a new issue of e.g. MPE is published. 

So here's my suggestion:

  Feed update period: every 30 minutes  
  Max items per post: 1  
  Max items per day: 250  
  Trickle: Newest first  

These settings should trickle new papers in to people's without being disruptive, and do so over the course of the day (i.e. hitting all timezones). Using 'Newest first' means that on busy days with more than the 48 maximum number of papers (2*24*1) you'll start to build up a store of papers to trickle out on quieter days. 

If you find any mistakes or omissions in this document, raise an issue on the github page and I'll fix it.

This is free to use and redistribute under a CC0 license.