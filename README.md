# Build a literature bot in three steps
These instructions tell you how to set up a literature bot that automatically posts papers on particular topics to Bluesky. 

We'll use the building of [https://bsky.app/profile/phypapers.bsky.social](the phypapers bot on Bluesky) as an example. It takes about 20 minutes.

Literature bots can be a useful way to keep up with the latest research. Casey Bergman started it all with a Drosophila literature bot called flypapers back when Twitter existed. There are now hundreds similar ones, many built with the instructions below and many of which can be found on this list: https://twitter.com/caseybergman/lists/literaturebots/members. The detailed instructions here were originally inspired by [Casey Bergman's blog post](http://caseybergman.wordpress.com/2014/02/24/keeping-up-with-the-scientific-literature-using-twitterbots-the-flypapers-experiment/). 

This repo has some very detailed instructions to make your own literature bot. 

> This is a new set of instructions which I've cleaned up substantially, and streamlined for BlueSky. If you're looking for the old instructions, which had notes for Twitter, Tumblr, and things for using Microsoft Flow etc. you can find them [here](https://github.com/roblanf/phypapers/tree/v1-twitter)

## The three basic steps

### 1. Set up a Bluesky account

Obviously you need an account to post to. This part gets you set up on Bluesky, whether you have an existing personal account or not.

1. If you don't have a Bluesky account: Go to https://bsky.app/, and click 'Sign Up'
2. If you do have a Bluesky account, log in then go to `Settings` and click `Add Account`
3. Leave the hosting provider as Bluesky Social
4. Fill out your details to set up your new account:
    * Protip: If you already use your gmail address for your account, you can just append to it to create a new account. E.g. if your personal account is porcelain.crab@gmail.com, you could use porcelain.crab+phypapers@gmail.com (the '+' stays). This helps keep mail separate.
5. Decide on a handle. Following flypapers' lead, I suggest a short prefix followed immediately by `papers`, e.g. `flypapers`, `phypapers`. This means we all know it when we see a literature bot.
6. Click on your new profile, go to `Edit Profile`
    * `Username`: I suggest making this `prefix_papers` e.g. `fly_papers` or `phy_papers`. As above, this helps everyone know what's a literature bot
    * `Description`: pretty obvious, but it's always nice to know the human who runs it, so good to put your name there if you want to
7. Go to `Settings` and scroll down to `App Passwords` and set one up (this is something which allows another service to post to Bluesky on your behalf)

### 2. Set up your RSS feeds

Literature bots use [RSS feeds](https://en.wikipedia.org/wiki/RSS) to post papers. The service we use below (dlvr.it) allows three free RSS feeds. You can use any RSS feed you like, but for the purposes of this tutorial I'll show you how to do the three big ones for what I do: pubmed, arxiv, and biorxiv. The general rule though is that you should establish up to three RSS feeds that cover as much of the literature as you possibly can for whatever your literature bot is for. 

#### 2.1 Pubmed RSS feed

1. Go here: [http://www.ncbi.nlm.nih.gov/pubmed/](http://www.ncbi.nlm.nih.gov/pubmed/)
2. Type in your favourite search terms remembering that wildcards are useful (e.g. `phylogen*` will match anything starting with `phylogen`, and logical operators can be really good, e.g. you can have `phylogen* OR raxml OR splitstree`.
3. Click `search`
4. Click the `Create RSS` link just below the search box
5. Set "Number of items to be displayed" to 100
6. Click `Create RSS`
7. Record the RSS URL somewhere

> NB I recommend leaving the name of the RSS feed for this as-is. This is because it's the only way I know of to look up in the future what the search terms actually were, which is crucial for tweaking your RSS feeds if they are not quite doing what you want


#### 2.2 arXiv preprints

arXiv has preprints for many subjects, so it's worth considering. Setting up the RSS feed is trivial:

1. Edit this URL to include your search term: 'http://export.arxiv.org/api/query?search_query=all:[YOURSEARCHTERMHERE]&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending', e.g. for this example: 'http://export.arxiv.org/api/query?search_query=all:phylogen*&start=0&max_results=10&sortBy=lastUpdatedDate&sortOrder=descending'

#### 2.3 bioRxiv preprints

bioRxiv is also great for biology preprints. I don't know of a way to get an RSS feed with search terms in bioRxiv, but we can get ALL the preprints in an RSS, and filter them later using search terms in dlvr.it. So for bioRxiv we just use the following URL:

[http://connect.biorxiv.org/biorxiv_xml.php?subject=all
](http://connect.biorxiv.org/biorxiv_xml.php?subject=all
)

### 3. Set up dlvr.it

#### 3.1 Get started

First you need to set up your account and link your first RSS to you Bluesky account

1. Go here: [http://dlvr.it/](http://dlvr.it/)
2. Follow the steps to sign up for a new account (NB: the same gmail trick that works for twitter works here too...)
3. CLick the big magnifying glass which says `Feed Search` and paste the RSS feed URL for Pubmed (from step 7 in the pubmed section, above) into the box
4. Your pubmed feed should appear below the search box, now click the `Connect` box to the right of it
5. Click the `Post Settings` box when it goes green (you need to wait a few seconds)
6. Leave the settings as-is for now, and click the green `Connect Socials` box
7. Select the 'Connect New' and click the Bluesky butterly icon and follow the steps authorise dlvr.it to post to your Bluesky account (you'll need to put in the App password that you set up above)
6. Click 'Done'

#### 3.2 Add your other RSS feeds

Now you can add up to two more RSS feeds. We'll add the arXiv and BioRxiv feeds from above.

1. Go to you dlvr.it homepage, which should be [https://app.dlvrit.com/](https://app.dlvrit.com/)
2. For each of the RSS feeds you want to add:
   * Click your literature bot (mine is called `phy_papers`)
   * Click the `Add Feeds` link (above the current feed and on the right)
   * Follow the steps

#### 3.3 Tweak the settings for your feeds

Now we want to make sure our feeds behave nicely! This just requires settings a few options for each of them. 

Remmeber to repeat these steps for all of your feeds (by this point you might have three, or more if you are paying for dlvr.it)

1. In your dlvr.it homepage, click your new route (mine's called `phy_papers`)
2. In the `Feeds` box, click on the cog symbol next to your pubmed feed
3. Click `Updates` tab, my suggestions for settings are:
   * Feed update period: 3 hours (that's the quickest you can get for free)
   * Max items per update period: 10 (this is so you don't flood people's feeds with too many papers at once)
   * Max items per day: 250 (that's the maximum for a free account)
   * Trickle: Newest items first
   * Subscribe to PuSH Updates: off (this stops it posting big PubMed logos with every post)
4. Click the `Advanced` tab, my suggestions for settings are the defaults except:
   * Enable photo posting: Off
5. If you want to add a prefix (or suffix) to each post, you can do that on the `Item Text` tab. For example I like to add the prefix `preprint:` to papers from preprint servers. 
6. If you have a feed (like bioRxiv) where you couldn't filter the content by search terms, then click the `Filters` tab and add your search terms here. This will keep only the relevant papers from these kinds of feeds. I usually choose all parts of the message, and use the 'any of the terms' box to enter a comma-separated list.
7. Click 'Save' in the bottom right

> Remember to repeat this for all of your feeds!

#### 3.4 Tweak the settings for your posting

Now you can tweak how dlvr.it makes your feeds look on Bluesky. To do this:

1. In your dlvr.it homepage, click your new route (mine's called `phy_papers`)
2. Click the cog to the right of your Bluesky account (the bottom box called something like `Sharing to 1 Social`)
3. My suggestions for settings are:
  * Post Style: Status Update (makes is clean plain text)
  * What to post: I usually just choose the title and the URL. This keeps it short and sweet, and allows for rapid scrolling through papers
4. Slick `Save` at the bottom

### 4. Tweak, revise, repeat

That's it. My best advice now is to make sure you follow and check your own feed. If it seems like it's posting rubbish, go tweak the search terms. For example, I noticed that the search above, using 'phylogen*' in pubmed gathers all sorts of papers that just happen to have estimated a phylogenetic tree. That's not what I want, since the focus of most of those papers is often nothing to do with phylogenetics. So I revised my searches to be more specific. I now use this:

`phylogenetics OR phylogenomics OR "phylogenetic analysis" OR "phylogenomic analysis" OR "phylogenetic analyses" OR "phylogenomic analyses" OR "phylogenetic methods" OR "phylogenomic methods" OR phyloinformatics OR "phyloinformatic analysis" OR "phyloinformatic analyses" OR "phyloinformatic methods" OR "phyloinformatic methods"`

Long and unwieldy, but more precisely targeted to my interests, and less likely to fill mine and other people's streams with content we're not interested in.

If you find any mistakes or omissions in this document, raise an issue on the github page and I'll fix it.

