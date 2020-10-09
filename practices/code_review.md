# Code Review

When we review each other's work we build shared expertise and produce higher quality projects. We expect that all changes to project code will go through a github pull request process and be reviewed by a teammate before being merged.

We aspire to [egoless code reviews](https://github.com/oreillymedia/97-things-every-agile-developer-should-know/blob/master/Egoless_Code_Reviews.asciidoc).

Changes to internal documentation does not necessarily need to go through the code review process, although it is good practice to get team buy-in.

Aspects of a good review include:
- Reading through each file and ensuring that future-you will understand the intention of the code, what the code actually does, and that these match.
- Ensuring any new functionality has a test or tests that fully exercise the feature.
- Reading the code for anything that might cause production systems to become undeployable, or that might expose client data or keys that should remain private.
- You may choose to pull the code down locally to run it and poke at it, particularly for larger features, but this is not a required part of DCE's process.
