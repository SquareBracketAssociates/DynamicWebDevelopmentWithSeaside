# Dynamic Web Development With Seaside

A port of the original
[Dynamic Web Development with Seaside](http://book.seaside.st/book) book from
LaTeX/Pier to [Pillar markup](https://github.com/pillar-markup/pillar),
updated for the latest version of [Seaside](http://www.seaside.st/).

## Roadmap

The general plan of action is as follows.

1. *(done)* Commit the LaTeX source code for the original book.
    (at `original_source`/[book.tex](original_source/book.tex)).
    The images will be committed as each chapter is converted.

2. Create skeleton chapter structure (see
    [Revised Table of Contents RFC](https://github.com/SquareBracketAssociates/DynamicWebDevelopmentWithSeaside/issues/2)),
    with initial `.pillar` files for each chapter containing the title and the
    `@cha:introduction` type anchor tags, so that forward links work.

3. (Optional) Separate the LaTeX for each chapter into the appropriate
    directories, for convenience of converting them.

4. Convert each chapter to Pillar format, link to figures/images.

5. Update/revise the chapters to match newer versions of Seaside (and the
    various Smalltalk distributions).

## Contributing
This book follows the
[fork-and-pull](https://help.github.com/articles/using-pull-requests/#fork--pull)
GitHub workflow for contributions:

1. Fork the repository

2. For each modification, create a quick topic branch named in the form of...

   * initials_ChapterName_my_topic_description   

   example: `git checkout -b sd_UsingComponents_update_screen_snapshots`

3. Make commits to that branch. When you're ready, make a
    [Pull Request](https://help.github.com/articles/using-pull-requests/#sending-the-pull-request)

4. The request will receive comments/corrections, and will be merged into the
    main repo.

## License
This book is licensed under a
[Creative Commons Attribution-Noncommercial-Share Alike 3.0 license](http://creativecommons.org/licenses/by-nc-sa/3.0/).

## Attribution
Original book Copyright
[Stéphane Ducasse](http://stephane.ducasse.free.fr/),
[Lukas Renggli](http://www.lukas-renggli.ch/),
[C. David Shaffer](http://www.shaffer-consulting.com/),
[Rick Zaccone](http://www.bucknell.edu/x15822.xml).
