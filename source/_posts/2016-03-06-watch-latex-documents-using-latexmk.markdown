---
layout: post
title: "Watch LaTeX Documents using Latexmk"
date: 2016-03-06 16:54:47 +0100
comments: true
categories: 
---

*tl,dr:* The `Watch Document` functionality of Textmate automatically compiles Tex documents when saved. This post describes how to achieve this behavior with any editor. It works for Unix-based systems, such as OSX or Linux. An example repository can be found [here](https://github.com/paulklemm/latexmk-synctex-example).

## Introduction: Switch from Textmate

I've been using Textmate to write and compile my LaTeX Documents since 2007. [This Blog post](http://jann.is/daily/archives/756-LaTex-Live-PDF-preview-with-TextMate-and-PDFView.html) by Jannis Hermanns basically defined my workflow since then. It's core is the neat `Watch Document` feature of Textmate, which compiles the document and looks for changes. It recompiles the document as soon as it is saved.

{% img center /media/2016-03-06-watch-latex-documents-using-latexmk/textmate_watch_document.gif The watch document command of Textmate previews changes when saving the document %}

This also works with LaTeX projects consisting of multiple files. Simply put the watch on the project master file and it will also look for changes in the associated files. This even extends to included images.

If you use a PDF viewer that supports [SyncTeX](http://itexmac.sourceforge.net/SyncTeX.html), such as [Skim](http://skim-app.sourceforge.net/), you can even jump to the current position of the cursor from the TeX document to the PDF and vice versa.

Once I got used to this convenience, compiling LaTeX documents by hand felt like a chore. Unfortunately, using Textmate began to feel likewise. Since I do all my coding in Sublime Text, I [looked for a similar workflow for that editor](http://tex.stackexchange.com/questions/276973/pendant-to-textmate-watch-document-function-in-sublime-text-latextools).
Turns out, it doesn't matter which editor you use, you can simply incorporate [`latexmk`](https://www.ctan.org/pkg/latexmk/) to achieve this functionality. Even better, `latexmk` is shipped per default with the most popular LaTeX distributions, so you should have it installed already.

## Compile LaTeX Projects using Latexmk

Consider the following LaTeX document `article.tex`:
{% codeblock article.tex lang:latex %}
\documentclass[12pt]{article}
\begin{document}
Hi!
\end{document}
{% endcodeblock %}

Using `latexmk`, you can compile this into a PDF file by entering the following command:

{% codeblock lang:bash %}
latexmk -pdf article.tex
{% endcodeblock %}

In order to watch for changes, you can simply add the `-pvc` option, which stands for "preview continuously":

{% codeblock lang:bash %}
latexmk -pdf -pvc article.tex
{% endcodeblock %}

As you can see, the command will not terminate back into the prompt, as `latexmk` watches for updated files (*targets*) associated with the previewed file. You have to shutdown it manually using `CTRL-C`. This is already close to the desired behavior. I suggest adding the following options:

- `-bibtex` to compile associated BibTeX files,
- `-f` to continue compiling albeit found errors,
- `-quiet` to make `latexmk` less verbose and show only important messages,
- `-use-make` to resolve missing files after the `pdflatex` call using custom dependencies, and
- `-pdflatex="pdflatex -synctex=1 -interaction=nonstopmode"`  to add options to the pdflatex command, most important the `synctex` command to allow jumping between the source PDF and the text editor.

For a precise description of each command, refer to the [latexmk readme](http://ftp.fernuni-hagen.de/ftp-dir/pub/mirrors/www.ctan.org/support/latexmk/latexmk.pdf). All options together yield the following command:

{% codeblock lang:bash %}
latexmk -quiet -bibtex -pvc -f -pdf -pdflatex="pdflatex -synctex=1 -interaction=nonstopmode" article.tex
{% endcodeblock %}

This is exactly the behavior I was looking for. And as a big plus, it is independent of your editor choice. Using it with [Sublime Text](https://www.sublimetext.com/) and the [LaTeXTools Plugin](https://www.sublimetext.com/) along with a Skim even allows to jump between source and compiled PDF using the enabled `synctex` feature.

{% img center /media/2016-03-06-watch-latex-documents-using-latexmk/synctex_sublime_skim.gif Jump between the IDE (Sublime Text 3 with LatexTools) and the PDF Viewer (Skim) using the Synctex feature %}

## Use the Power of `make` with `latexmk`

Utilizing this knowledge with `makefiles` is easy. I know that makefiles often look scary due to their seemingly obscure syntax, but actually they are easy to set up and will spare you *a lot* of work. If you want to have a quick introduction to `make`, I strongly recommend Mike Bostocks [beautiful blog post on it](https://bost.ocks.org/mike/make/). The commands applied here, however, are basic and should be self explanatory.

The `makefile` I propose is an adapted version from [this stackoverflow post](https://tex.stackexchange.com/questions/40738/how-to-properly-make-a-latex-project). I built the makefile to work as follows:

- `make` compiles the document, but does not watch for changes,
- `make preview` compiles the document and watches for changes using the `latexmk -pvc` command, and
- `make clean` removes all files produced using the LaTeX compilation process, including the resulting PDF.

{% codeblock makefile lang:make %}
# File adapted from this stackoverflow question: https://tex.stackexchange.com/questions/40738/how-to-properly-make-a-latex-project

# The first rule in a Makefile is the one executed by default ("make"). It
# should always be the "all" rule, so that "make" and "make all" are identical.
all: article.pdf

# MAIN LATEXMK RULE

# -pdf tells latexmk to generate a PDF instead of DVI.
# -pdflatex="" tells latexmk to call a specific backend with specific options.
# -use-make tells latexmk to call make for generating missing files.
# -interaction=nonstopmode keeps the pdflatex backend from stopping at a
# missing file reference and interactively asking you for an alternative.
# -synctex=1 is required to jump between the source PDF and the text editor.
# -pvc (preview continuously) watches the directory for changes.
# -quiet suppresses most status messages (https://tex.stackexchange.com/questions/40783/can-i-make-latexmk-quieter).
article.pdf: article.tex
    latexmk -quiet -bibtex $(WATCH) -f -pdf -pdflatex="pdflatex -synctex=1 -interaction=nonstopmode" -use-make article.tex

# The .PHONY rule keeps make from processing a file named "preview" or "clean".
.PHONY: preview
# Set the preview variable to -pvc to switch latexmk into the preview continuously mode
preview: WATCH=-pvc
preview: article.pdf

.PHONY: clean
# -bibtex also removes the .bbl files (http://tex.stackexchange.com/a/83384/79184).
clean:
    latexmk -CA -bibtex
{% endcodeblock %}

The `all` task creates the target file "article.pdf" using `latexmk`. You'll notice the `$(WATCH)` where the `-pvc` option should be. `$(WATCH)` refers to the variable `WATCH`, which is empty for all calls except for `make preview`, where `WATCH` is set to `-pvc`. The `make clean` command removes all LaTeX output files, including the BibTeX files as well as the PDF.

If you ran `make` and now want to run `make preview`, it will get the message "Nothing to be done for `preview`", since make sees that the target PDF is up to date. The `-B` option *forces* make to create the targets, so `make -B preview` will work.

## Closing Remarks

You can find a minimal example of a `LaTeX` project using this setup in this Github repository: [github.com/paulklemm/latexmk-synctex-example](https://github.com/paulklemm/latexmk-synctex-example). Feel free to play around with your own `makefile` rules and explore the possibilities.
