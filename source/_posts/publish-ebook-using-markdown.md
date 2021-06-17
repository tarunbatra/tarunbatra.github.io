---
title: Publish eBook using Markdown
date: 2021-06-01 21:18:06
tags: tools
photos: /data/images/publish-ebook-using-markdown/cover.png
---

You want to publish an eBook, maybe upload it to the Kindle Store or read it on your iPad as an epub file, or maybe just read it on your Linux system as a PDF. I wanted to do the same and was looking for a way in which I can do it using my preferred writing format, Markdown, without having to lock myself in a SaaS. When I started searching, it became clear [Pandoc][pandoc-url] was the way to go.

Pandoc is an awesome tool to convert documents from one format to another and it supports a lot of the formats. Due to it being highly configurable it can be difficult for someone new to start using it. So I will try to explain how you can utilize it for creating documents like ebooks just using Markdown. Though it should work similarly for other formats too.


## Structure

Broadly there are three things required to get started.
1. A **config file** which will tell Pandoc how to render the data, for example, do we need a TOC in the ebook, if yes then what should be the maximum depth of it?
2. A **metadata file** which will contain information like author name, copyright, etc.
3. Markdown **source file(s)** which will form the content of the ebook.

**📝 NOTE:** I already have a [base repository][starter-repo] that you can use as a starter for your book which you can fork and get going. Do not forget to change the Author name though. ;)

## Installing Pandoc
Pandoc is the only tool you need to get started with your ebook. [Pandoc's installation guide][pandoc-docs-installing] has the most up-to-date information on how to get it on your system.

If you are an Ubuntu user, most probably the following command will do the job:
```console
apt-get install pandoc
```
For Mac users:
```console
brew install pandoc
```

## Config file
A config file can be used to manage all the options passed to pandoc without passing each option through the cli. Here's an example config file.
```yml config.yml
metadata-file: metadata.yml  # file to use for metadata
table-of-contents: true      # enabling table of contents
toc-depth: 1                 # setitng max depth of 1 for table of contents
number-sections: true        # we will prefix headings with numbers
variables:
  documentclass: scrbook     # supported values: book, scrbook, article, etc.
  fontsize: 14pt
  classoption: openany       # disable empty pages in PDF
  papersize: a4
```

## Metadata file
Your book will have some metadata like title, author name, copyright. All of that goes in the `metadata.yml` file. An example file will do a far better job in explaining than I can.
```yml metadata.yml
---
title: My ebook
author: John Doe
cover-image: cover.png
rights: © 2021 John Doe, CC BY-NC
lang: en-US
---
```
The cover photo provided here will be used as the cover photo of the ebook. More options can be found in [Metadata docs][pandoc-docs-metadata] of Pandoc.

## Source files
You can write chapters in Markdown files. To preserve the order of the files, I suggest prefixing the name of the file with chapter number. It will ensure that the ordering of files is correct when you view it. The source files can also be individually passed to pandoc.
```console
pandoc -d config.yml 01-first-chapter.md 02-secon-chapter.md
```

## Generating files
In the [starter repo][starter-repo], all the Markdown files added to the `src` will be picked up and used as chapters of the book sorted in alphanumeric order. The images can be added to the `images` directory (or any other directory; I like to have my images in one place).

Thanks to the Makefile in the starter repo, generating an ebook is as simple as `make epub` and similarly `make pdf` can be used for generating a printable PDF version of the book.

The best part about using pandoc is that it supports a lot of formats formats and options, for example - custom css can be used to generate documents, if needed! I hope this gives you enough confidence and sets you on your eBook journey!

[pandoc-url]: https://pandoc.org
[starter-repo]: https://github.com/tarunbatra/ebook-starter
[pandoc-docs-installing]: https://pandoc.org/installing.html
[pandoc-docs-metadata]: https://pandoc.org/MANUAL.html#metadata-blocks
