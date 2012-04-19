# Git recipes book

This is the complete source for my simple recipe book on Git. It includes all content and scripts to generate the epub file.

**Note: this is very much a work in progress.**

## Summary

Git is a popular version control system, that has many tricks up its sleeve. This will be a simple book about the workings of Git, and how to use it in your daily workflow. It is aimed at intermediate and advanced Git users, who know how to branch, commit, push and pull. It shall contain a mix of simple to follow recipes and background information.

**The best way to get a preview of a book is to clone the repo and run `rake preview.html` to generate a one-page HTML version of all the content.**

## Contributing

All contributions -- be it fixes or entire recipes -- are very welcome. Follow these steps:

1. Clone the repository on Github
2. Create a new feature branch for you changes
3. Push your changes to your fork
4. Send me a pull request in Github

## Installation

### Requirements

* Ruby 1.9+ (1.9.3 recommended)
* RubyGems 1.8+
* Rpub gem
* Git 1.7+

### Get the project

Clone the project Git repository from Github:

    $ git clone [URL]

### Generate the book

Use the [rpub][] gem to compile the input files to an .epub book:

    $ rpub compile

## Credits

**Author**: Arjan van der Gaag  
**URL**: http://arjanvandergaag.nl

## License

All content in the book is copyright 2011 Arjan van der Gaag and/or noted contributors. The text of the book is released under a [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Netherlands License][cc]. This means you can re-use and adapt the book for your own purposes, as long as it is not commercial and you share your modifications under the same license. All code in the book, and code to generate the epub file, is released under the MIT license.

[RVM]: http://beginrescueend.com
[Rbenv]: http://rbenv.org
[Markdown]: http://daringfireball.org/projects/markdown
[Coderay]: http://coderay.rubychan.de/ 
[cc]: http://creativecommons.org/licenses/by-nc-sa/3.0/nl/
[rpub]: https://avdgaag.github.com/rpub
