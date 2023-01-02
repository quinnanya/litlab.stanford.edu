# LitLab.stanford.edu guide

## How the site works

In the big picture, all the site content -- project descriptions, Techne posts, pamphlets, information about Lab members -- exists as a series of text files, mostly using [markdown](https://www.markdownguide.org/basic-syntax/) syntax, which get processed through a set of templates to create the website. To update the site contents, you create new markdown files or update existing ones in the *src* directory; you can also add images or other files and reference them in your Markdown files.

The one exception is the information about Lab members, which is managed in a JSON file at *src/data/LabPeople.json*.

We'll go through the site one content type at a time.

## Projects


## Techne

Techne posts live in *src/techne*. Their filenames should follow the following convention: `yyyy-mm-dd-post-title-separated-by-hyphens.md`. At the top of each Techne post markdown file, there's *frontmatter* -- basically, metadata. The frontmatter section begins and ends with three hyphens, and you should fill in the values for each of the fields. Here's an example:

    ---
	permalink: '/techne/example-post/'
    title: "Techne's Example Post"
    authors: [efredner, malgeehewitt]
    projects: [typicality]
    date: 2020-07-18
    teaser: |
      A sentence or two goes here, and you don't have to worry about single or double quotes.
	---

Text that isn't part of a list should be surrounded by either single or double quotes; if the text includes an apostrophe (like in the example title here), use double quotes, and vice-versa.

Lists should be in square brackets, and you don't have to put quotes around list items. The two lists associated with Techne posts are *authors* and *projects*. To reference a Lab member or collaborator, use the first letter of their first name and their full last name, all lower-case. (If you're not sure look at the *src/data/LabPeople.json* file, which is ordered alphabetically by last name.)

## License
