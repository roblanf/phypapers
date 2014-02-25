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
2. Type in your favourite search terms (this example: 'phylogenetics phylogenomics') 
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

That's it. Your new twitterbot is running.

## 4. Adding a feed from arxiv preprints


1. Edit this URL to include your search term: 'http://export.arxiv.org/api/query?search_query=all:[YOURSEARCHTERMHERE]&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending', e.g. for this example: 'http://export.arxiv.org/api/query?search_query=all:phylogenetics&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending'
2. Log into dlvr.it
3. Under 'Routes' your dlvr.it route which posts to your twitter account will be listed. It might be called 'my new route' if you didn't rename it. Click on that. 
4. In the 'Sources' box, click the '+add' rectangle
5. Click the square orange RSS symbol
6. Click 'add feed'
7. Paste the link from step 1 into the 'Feed URL' box
8. In the dropdown menu, click the "All items" option.
9. Click 'Save source'

## A few thoughts

### Making things easier for people to ignore
It might be an idea to put a short prefix onto each different feed. On phy_papers I'm experimenting with adding prefixes before preprints to let people know what servers they're from, e.g. "arxiv" for those from arxiv. This might be helpful because typically arxiv papers are more math oriented than other papers, which will affect peoples interest (in both directions!). To do this, just edit your sources in dlvr.it (hover over the source and click the pencil).

Also, it's interesting to consider what settings might work best for your sources in dlvr.it: how often to collect data, and how often and how many tweets to post at a time. You don't want to obliterate your followers feeds with new papers, and in general twitter works best with lots of small amount of information spread over the day (note that your followers could be in any timezone). So my best guess is that these settings are sensible:

Feed update period: every 30 minutes
Max items per post: 5
Max items per day: 250 (that's the maximum)
Trickle: Oldest first

These settings should trickle new papers in to people's feeds relatively effectively. One additional consideration - if you have a lot of sources, it would be a good idea to offset their updates.


If you find any mistakes or omissions in this document, raise an issue on the github page and I'll fix it.

This is free to use and redistribute under a CC0 license.