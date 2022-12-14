---
permalink: '/counting-words-in-hathitrust-with-python-and-mpi/'
title: "Counting words in HathiTrust with Python and MPI"
authors: [dmcclure]
projects: []
date: 2016-08-26
teaser: |
  In recent months we’ve been working on a couple of projects here in the Lab that are making use of the Extracted Features data from HathiTrust. To help kick off the lab’s new Techne series, I wanted to take a look at some of the programming patterns we’ve been using that make it easier to work these kinds of large data sets – namely the “Message Passing Interface” (MPI), a set of semantics for spreading out programs in large computing grids.
---

In recent months we've been working on a couple of projects here in the Lab that are making use of the [Extracted Features data set](https://analytics.hathitrust.org/features) from HathiTrust. This is a fantastic resource, and I owe a huge debt of gratitude to everyone at HTRC for putting it together and maintaining it. The extracted features are essentially a set of very granular word counts, broken out for each physical page in the corpus and by part-of-speech tags assigned by the OpenNLP parser. For example, we can say -- on the first page of *Moby Dick*, "Call" appears 1 time as a `NNP`, "me" 5 times as a `PRP`, and "Ishmael" 1 time as a `NNP`, etc. What's missing, of course, is the syntagmatic axis, the actual sequence of words -- "Call me Ishmael." This means that it's not possible to do any kind of analysis at the level of the sentence or phrase. For instance, we couldn't train a [word2vec](https://en.wikipedia.org/wiki/Word2vec) model on the features, since word2vec hooks onto a fairly tight "context window" when learning vectors, generally no more than 5-10 words. But, with just the per-page token counts, it is possible to do a really wide range of interesting things -- tracking large-scale changes in word usage over time, looking at how cohorts of words do or don't hang together at different points in history, etc. It's an interesting constraint -- the macro (or at least meso) scale is more strictly enforced, since it's harder to dip back down into a chunk of text that can actually be read, in the regular sense of the idea.

The real draw of this kind of data set, though, is the sheer size of the thing, which is considerable -- 4.8 million volumes, 1.8 billion pages, and many hundreds of billions of words, packed into 1.2 terabytes of compressed JSON files. These numbers are dizzying. I always try to imagine what 5 million books would look like in real life -- how many floors of the stacks over in Green Library, here at Stanford? How many pounds of paper, gallons of ink? In the context of literary studies, data at this scale is fascinating and difficult. When we make an argument based on an analysis of something like Hathi -- what's the proper way to frame it? What's the epistemological status of a truth claim based on 5 million volumes, as opposed to 2 million, 1 million, a hundred thousand, or ten? Surely there's a difference -- but how big of a difference, and what type of difference? Is it categorical or continuous? What's the right balance between intellectually capitalizing on the scale of the data -- using it to make claims that are more ambitious than would be possible with smaller corpora -- and also avoiding the risk of over-generalizing, of mistaking a (large) sample for the population?

These are wonderful problems to have, of course. In addition to the philosophical challenges, though, we quickly realized that the size of the corpus also poses some really interesting technical difficulties. The type of code that I'm used to writing for smaller corpora will often bounce right off a terabyte of data -- or at least, it might take many days or weeks to inch through it all. To help kick off the lab's new *Techne* series, I wanted to take a look at some of the parallel programming patterns we've been working with that make it possible to spread out these kinds of big computations across many hundreds or thousands of individual processors -- namely, a protocol called the "[Message Passing Interface](https://en.wikipedia.org/wiki/Message_Passing_Interface)" (MPI), a set of programming semantics for distributing programs in large computing grids. This is under-documented, and can feel sort of byzantine at times. But it's also incredibly powerful, and, from a standpoint of programming craft, it introduced me to a whole new way of structuring programs that I had never encountered before.

Now, I'd be remiss not to mention that HathiTrust actually provides a platform that makes it possible to run custom jobs on their computing infrastructure. (You can sign up for an account [here](https://analytics.hathitrust.org/signup).) This is extremely cool, though we've run into a number of situations recently -- both with Hathi and with other data sets -- where we found ourselves needing to write this type of code, so I wanted to figure out how to do it in-house. The extracted features seemed like an obvious place to start.

#### The simple way -- loop through everything, one-by-one

So, we've got 5 million bzipped JSON files. Generally, to pull something of interest out of the corpus, we need to do three things -- decompress each file, do some kind of analysis on the JSON for the volume, and then merge the result into an aggregate data structure that gets flushed to disk at the end of the process.

Say we've got a Python class that wraps around an individual volume file in the corpus:

<p><script src="https://gist.github.com/davidmcclure/3d45c6ca3cdd17c535e7abc3fb704356.js"></script></p>


This just reads the file, parses the JSON, and sets the data on the instance. Here, we've got a `token_count()` method, which steps through each page and adds up the total number of tokens in the book.

And, similarly, say we've got a `Manifest` class, which wraps around a local copy of the `pd-basic-file-listing.txt` from Hathi, which provides an index of the relative locations of each volume file inside the pairtree directory. `Manifest` just joins the relative paths onto the location of the local copy of the features directory, and provides a `paths` attribute with absolute paths to all 5M volumes:

<p><script src="https://gist.github.com/davidmcclure/bd4701a5b44db60089f81d40462644c0.js"></script></p>


To run code on the entire corpus, the simplest thing is just to loop through the paths one-by-one, make a volume instance, and then do some kind of work on it. For example, to count up the total number of tokens in all of the volumes:

<p><script src="https://gist.github.com/davidmcclure/ca85a90decc0606b293870f6713f5f9e.js"></script></p>


This kind of approach is often good enough. Even if it takes a couple hours on a larger set of texts, it often makes more sense to keep things simple instead of putting in the effort to speed things up, which itself takes time and tends to make code more complex. With Hathi, though, the slowness is a deal-breaker. If I point this at the complete corpus and let it run for an hour, it steps through 14,298 volumes, which is just 0.2% of the complete set of 4.8 million, meaning it would take 335 hours -- just shy of 14 days -- just to loop through the pages and add up the pre-computed token counts, let alone do any kind of expensive computation.

Why so slow? Reading out the raw contents of the files is fast enough, but, once the data is in memory, there's a cost associated with decompressing the .bz2 format and then parsing the raw JSON string, which, for an entire book's worth of pages, is long. But, since neither of these steps are IO-bound -- the costly work is being done by the CPU, not the disk -- this is ripe for parallelization. Out of the gate, though, Python isn't great for parallel programming. Unlike some more recent languages like Go, for example -- which bakes [concurrency primitives](https://www.golang-book.com/books/intro/10) right into the core syntax of the language -- Python programs always run on a single CPU core, and the much-maligned "global interpreter lock" means that only a single thread is allowed to do work at any given moment, regardless of the resources available on the machine.

#### The better way -- multiple cores on the same machine

To work around this limitation, though, Python has a nice module called `multiprocessing` that makes it easy to make use of multiple cores -- the program is duplicated into separate memory spaces on different CPUs, work is spread out across the copies, and then the results are gathered up by a controller process at the end. The API is [fairly large](https://docs.python.org/3.5/library/multiprocessing.html), but it's generally easiest to use the `Pool` class, which basically provides parallel implementations of `map` in a couple of different flavors. For example, with the Hathi data -- we can write a worker function that takes a file path and returns a token count, and then use the `imap_unordered` function to map this across the list of paths from the manifest:

<p><script src="https://gist.github.com/davidmcclure/a7aff2ed46406d37becb70674bc31ab6.js"></script></p>


This produces a really nice speedup -- now, over an hour, a 16-core node steps through 117,951 volumes, an 8x speedup from the single-process code. But, the numbers are still forbidding when scaled up to the full set of 5M volumes -- even at ~120k volumes an hour, it would still take about 40 hours to walk through the corpus. And again, this is just the bare minimum of adding up the total token count -- a more expensive task could run many times slower.

Can we just keep cranking up the number of processes? In theory, yes, but once we go past the number of physical CPU cores on the machine, the returns diminish fairly quickly, and beyond a certain point the performance will actually drop, as the CPU cores start scrambling to juggle all of the processes. One solution is to find a massive computer with lots of CPUs -- Amazon Web Services, for example, now offers a gigantic "X1 32xlarge" instance with 128 cores. But this is pretty much the upper limit.

#### MPI -- multiple cores on multiple machines

So, only so many cores can be stuffed into a single machine -- but there isn't really a limit to the number of computers that can be stacked up next to each other on a server rack. How to write code that can spread out work across multiple computers, instead of just multiple cores?

There are few different approaches to this, each making somewhat different assumptions. On the one hand there are "MapReduce" frameworks like [Hadoop](http://hadoop.apache.org/) and [Spark](http://spark.apache.org/), which grew up around the types of large, commodity clusters that can be rented out from services like Amazon Web Services or Google's Compute Engine. In this context, the inventory is often enormous -- there are lots and lots of servers -- but it's assumed that they're connected by a network that's relatively slow and unreliable. This leads to a big focus on fault tolerance -- if a node goes offline, the job can shuffle around resources and recover. And, since it's slow to move data over a slow network, Hadoop is really invested in the notion of "data locality," the idea that each node should always try to work on a subset of the data that's stored physically nearby in the cluster -- in RAM, on an attached disk, on another machine on the same server rack, etc.

Meanwhile, there's an older approach to the problem called the "Message Passing Interface" (MPI), which is used widely in scientific and academic contexts. MPI is optimized for more traditional HPC architectures -- grids of computers wired up over networks that are fast and reliable, where data can be transferred quickly and the risk of a node going offline is smaller. MPI is also more agnostic about programming patterns than MapReduce frameworks, where it's sometimes necessary to formulate a problem in a fairly specific way to make it fit with the map-reduce model. MPI is lower-level, really just a set of primitives for exchanging data between machines.

From the perspective of the programmer, MPI flattens out the distinction between different cores and different computers. Programs get run on a set of nodes in a computing cluster, and, depending on the resources available on the nodes, the program is allocated a certain number of "ranks," which are essentially parallel copies of the program that can pass data back and forth. Generally, one rank gets mapped onto each available CPU core on each node. So, if a job runs on 32 nodes, each with 16 cores, the program would get replicated across 512 MPI ranks.

Writing code for MPI was a bit confusing for me at first because, unlike something like a multiprocessing `Pool`, which is functional at heart -- write a function, which gets mapped across a collection of data -- with MPI the distinction between code that *does work* and code that *orchestrates work* is accomplished with in-line conditionals that check to see which rank the program is running on. You just write a single program that runs everywhere, and that program has to figure out for itself at runtime which role it's been assigned to. MPI provides two basic pieces of information that makes this possible -- the `size`, the total number of available ranks, and the `rank`, a offset between 0 and `size` that identifies this particular copy of the program. To take a trivial example -- say we've got 5 MPI ranks, and we want to write a program to compute the square root of 4 numbers. Rank 0 -- the controller rank -- broadcasts out each of the numbers, and then ranks 1, 2, 3, and 4 each receive a number and do the computation:

<p><script src="https://gist.github.com/davidmcclure/c6b283a26c9198def30363e074ab695a.js"></script></p>
<p><script src="https://gist.github.com/davidmcclure/2d446e4c9ada195458c6f747a187494f.js"></script></p>

Beyond this kind of simple “point to point” communication – one ranks sends some data, another receives it – MPI also has a number of synchronization utilities that make it easier to coordinate work across groups of ranks. Unlike the code above, most MPI programs have just two branches – one for rank 0, which is responsible for splitting up the task into smaller pieces of work, and another for all of the other ranks, each of which uses the same code to pull instructions from rank 0. For example, the scatter and gather utilities make it possible to split a set of input data into N pieces, “scatter” each piece out to N ranks, wait until all of the worker ranks finish their computations, and then “gather” the results back into the controller rank. Eg, for the square roots:

<p><script src="https://gist.github.com/davidmcclure/3fd74edb99f428614cdacdddf2543c64.js"></script></p>


This is starting to look like the kind of approach we’d want for a data set like Hathi – just replace the integers with volume paths, and the square roots with some kind of analysis on the feature data. In essence, something like:

<p><script src="https://gist.github.com/davidmcclure/f6ae97c08e599490fbba11f533bfe327.js"></script></p>


This works like a charm. Here’s a complete program that counts up the total number of tokens in the corpus:



<p><script src="https://gist.github.com/davidmcclure/8becd4679c99ada94f3fce05293274db.js"></script></p>


(Simplified just a bit for readability -- see the full version [here](https://github.com/davidmcclure/hathi-mpi/blob/master/jobs/scatter_list.py), along with the benchmarking programs for all the other code in this post.)

On 16 nodes on Stanford's Sherlock cluster, this runs in about 140 minutes -- ~2.3 hours -- a 18x speedup over the `multiprocessing` solution on a 16-core machine and 145 times faster than the original single-threaded code. And, this scales roughly linearly with the number of nodes -- 32 nodes would finish in just over an hour, 64 in half an hour, etc. This approach has served us well with the first projects we've been using the Hathi data for -- a look at the history of the word "literature," in collaboration with a group at Paris-Sorbonne. Though, I'm still new to this type of programming, and my guess is that there are ways that this could be improved pretty significantly.

One question I'm still unsure about -- instead of decompressing the volumes on-the-fly during jobs, would it make sense to just do this once, write the inflated files back to the disk, and then run jobs against the regular JSON? I think this would speed up the jobs themselves -- the decompression step accounts for about 40% of the time that it takes to materialize a volume. (Though, we'd also be pulling more data off the filesystem, which takes time -- so I'm not sure.) I haven't gone down this road, though, because it seems like there are other costs, if only in terms of data management and programming hassle. It would take up much more disk space, for one thing -- about 9 terabytes, on top of the 1.2 for the original files. And, it would mean that we'd have to remember to re-run this step if Hathi updates the corpus, etc. As a rule of thumb -- I never really love the idea of creating "downstream" versions of data sets when it can be avoided, since I think it often adds surface area for mistakes and makes things harder to reproduce down the line. If the MPI-ified job can run in 2 hours against the original .bz2's, I'm not sure it would be worth adding complexity to the code just to get it down to 1 hour, or whatever. I guess this might make sense if we were running lots and lots of jobs, but I doubt we'll be doing that.

So, how many tokens in Hathi? We count 814,317,177,732 -- which, I have to pinch myself to remember, is 80% of a trillion. This is actually quite a bit more than the 734 billion number [reported by Hathi](https://www.hathitrust.org/htrc-releases-massive-dataset) back in 2015. Maybe it's grown a billion-odd words in the last year? Or, we might be counting differently -- we're just adding up the top-level `tokenCount` keys on each page, which I believe include all the OCR errors that would get filtered out in real projects.

Either way -- Hathi is a kind of Borgesian dream. More to come.