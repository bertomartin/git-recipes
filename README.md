# Git recipes book

This is the complete source for my simple recipe book on Git. It includes all content and scripts to generate the epub file.

**Note: this is very much a work in progress.**

## Summary

Git is a popular version control system, that has many tricks up its sleeve. This will be a simple book about the workings of Git, and how to use it in your daily workflow. It is aimed at intermediate and advanced Git users, who know how to branch, commit, push and pull. It shall contain a mix of simple to follow recipes and background infomration.

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
* Rake
* Kramdown
* Git 1.7+

### Get the project

Clone the project Git repository from Github:

    $ git clone [URL]

## Usage

This project consists of two main parts:

1. A set of content files in the `content` directory
2. A collection of Rake tasks and scripts to convert these files into an epub book

### Writing content

Content is written in [Markdown][] format in files in the `content` directory with file extension `md`. Every file will be assumed a chapter in the book, and every chapter title is taken from the first heading in the chapter. Every filename should start with its chapter number, e.g. `01-introduction.md`.

### Styling and layout

The look of the book is determined by the `layout.xhtml` and `syles.css` files. These should contain strict XHTML and CSS 2.1 code.

### Configuring book metadata

The `config.yml` file specifies all the book settings and metadata, such as title, author and copyright.

### Generating the book

Once you have written some chapters, you can generate the epub file from the command line:

    $ rake compile

Or, for the same result, as `compile` will be default:

    $ rake

This will generate a file in your project root directory based on the `filename` and `version` keys in `config.yml`, such as "my-awesome-book-0.0.1.epub".

### Code highlighting

The Kramdown library gives us code highlighting for free, as long as we specify a `lang` attribute on a code block like so:

    Here is a code sample:

        def hello
          "Hello, world"
        end
    {:lang="ruby"}

    Doesn't it look great?

Under the hood, Kramdown uses [Coderay][], so refer to its documentation for a list of supported languages.

## Wishlist

* Add proper support for images, including compression
* Add proper support for guides, indicating cover page, toc, etc.
* Add support for Kramdown features such as footnotes and abbreviations

## Task reference

* `rake preview.html`: generate a `preview.html` file that contains all content from the `content` directory. You can open it in a browser to get a preview of the book's contents in a single page.
* `rake clean`: remove al temporary files, such as the generated files in the `book` directory and preview files.
* `rake clobber`: remove all generated files, including any .epub files.
* `rake compile`: compile all contents into an .epub file, but only when the file does not yet exist or is outdated.
* `rake recompile`: force re-compilation of the .epub file, even when the file exists and is not outdated.
* `rake test`: use the `epubcheck` program to test if .epub file has any errors.

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
