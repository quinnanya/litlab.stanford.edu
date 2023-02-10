---
permalink: '/techne/new-litlab-website/'
title: "A New LitLab Website"
authors: [mwarner, qdombrowski]
projects:
date: 2023-02-20
teaser: "We're pleased to introduce the new website for the Stanford Literary Lab; it's the first overhaul of the site since 2015, and the first time the site has moved off of WordPress."
---

We're pleased to introduce the new website for the Stanford Literary Lab; it's the first overhaul of the site since 2015, and the first time the site has moved off of WordPress.

When the Lab's built its website (first captured by the Internet Archive in [February 2011](https://web.archive.org/web/20110206155231/http://litlab.stanford.edu/)), WordPress was the obvious choice for a simple, no-fuss informational website, and it  served the Lab well as low-maintenance infrastructure for a decade.

As we updated the site to a recent version of WordPress, we realized how much the platform has changed. For a modern-looking theme with a reasonable amount of flexibility, prepare to pay -- and not once, but as an annual single-site subscription, typically requiring an annual subscription to one or more premium plugins. These plugins and themes override each other in surprising and one-off ways, leading to a site that is frustrating to build, challenging to document for future site maintainers, and depends on ongoing payments to keep working. We needed a better path.

There are currently two common routes that lead towards static websites: principles and praxis. The former adopts minimal computing practices as an ideology, seriously considering issues of ecological impact and global infrastructural limiatations. The latter is often borne out of acknowledging the unending labor that must go into maintaining the technical underpinnings of a site, in order to avoid the consequences of being hacked, or the site falling apart as dependencies further down in the stack are upgraded beyond the site's ability to bear that change.

For us, the shift to a static site (using the [Node-based Eleventy framework](https://www.11ty.dev/)) was more motivated by the latter path more than the former, though the "[questions of minimal computing](http://www.digitalhumanities.org/dhq/vol/16/2/000646/000646.html)" posited by Roopika Risam and Alex Gil also inform our thinking with this new site:

> 1) “what do we need?”; 2) “what do we have”; 3) “what must we prioritize?”; and 4) “what are we willing to give up?”

## What do we need?

Arguably, this question is the most challenging for the Lab, revealing some of the biggest differences of opinion. The Lab has an ISSN for its Pamphlets series, which resolves to a webpage according to an international metadata record. At a minimum, we need a website for that. How valuable is a list of current and former members? How useful is a list of our current projects? What about the past projects? Do we want to maintain a page for each project, in perpetuity, for current and former members to point to as part of their CVs? And then there's the question of anything blog-like, be it the "[Techne](/techne)" posts or something that looks more like "[News](/news)", covering things like Lab presentations, conference talks, and grant awards. The tensions around any kind of updates run deep in the Lab; looking back to the first version of the website, it had [its own News section](https://web.archive.org/web/20110630171551/http://litlab.stanford.edu/?page_id=107) with several posts in 2011... all of which [vanish by 2012](https://web.archive.org/web/20120113180402/https://litlab.stanford.edu/).

This new version of the site pushes the Lab the farthest it's ever gone in the direction of publicly recording its work, whether or not that work manifested in a recognized publication with a pamphlet. We've also gone back through our mailing list to find old project presentations, and backfill a new generation of "News" with those announcements. This approach is intended to acknowledge and honor the work that goes into the kinds of projects that we do at the Lab, recognizing that despite the best efforts of entire project teams, projects will not always produce some satisfying scholarly output at the end -- but that doesn't mean it was a failure or a loss. Our answer to the question of "what do we need" is that we need some different technical infrastructure on the website, including persistent project pages and a news feed, so members' ability to point to their work at the Lab doesn't depend on the outcome of those projects, which may be out of members' hands. This approach makes experimentation and taking on challenging questions less risky.

## What do we have?

What we have is fundamentally shaped by the website framework we have been operating in: pamphlets, a static, single projects page with (varying degrees of outdated information about what once were) current projects, and Techne as a kind of blog with 16 posts over 7 years. 

What we have is less interesting than what we are willing to make. And to that end, over the course of rebuilding this website we have created close to 50 news posts that capture the Lab's work, quarter-by-quarter, through its presentations. We have created [archived project](/projects/archive/) pages to connect with each pamphlet, along with several projects from the last five years that came to a close without producing a pamphlet, but reflect a considerable amount of thought and work nonetheless. We have also grappled with the always-challenging question of "what are our [current projects](/projects)", and made pages for those as well.

## What must we prioritize?

The data modeling choices that we've made in rebuilding the site prioritize the needs of members for whom it is meaningful to have a pointer to their work on Lab projects, including those on the job market. We've also overhauled the People page to give core Lab researchers more visible space.

The desire to minimize technical site maintenance is real as well. Reducing the technical infrastructure of the site means that there are fewer things that could fundamentally bring down the site -- which is always appealing when there is no long-term webmaster, but that role is handed off amongst grad students.

## What are we willing to give up?

For people accustomed to working with WordPress's WYSIWYG interface, shifting to working directly with Markdown files can be a bit of a shock. That said, that has never been the Lab's workflow: there's one designated webmaster handling website changes, but the way WordPress handled authorship meant that several people needed (at least dummy) accounts for attribution. 

We've set up a series of templates to process the site content, and written documentation (LINK TO GITHUB REPO) for how to update or add new instances of all the types of content on the site. For most Lab members, we expect their experience with the website will involve filling in text-file templates for different kinds of content using Markdown and/or YAML, and either emailing them to the webmaster or -- for the bold -- proposing them as a pull request on GitHub, where the site is published. Writing in Markdown and/or YAML may not be the most comfortable for Lab members, but those skills are useful in the context of other tools, and worst case, there are format converters that can lighten the load on the webmaster if people end up struggling. Even publishing on GitHub means a set of trade-offs: it takes us outside the sphere of Stanford IT support requests, but also leaves us less vulnerable to the web platform shifts that have characterized the last several years at Stanford.

## The future of the Lab site

If you're curious about what we're doing technically with the LitLab site, check out our (GITHUB REPO). We've come up with some interesting workarounds for things like bibliographic formatting, by using a [plugin that formats citations from bibtex](https://github.com/Savjee/eleventy-plugin-bibtex), and automatically generating the bibtex from (arguably) friendlier-to-author YAML frontmatter. 

That said, in the end, the biggest challenges for website maintenance are social, rather than technical. Only time will tell how it all will turn out.