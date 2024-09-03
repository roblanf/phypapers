# Build a literature bot in three steps

These instructions tell you how to set up a literature bot that automatically posts papers on whatever you're intersted in to Bluesky. If you use them to build a bot, please post a note in the ["Show and tell" of the discussions](https://github.com/roblanf/phypapers/discussions/new?category=show-and-tell). 

> A key pre-requisite is that you'll need an account with Microsoft Power Automate. Many people have that for free through an institutional Office365 subscription. To see if you have a subscription, go to: https://make.powerautomate.com/, and try to log in. Microsoft Power Automate is a horrendous way to build *anything*, but the massive advantage for literature bots is that if you have a subscription, you get free server time. So once your literature bot is running, you're all done.

Literature bots can be a useful way to keep up with the latest research. Casey Bergman started it all with a _Drosophila_ literature bot called flypapers back when Twitter existed. There are now hundreds similar ones, many of which can be found on this list: https://twitter.com/caseybergman/lists/literaturebots/members. The detailed instructions here were originally inspired by [Casey Bergman's blog post](http://caseybergman.wordpress.com/2014/02/24/keeping-up-with-the-scientific-literature-using-twitterbots-the-flypapers-experiment/). 

This repo tells you how to build your own literature bot, using [the phypapers bot on Bluesky](https://bsky.app/profile/phypapers.bsky.social) as an example.

> This is a new set of instructions which I've cleaned up substantially, and streamlined for BlueSky. If you're looking for the old instructions, which had notes for Twitter, you can find them [here](https://github.com/roblanf/phypapers/tree/v1-twitter)

## Examples

If you just want to see what this is all about, here are some examples I know of.

| Subject | Name | Account |
|---------|------|---------|
| Phylogenetics | phy_papers | [@phypapers.bsky.social](https://bsky.app/profile/phypapers.bsky.social) | 
| Biogeography | biogeo_papers | [@biogeopapers.bsky.social](https://bsky.app/profile/biogeopapers.bsky.social) | 

## Overview

Here's an overview of what the literature bot does:

1. Runs once every 24 hours
2. Checks a lot of RSS feeds for papers that have come out in the last 24 hours
3. Makes sure the papers are relevant (i.e. some places let you _generate_ relevant RSS feeds, but others don't)
4. Removes duplicate papers
5. Posts the papers to Bluesky account over a 23 hour period, evenly spaced (this is so I don't flood everyone's feed with 100 papers at once)

## The three basic steps to setting it up

### 1. Set up a Bluesky account

Obviously you need an account to post to. This part gets you set up on Bluesky, whether you have an existing personal account or not.

1. If you don't have a Bluesky account: Go to https://bsky.app/, and click 'Sign Up'
2. If you do have a Bluesky account, log in then go to `Settings` and click `Add Account`
3. Leave the hosting provider as Bluesky Social
4. Fill out your details to set up your new account:
    * Protip: If you already use your gmail address for your account, you can just append to it to create a new account. E.g. if your personal account is porcelain.crab@gmail.com, you could use porcelain.crab+phypapers@gmail.com (the '+' stays). This helps keep mail separate.
5. Decide on a handle. Following flypapers' lead, I suggest a short prefix followed immediately by `papers`, e.g. `flypapers`, `phypapers`, etc. This means we all know it when we see a literature bot.
6. Click on your new profile, go to `Edit Profile`
    * `Username`: I suggest making this `prefix_papers` e.g. `fly_papers` or `phy_papers`. As above, this helps everyone know what's a literature bot
    * `Description`: pretty obvious, but it's always nice to know the human who runs it, so good to put your name there if you want to. It would be great if you could also put a link to these instructions on your literature bot - that way anyone who sees yours can also make their own. On my profile I just wrote: "Make your own literature bot with these instructions: https://github.com/roblanf/phypapers"
7. Go to `Settings` and scroll down to `App Passwords` and set one up (this is something which allows another service to post to Bluesky on your behalf)

### 2. Set up your RSS feeds

Literature bots use [RSS feeds](https://en.wikipedia.org/wiki/RSS) to post papers. You can use any RSS feed you like, but for the purposes of this tutorial I'll show you how to do the three big ones for what I do: pubmed, arXiv, and bioRxiv. The general rule though is that you should establish RSS feeds that cover as much of the literature as you possibly can for whatever your literature bot is for. 

#### 2.1 Pubmed RSS feeds

1. Go here: [http://www.ncbi.nlm.nih.gov/pubmed/](http://www.ncbi.nlm.nih.gov/pubmed/)
2. Type in your favourite search terms remembering that wildcards are useful (e.g. `phylogen*` will match anything starting with `phylogen`, and logical operators can be really good, e.g. you can have `phylogen* OR raxml OR splitstree`.
3. Click `search`
4. Click the `Create RSS` link just below the search box
5. Name is something sensible
6. Set `Number of items to be displayed` to 100
7. Click `Create RSS`
8. Record the RSS URL somewhere

Create as many RSS feeds as you like from PubMed, and note them down. Why would you create more than one? Because each feed is limited to retreiving 100 papers, so on the off chance that there's a bumper day for your subject (like a special issue coming out), you might miss papers by having one general feed. For [phypapers](https://bsky.app/profile/phypapers.bsky.social) I went totally overboard and made the following list of RSS feeds:

1. [phylogenetics[Title]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1tYbWOIP0tIVreX9rPCvdGmmbxHJobuBntOy3VyMFivsPJcEG1/?limit=100&utm_campaign=pubmed-2&fc=20240528181849)
2. [phylogenomics[Title]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1bUrbZONKdKY6mFb4tOeokyXplUngAStuFKAcG88ZfRCNqFE5a/?limit=100&utm_campaign=pubmed-2&fc=20240528181921)
3. [phylogenomic[Title]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1T5FW5K6kI71ia_6eneQzMtEXpGBLaOr06kN1qxSU80qPUWQcW/?limit=100&utm_campaign=pubmed-2&fc=20240528182102)
4. [phylogenetic[Title]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1-ONS2P_EKb8HyuP5cSNsVIPVmKKl4rbk16StHDuvXiZWQv9Em/?limit=100&utm_campaign=pubmed-2&fc=20240528182114)
5. [phylogenetics[TitleAbstract]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1bAXfGTh08tVkaeuklkzsn7cdc7iJJPE6uvrK1L3guOpfhwkF_/?limit=100&utm_campaign=pubmed-2&fc=20240528182223)
6. [phylogenomics[TitleAbstract]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1TyHVUJDxNJTq_goUvgFwCZllkgW6UrIpAskwDT-8mQJ3bn9cD/?limit=100&utm_campaign=pubmed-2&fc=20240528182242)
7. [phylogenomic analysis[TitleAbstract]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1NwSQ1kPYoZ_BGXTxnE9MqKYXgYR6mL9HahsJL_YZ-77lpmspk/?limit=100&utm_campaign=pubmed-2&fc=20240528182259)
8. [phylogenetic analysis[TitleAbstract]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1DSoZAVEXfx-7I2bn7qqJUrjCjP9uo4KuCG4G0VbH3DyAAL9Su/?limit=100&utm_campaign=pubmed-2&fc=20240528182329)
9. [ancestral recombination graph[TitleAbstract]](https://pubmed.ncbi.nlm.nih.gov/rss/search/1bYz7DSbRS5oPC2jrkUeb9exioZTLpMlGljCvk088lBI7qagvL/?limit=100&utm_campaign=pubmed-2&fc=20240528182432)

If you click on them you can see what each one entails (the search terms are near the top). And note that it doesn't matter that these will pick up a lot of duplicate papers.

#### 2.2 arXiv preprints

arXiv has preprints for many subjects, so it's worth considering. Setting up the RSS feed is trivial. All you need to do is edit this URL to include your search term: 

```http://export.arxiv.org/api/query?search_query=all:[YOURSEARCHTERMHERE]&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending```

For [phypapers](https://bsky.app/profile/phypapers.bsky.social) I used two RSS feeds. I include the text of the links below so you can see how they're built

1. [phylogen*](https://export.arxiv.org/api/query?search_query=all:phylogen*&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending)
   * `https://export.arxiv.org/api/query?search_query=all:phylogen*&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending`
2. ["ancestral recombination graph"](https://export.arxiv.org/api/query?search_query=all:%22ancestral%20recombination%20graph%22&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending)
   * `https://export.arxiv.org/api/query?search_query=all:%22ancestral%20recombination%20graph%22&start=0&max_results=100&sortBy=lastUpdatedDate&sortOrder=descending`

#### 2.3 RSS feeds which need searching

bioRxiv and EcoEvoRxiv are great for biology preprints. I don't know of a way to get RSS feed with search terms from them though. However, we can get ALL the preprints in an RSS, and filter them using search terms. For bioRxiv you have two options. You can do the simple thing and just get the single RSS feed with all recent paper:

[http://connect.biorxiv.org/biorxiv_xml.php?subject=all
](http://connect.biorxiv.org/biorxiv_xml.php?subject=all
)

OR... you can go overboard like me and get each subject category indpendently. This is probably overkill, but since bioRxiv returns only the last 30 papers, it will help avoid missing anything. Here's the full list in the format you'll need for Microsoft Flow.

```
[
  "https://ecoevorxiv.org/rss/preprints/",
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

Once you've decided on your list of RSS feeds, you then need some search terms - these will be used to find papers with any matches in the title or abstract. For [phypapers](https://bsky.app/profile/phypapers.bsky.social) I use these:

```
[
  "phylogenetic",
  "phylogenomic",
  "ancestral recombination graph"
]
```


### 3. Set up Power Automate

#### 3.1 Set up your variables


First we have to upload the template, which will do all the posting for you:

1. Download the zip file in this repo called `bluesky_literature_bot.zip`, but don't unzip it.
2. Log into https://make.powerautomate.com/
3. On the left hand navigation bar, click "My Flows"
4. Up the top, click 'Import', then 'Import Package (Legacy)'
5. Upload the `bluesky_literature_bot.zip` file, then click 'Create as new', then the blue 'Save' button
6. If there are other rows in the "Review Package Content" table beyond the `bluesky_literature_bot`, you have to review those too. For each one, just click the link under the `import setup` column, then follow the instructions and click the blue 'Save' button.
7. When you've done every row, the 'Import' button at the bottom will turn blue. Click it. Wait for your bot to import.

Next you just have to edit a few of the variables at the top of the template:

1. Click `My Flows`
2. Click `bluesky_literature_bot`, then `Edit` at the top left
3. Click the purple variable `RssFeedsThatNeedKeywordSearch`, and edit it to include any RSS feeds which are not pre-searched (by default it has all of bioRxiv and EcoEvoRxiv, but you can change this to anything). Note the format is JSON, as in step 2.3 above.
4. Click the purple `OtherRssFeeds`, and add in all your PubMed and arXiv RSS feeds. I've left a couple in there so you can see the format, but you will need to delete these and replace them (unless you want to mostly duplicate phypapers).
5. Click on the purple `Keywords` variable, and put in your search terms. As before, I've left mine there so you can see them. (Hint, don't include TOO many - Microsoft Power Automate is terrifyingly slow, I'd say 10 maximum)
6. Click on the purple `BlueskyUsername` variable and change the bottom part from `YOUR_USERNAME.bsky.social` to your username, e.g. `phypapers.bsky.social` is phypapers'.
7. Click on the purple `BlueskyAPIPassword` and change the dummy password `xxxx-xxxx-xxxx-xxxx` to yours from step 1 above.
8. Click `Save` at the top.
9. Click the `<-` back arrow at the top left
10. Click `Turn on` at the top.

#### 3.2 Test it

It's a good idea to do a dry run first. To do that, just hit the `Run` button at the top. This will run your literature bot, and as long as it found at least one paper matching your search terms from the last 24 hours, you'll be able to see it posted to your Bluesky account.

If nothing posts, either follow up the errors in Microsoft Flow, or if it says it 'Succeeded' in the 28-day-run-history, then check your RSS feeds. The chances are that there is nothing from the last 24 hours to post. 

#### 3.3 Set the trigger time

Finally, if you want you can set the time that the search happens each day. By default it starts at 10AM AEST, but to change it just:

1. Click `Edit`
2. Click the blue `Recurrence` variable
3. Change the `Time zone` and `At These Hours` to what you want. But ONLY use one hour. The system is built to run once every 24 hours, and if you do more than that you'll just get a lot of duplicates.

## Once you've got it running...

### Tweak, revise, repeat

Make sure you follow and check your own feed regularly. If it seems like it's posting rubbish, go tweak the RSS feeds. Leave it runnning for a week or two before telling too many people, so that you can see that it works like you want, and so when people find it they can see a bunch of good papers too.

### Let me know about your literature bot

I'd love to know if you built a literature bot with these instructions. If you did, please let me know via ["Show and tell" of the discussions](https://github.com/roblanf/phypapers/discussions/new?category=show-and-tell). 
