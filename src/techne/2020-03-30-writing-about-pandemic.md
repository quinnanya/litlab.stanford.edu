---
permalink: '/writing-about-epidemic/'
title: "Writing about Epidemic"
authors: [mwarner]
projects:
date: 2020-03-30
teaser: |
  Shortly after the Lab released my recent pamphlet on the structure of the literary canon, New York magazine ran an article about the 21st century canon, in which a panel of judges pick an early version of the literary canon from the century so far.
---

This project started life as a "what to do while we're all stuck inside" activity---starting a project on writing and disease seemed like it would be a useful way of channeling our restlessness into something productive. This isn't a "coronavirus" project, per se, though: while we're interested in current events, we're also trying to think more broadly about how writers of all kinds have responded to conditions of epidemic and pandemic in their own times and places. Nevertheless, given the timely nature of the topic, we thought it would be good to try to do something *now *and share the interim results as quickly as possible. To facilitate this, we've branched in three different directions to explore different methods, corpora and questions---depending on how things go, we might eventually settle on one particular angle as the most generative, or we might have a three-pronged project on our hands.

We're going to be using this blog as a venue to publish what we find as we go, and writing about our methods, tools, and practical and impractical decisions along the way. This first post outlines the different avenues we're exploring and the corpora we're using. A second post, with some tentative results, will be forthcoming.

Project members: Mark Algee-Hewitt, Yibing Du, Charlotte Lindeman, Nika Mavrody, Nichole Nomura, Carmen Thong, Matt Warner.

#### The Evolving Language of Disease

*Question: *How has the reporting language around coronavirus changed over the course of the pandemic?

*Corpus: *Right now, we're working with two different corpora: one is a collection of new articles from the news sites NBC News, Huffington Post, Politico, ABC News, CBC News and the Guardian UK. We've been scraping these sites for all results pertaining to "Coronavirus" to put together a fairly large corpus---we're hoping to eventually have about 10 million words, so that we can build some word-embedding models of the language used. We're imagining that down the road we might add additional sites to this list (we picked these ones because they don't have paywalls and were---fairly---easy to scrape).

At the same time, we've been building a hand-collected corpus of recent news coverage in three important (paywalled) American news venues. We started with the New York Times, and we added the Wall Street Journal and the Financial Times--this lets us compare conservative and liberal news outlets, as well as three points on the spectrum from broad readership to business specificity, since some of us are interested in the economic language used to discuss COVID-19. This corpus consists of the top ten results for each site on each day of March up to the 20th.

*Methods: *Our approach starts with vector modelling: we want to build a model of the larger scraped corpus so we can start to explore the contours of pandemic-related language.We've been talking about different ways to extend this model or use it so we can track change, but while we work on that, we want to try to do vector space models for each of our newspapers newspaper for each day. Because of the tiny size of this corpus, and our interest in tracking diachronic change, we'll use singular value decomposition with positive pointwise mutual information, which means that we won't have to filter out initial randomization as we would for a GloVe or Word2Vec model. SVD isn't something we've used extensively in the lab, since it's computationally demanding / slow (and in some ways inferior in its results, too), but it also doesn't require millions of words, so it's (hopefully!) perfect for this.

#### Literature of Confinement

*Question: *What textual features correlate with narratives that involve severe limitations of physical space and/or character space?

*Corpus: *This is a corpus of literature, selected by group members. We're after prose texts that involve someone being stuck in a small place (like "The Yellow Wallpaper") or isolated somewhere (like Robinson Crusoe) or otherwise dealing with solitude and limited space. To begin with, we're casting a wide net in terms of period, form, genre, nationality, and so on. The goal is to see if these limitations within the diegesis have consistent textual correlates---but, also, this is just a good time to think about these stories.

*Methods: *We're going to take a very exploratory approach, but as an initial pass we may try collecting most distinctive words relative to a comparison corpus (of literature that has nothing to do with confinement). Since this is a small corpus that members of the group know pretty well, we may also use more qualitative methods like feature tagging.

#### Personifying Illness

*Question: *How and when do we switch from talking about illness as a thing to talking about illness as an agent, character, or person?

*Corpus: *This project is interested in both fictional and non-fictional personifications of germs, illness, disease, and epidemic. Our investigation is starting with a hand-assembled list of fiction and non-fiction we think personifies illness in some way; and eventually we're likely to search for the features we learn from this corpus to others.

*Methods: *We don't have a predetermined feature set for identifying personification, characters, or agents in text, but work on other Lab projects suggests that Named-entity-recognition, advanced searching, vector space, and close-reading might be a good place to start.